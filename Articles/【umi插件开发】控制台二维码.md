### 前言
在进行移动端webapp开发时，你是否会想要在真机上调试项目。
下面分析一下本地运行项目时，真机调试需要的步骤和麻烦的点。

1. 你需要将手机和运行项目的电脑连接到同一局域网（连接同一个WiFi即可）。
2. 查看电脑在局域网内的ip地址（`windows`在命令行输入`ipconfig`）。
3. 手机浏览器中输入ip和端口打开项目。

然而局域网内的ip地址不是固定的，项目运行的端口也有可能由于端口占用导致端口不固定，所以存书签可能会失效，输入长串的ip地址和端口会比较繁琐，如果要在多台手机调试兼容问题就更让人生无可恋了。这自然就产生了需求：在运行项目时，在控制台输出二维码，扫码访问项目地址。

### webpack插件
惯例，在造轮子之前去`GitHub`搜索一下有没有已经造好的轮子。果然，我不是第一个有这种需求的人。
[devserver-qrcode-webpack-plugin](https://github.com/li-shuaishuai/devserver-qrcode-webpack-plugin/blob/master/index.js)是一个基于`webpack`的插件。但我平时开发项目使用的`umi`脚手架，但是我们都知道`umi`封装的是`webpack`，所以`webpack`插件也是可以使用的，那我们就试试看行不行。


`umi`使用`chain-webpack`自定义`webpack`配置，安装插件后，在`.umirc.js`添加如下配置即可
```js
import qrcode from 'qrcode-webpack-plugin'

export default {
  chainWebpack(config, { webpack }) {
    config.plugin('qrcode')
      .use(qrcode)
  }
}
```
让我们运行试一下
![umistart](https://user-gold-cdn.xitu.io/2019/7/26/16c2bfb7631e7ef8?w=765&h=648&f=gif&s=135807)

没有我们想要的二维码输出，看来该插件并不兼容`umi`，原因是什么呢，我们查看插件源码
```js
apply(compiler) {
    const devServer = compiler.options.devServer

    if (!devServer) {
      console.warn('webpack-server-qrcode: needs to start webpack-dev-server')
      return
    }

    const protocol = devServer.https ? 'https' : 'http'
    const port = devServer.port
    const _ip = this.getIPAdress()[0]
    const url = `${protocol}://${_ip}:${port}`
    this.printQRcode(url)
  }
```
插件是从`webpack`的编译器实例中的`options`中读取`DevServer`的信息的，然而`umi`中`webpack`及`DevServer`的启动方式与直接使用`webpack`有很大的不同，所以`compiler`实例中并没有`DevServer`的信息，所以该插件无法正常使用。

那么，下面就正式开始造轮子之路吧！

先拆解问题，要实现我们想要的效果，只需要重要的两步：

1. 获取项目运行的地址。
2. 输出包含ip地址和端口的二维码。

### 获取ip和端口
ip地址可以使用`node`的内置模块`os`模块获取，`os`模块中的`networkInterfaces`方法可以获取设备网卡的相关信息，包括IP地址和mac地址等。
```js
// getIp.js
const os = require('os');

module.exports = () => {
  const interfaces = os.networkInterfaces();
  
  for (let interface of Object.values(interfaces)) {
    for(let items of interface) {
      if (items.family === 'IPv4' && !items.internal) {
        // TODO: 可以完善的判断条件，兼容多网卡和不同平台设备
        return items.address;
      }
    }
  }
}
```

对于项目运行的端口，使用过`umi`的都知道，直接从配置中或者环境变量`process.env.PORT`中获取是不准确的，因为`umi`有一套端口选择机制，当指定的端口被占用时，会自动选择其他的的端口运行。`umi`内部使用`detect-port`检查端口，`detect-port`的原理很简单，使用`node`内置`net`模块尝试监听指定的端口，当端口被占用时尝试监听其他端口，在监听成功后马上取消监听，并将端口号返回。

在外部没有好的检查方案，好在`umi`在今年5月的一个更新中，在`afterDevServer`事件中提供了`devServerPort`，这为插件获取端口号提供了简便的接口，具体可查看[issue#2386](https://github.com/umijs/umi/pull/2386)。


插件中获取port的基础逻辑如下：
```js
export default api => {
   api.afterDevServer(({ server, devServerPort }) => {
    console.log(devServerPort)
  });
}
```

### 控制台输出二维码
控制台输出二维码可以使用`qrcode-terminal`，该库的主要逻辑为：根据字符串生成二维码每个位置的颜色信息二维数组。用字符`'\u2588'`表示纵向两个白色，`'\u2580'`表示先白后黑，`'\u2584'`表示先黑后白，对于二维码大图，使用的是`'\033[40m  \033[0m'`和`'\033[47m  \033[0m'`。
想要了解细节的可以去查看其源码[github](https://github.com/gtanner/qrcode-terminal)。

使用它输出二维码的代码如下：
```js
import qrcode from 'qrcode-terminal';

qrcode.generate(`http://${ip}:${port}`, { small });
```

### umi-plugin-qrcode
整合上面两步，[umi-plugin-qrcode插件](https://github.com/Eschere/umi-plugin-qrcode)自然就成型了。获取ip地址其实也不用自己写，有许多js库都有这个功能，实现思路都大同小异，并且考虑了其他边界情况。
```js
import address from 'address';
import qrcode from 'qrcode-terminal';

export default (api, { small = true, once = true } = {}) => {
  let port;
  let ip = address.ip();

  api.afterDevServer(({ server, devServerPort }) => {
    port = devServerPort;
  });

  api.onDevCompileDone(({ isFirstCompile }) => {
    once
      ? isFirstCompile && qrcode.generate(`http://${ip}:${port}`, { small })
      : qrcode.generate(`http://${ip}:${port}`, { small })
  })
}

```

在`umi`中如何使用`umi`插件呢？（不是`webpack`插件）
1. 安装插件
```bash
yarn add umi-plugin-qrcode --dev
```

2. `.umirc.js`中配置插件
```js
// .umirc.js
export default {
  plugins: [
    ['umi-plugin-qrcode', {
      small: false, // 二维码大小，默认small=true
      once: false // 第一次编译完成输出（默认）或者每次编译完成输出
    }]
  ]
}
```

由于上文所说的原因，插件支持的`umi`版本为`2.7.0+`。
运行效果如下：
![template](https://user-gold-cdn.xitu.io/2019/7/26/16c2bfb6affddf0f?w=853&h=803&f=gif&s=138673)





【**作者简介**】：**叶茂**，芦苇科技web前端开发工程师，代表作品：口红挑战网红小游戏、服务端渲染官网、微信小程序粒子系统。擅长网站建设、公众号开发、微信小程序开发、小游戏、公众号开发，专注于前端领域框架、交互设计、图像绘制、数据分析等研究。 一起并肩作战： yemao@talkmoney.cn 访问 www.talkmoney.cn 了解更多。