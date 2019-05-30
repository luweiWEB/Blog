### 背景
富文本编辑是管理后台（cms）系统中的重要功能，编辑器的选择也非常多，如今大多编辑器都是走的简约路线，遇上挑剔的客户就无法满足他们的需求。百度的ueditor作为一款重量级的编辑器，提供了强大的功能，并且从word中直接copy到编辑器中的还原效果也非常好，但是由于官方已经很久没有维护了，所以对接已有的系统灵活度不够。
基于vue封装的ueditor组件挺多的，并且封装和改造的效果都还不错，比如[vue-ueditor-wrap](https://github.com/HaoChuan9421/vue-ueditor-wrap)，在封装[react-ueditor-component](https://github.com/Eschere/react-editor-component)过程中也借鉴了开源社区中的优秀代码。

### 功能需求
ueditor其他功能没什么需要改动的，但是上传文件的功能与后端耦合太高，不符合现在的前后端分离的系统设计，也不好对接第三方存储（如七牛OSS），所以要改造实现基本的两个功能：
1. 后端配置前移，上传文件的配置参数直接写在前端，不需要请求一次后端接口才能初始化上传文件的功能
2. 自定义上传文件的请求头和请求，虽然官方提供了解决方案，但是不够灵活

另一块就是基础的编辑功能，封装后的组件应该像`input`使用一样简单，`value`控制编辑器内容，`onChange`监听编辑器内容变化事件


下面解析一些核心功能的实现思路
### 初始化编辑器
`ueditor`的初始化是异步的，所以需要在编辑器准备就绪后才能进行后续的操作，这里使用`Promise`进行流程控制
```js
componentDidMount () {
  // 编辑器ready后再进行后续操作
  this.setState(state => ({
    editorReady: new Promise((resolve, reject) => {
      let ueditor = window.UE.getEditor(this.editorId, {
        ...this.ueditorOptions, // 一些默认参数
        ...this.props.ueditorOptions // props传入的参数
      });

      ueditor.ready(() => {
        resolve(ueditor);

        this.observerChangeListener(ueditor); // 初始化监听编辑器变化的方法，后面会具体说明

        ueditor.setContent(this.props.value || '');
      });
    })
  }));
}
```

### value

value改变触发`react-ueditor-component`中的编辑器的变化是个很简单的父组件向子组件传参，
使用`static getDerivedStateFromProps`就可以实现
```js
static getDerivedStateFromProps (nextProps, prevState) {
  let editorReady = prevState.editorReady;
  let value = nextProps.value;

  if (Object.prototype.hasOwnProperty.call(nextProps, 'value')) {
    editorReady && editorReady.then((ueditor) => {
      (value === prevState.content || value === ueditor.getContent()) || ueditor.setContent(value || '');
    });
  }

  return {
    ...prevState,
    content: value
  };
}
```
上面的代码比想象中复杂一点，在组件内的`state`中会创建一个属性`content`用于存储上次传过来的`value`，`props.value`会和`content`和编辑器中实际的内容比较
因为在一些特殊情况下，编辑器中的内容会发生变化，而同时`getDerivedStateFromProps`会被触发但是`value`并没有发生变化，如果不进行比较编辑器中的内容会被回退为旧值。

### onChange
编辑器内容变化可以使用ueditor提供的[contentChange](https://ueditor.baidu.com/doc/#UE.Editor:contentChange)，但是会有bug，比如按下多个按键时并不会触发该事件

`react-ueditor-component`采用`MutationObserver`监听DOM变化
```js
observerChangeListener (ueditor) {
  const changeHandle = () => {
    let onChange = this.props.onChange;

    if (ueditor.document.getElementById('baidu_pastebin')) {
      return;
    }

    onChange && onChange(ueditor.getContent());
  };

  this.observer = new MutationObserver(changeHandle); // FIXME: 这里可以使用debounce节流

  this.observer.observe(ueditor.body, {
      attributes: true, // 是否监听 DOM 元素的属性变化
      attributeFilter: ['src', 'style', 'type', 'name'], // 只有在该数组中的属性值的变化才会监听
      characterData: true, // 是否监听文本节点
      childList: true, // 是否监听子节点
      subtree: true // 是否监听后代元素
    });
}
```

### 后端配置前移
此功能的实现需要修改`ueditor`源码了，笔者从[fex-team/ueditor](https://github.com/fex-team/ueditor)fork了一份，基于`dev-1.4.3.3`分支创建了`dev-3.0.0`分支，[github](https://github.com/Eschere/ueditor/tree/dev-3.0.0)，所有代码的修改都用`MARK:`标记出来了，可以全局搜索查看所有源码改动

只需要找到获取配置的方法并修改就可以了，在`_src/core`中
```js
 UE.Editor.prototype.loadServerConfig = function(){
  this._serverConfigLoaded = false;

  try {
    utils.extend(this.options, this.options.serverOptions);
    utils.extend(this.options, this.options.serverExtra);
    this.fireEvent('serverConfigLoaded');
    this._serverConfigLoaded = true;
  } catch (e) {
    console.error(this.getLang('loadconfigFormatError'));
  }
}
```
相应的，封装的`react-ueditor-component`增加了字段配置
```js
window.UE.getEditor(this.editorId, {
  serverUrl: this.props.ueditorOptions.serverUrl,
  serverOptions: {
    imageActionName: 'uploadimage',
    imageFieldName: 'file',
    ...others
  },
  serverExtra: this.props.ueditorOptions.serverUrl
});
```

### beforeUpload钩子
`beforeUpload`钩子是自定义请求数据实现的关键，但实现的功能又不止于增加自定义请求数据

`beforeUpload`方法由参数传入ueditor

上传前需要进行的操作很多情况下可能是一个异步过程，这里使用`Promise`进行流程控制，以`autoupload.js`为例
```js
if (me.options.beforeUpload) {
  Promise.resolve(me.options.beforeUpload(file)).then(function (file) {
    if (!file) {
      return
    }

    // 设置请求头和请求内容，开始上传
  })
} else {
  // 设置请求头和请求内容，开始上传
}
```

### 自定义请求数据
自定义请求数据用`serverExtra`实现，需要这部分内容是随时可变的，所以需要新增一个方法，可以随时设置`serverExtra`
```js
UE.Editor.prototype.setExtraData = function (options) {
  try {
    utils.extend(this.options, options);
  } catch (e) {
    console.error(this.getLang('setExtraconfigFormatError'));
  }
}
```
上面的代码不难看出来，实际上`setExtraData`方法可以设置任何配置，但是后续封装组件并使用时，我只建议用于修改`serverExtra`，因为修改ueditor的其他参数并不一定有效，并且可能会出现无法预期的bug。

在每次执行上传之前应该读取配置、设置上传内容，以`autoupload.js`为例
```js
var fd = new FormData()
// 请求体中增加额外数据
if (me.options.extraData && Object.prototype.toString.apply(me.options.extraData) === "[object Object]") {
  for (var key in me.options.extraData) {
      fd.append(key, me.options.extraData[key]);
  }
}
// 请求头中增加额外数据
if (me.options.headers && Object.prototype.toString.apply(me.options.headers) === "[object Object]") {
  for (var key in me.options.headers) {
    xhr.setRequestHeader(key, me.options.headers[key]);
  }
}
```

封装在组件中，需要在`static getDerivedStateFromProps`中实现响应式更新
```js
if (Object.prototype.hasOwnProperty.call(nextProps.ueditorOptions, 'serverExtra')) {
  let serverExtraStr = JSON.stringify(nextProps.ueditorOptions.serverExtra);

  if (serverExtraStr === prevState.serverExtraStr) {
    return {
      ...prevState,
      content: value
    };
  }
  editorReady && editorReady.then((ueditor) => {
    ueditor.setExtraData && ueditor.setExtraData(nextProps.ueditorOptions.serverExtra);
  });
  return {
    ...prevState,
    serverExtraStr,
    content: value
  };
}
```
---

以上便是ueditor改造和封装中最核心的内容，下面简单介绍一下应该如何使用`react-ueditor-component`，详细的使用教程请看[readme.md](https://github.com/Eschere/react-editor-component#readme)，项目源码中也提供了完整的`demo`，`App.js`(不使用`react-ueditor-component`)、`OwnServer.js`(使用`react-ueditor-component`上传到自己的服务器)、`QiniuServer.js`(使用`react-ueditor-component`对接七牛OSS)。

### 安装和引入
安装组件
```bash
yarn add react-ueditor-component --save
```

下载修改后打包的[ueditor.zip](https://github.com/Eschere/react-editor-component/raw/master/assets/utf8-php.zip)，或者找到`node_modules/react-ueditor-component/assets/utf8-php.zip`，解压文件，放在网站的根目录，react项目一般放在`public`文件夹下，
`index.html`中`script`标签引入`ueditor`代码
```xml
<script src="/utf8-php/ueditor.config.js"></script>
<script src="/utf8-php/ueditor.all.js"></script>
```

#### 如果你只需要编辑功能
```jsx
import ReactUEditorComponent from 'react-ueditor-component';

export default class App extends React.Component {
  state = {
    value: ''
  }

  onChange = (value) => this.setState(value);

  render () {
    <div>
    <ReactUEditorComponent
      value={this.state.value}
      onChange={this.onChange}
    />

    {/* 配合antd的form */}
    {
      this.props.form.getFieldDecorator('content')(
        <ReactUEditorComponent />
      )
    }
    </div>
  }
}
```

#### 如何使用`beforeUpload`钩子
通常对接第三方OSS需要获取上传凭证，这就需要用到`beforeUpload`钩子

```js
export default class App extends React.Component {
  state = {
    value: '',
    serverExtra: {
      // 上传文件额外的数据
      extraData: {}
    }
  }

  beforeUpload = file => new Promise((resolve, reject) => {
    let key = 't' + Math.random().toString().slice(5, 16);

    // 请求服务器，获取七牛上传凭证
    fetch('getuploadtoken.com', {
      headers
    })
      .then(response => response.json())
      .then((data) => {
        // 设置七牛直传额外数据
        this.setState({
          serverExtra: {
            extraData: {
              token: data.token,
              key
            }
          },
          // 设置额外数据完成会触发`setExtraDataComplete`
          setExtraDataComplete: () => {
            resolve(file);
          }
        });
      });
  })

  onChange = (value) => this.setState(value);

  render () {
    return (
      <ReactUEditorComponent
        value={this.state.value}
        onChange={this.onChange}
        // 必须在state中
        setExtraDataComplete={this.state.setExtraDataComplete}
        ueditorOptions={{
          beforeUpload: this.beforeUpload,
          // 上传文件时的额外信息
          serverExtra: this.state.serverExtra,
          serverUrl: 'http://qiniuupload.com' // 上传文件的接口
        }}
      />
    )
  }
}
```

希望以上轮子有朝一日对你有所帮助，欢迎提供技术支持，或者加入我们 yemao@talkmoney.cn

作者简介：叶茂，芦苇科技web前端开发工程师，代表作品：口红挑战网红小游戏、服务端渲染官网、微信小程序粒子系统。擅长网站建设、公众号开发、微信小程序开发、小游戏、公众号开发，专注于前端领域框架、交互设计、图像绘制、数据分析等研究。 一起并肩作战： yemao@talkmoney.cn 访问 www.talkmoney.cn 了解更多