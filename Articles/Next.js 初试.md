---
title: next.js入门  
tag：next.js, react
---

### 序章

#### 服务端渲染

- 服务端渲染（SSR: Server Side Rendering）,html页面由服务器渲染好，客户端请求的是完整的html页面。
- egg，php，jsp等都是良好的服务端渲染技术。

##### 优点

- seo优化。
- 优化首屏加载速度：相比加载单页应用，只需加载当前页面内容，不用加载大量的js。

##### 缺点
- 线上排查bug不能用浏览器控制台查看数据流动。
- 不是前后端分离的最佳实践。

#### SEO
- 搜索引擎优化（SEO：Search Engine Optimization），利用搜索引擎的规则提高网站在有关搜索引擎内的自然排名。
- 为网站提供生态式的自我营销解决方案，让其在行业内占据领先地位，获得品牌收益。
- SEO包含站外SEO和站内SEO两方面。
- 从网站结构、内容建设方案、用户互动传播、页面等角度进行合理规划，从搜索引擎获得更多的免费流量。

### 安装

- 基于react，所以需要同时安装react和react-dom
```cmd
yarn next react react-dom --save
```
- 编写package.json中的script字段
```json
{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

### 目录结构
默认情况下：

- 每个.js文件是一个路由。
- ./page是服务器的渲染索引
- ./static中的文件映射到/static/路由

### 编写Hello World页面
- 编写一个无状态的页面组件

```js
// ./page/index.js

var num = 1
num ++ 
console.log(num)

function click () {
  console.log(num++)
}

export default () => (
  <div onClick={click}>Hello Next.js</div>
)

```
### 开发
```cmd
yarn dev 
yarn dev -p 8080 // 指定端口号
```
- 运行以上hello world页面可以看出，num在服务端终端和浏览器控制台都输出一次，单个js文件中的全局代码是服务端和客户端公用的代码，一次在全局中处理纯服务器逻辑可能会出错，反之亦然。

- 点击div可以打印出num，并且递增，符合react组件逻辑。

### 获取数据和组件生命周期
- 服务端渲染一个常见的业务场景：获取数据库（或其他服务器）数据，数据返回后再将其插入页面中，生成完整页面返回给客户端。
- 前面说过这个逻辑不能写在全局中，因为这里的代码服务器和客户端都会运行，并且这是一个异步过程，肯定要在一个异步函数、或回调函数中运行的逻辑。

#### getInitialProps
- 页面加载时，改方法只会在服务端执行一次。
- 该方法只有在路由切换时，客户端的才会被执行。
- 改方法的返回值已props注入组件，在客户端运行时可以获取数据
- getInitialProps方法的参数的属性包含：
pathname - URL 的 path 部分
query - URL 的 query 部分，并被解析成对象
asPath - 显示在浏览器中的实际路径（包含查询部分），为String类型
req - HTTP 请求对象 (只有服务器端有)
res - HTTP 返回对象 (只有服务器端有)
jsonPageRes - 获取数据响应对象 (只有客户端有)
err - 渲染过程中的任何错误
```js

import {Component} from 'react'

export default class App extends Component {
  static async getInitialProps(obj) {
    console.log(obj)
    console.log('where called')
    var fetch = (url) => {
      return new Promise((res, rej) => {
        res('获取数据然后渲染')
      })
    }

    let response = await fetch('/static/demo.json')
    console.log(response)
    return {response}
  }

  state = {
    num: 1
  }

  add = () => {
    this.setState((state) => {
      return state.num++
    })
  }

  render () {
    return (
      <div>
        <div>{this.props.response}</div>
        <div>{this.state.num}</div>
        <button onClick={this.add}>++</button>
      </div>
    )
  }
}
```
- 以上代码运行后，getInitialProps方法在服务器执行了，在服务端模拟了一次fetch请求，数据返回后渲染页面。
- 可以在浏览器右键选择“查看网页源代码”，查看从服务器传到客户端初始的html页面的内容。
以下是主要部分,可以看出数据是被预先渲染好的。

```html
<div>获取数据然后渲染</div><div>1</div><button>++</button></div></div>
```

### 兼容性
Next.js 支持 IE11 和所有的现代浏览器使用了@babel/preset-env。为了支持 IE11，Next.js 需要全局添加Promise的 polyfill。有时你的代码或引入的其他 NPM 包的部分功能现代浏览器不支持，则需要用 polyfills 去实现。
- polyfills默认配置加入。
- 开发环境中使用es6的新api，Set等，这些新的api是造成低版本浏览器无法运行的根本原因。
- next.js可以作为提高react应用性能，优化react应用首屏加载速度的解决方案，时单页应用做seo优化成为可能。
- next.js提供的脚手架，开发环境搭建简单，开发有‘热更新’加持，开发极为舒服。
- react本来就是构建复杂应用的框架，其大量使用es6特性，兼容性本来就差，next.js不是提高react应用兼容性的解决方案。

【完】 
 
![思考ing](https://user-gold-cdn.xitu.io/2019/6/21/16b77cacb3aa3495?w=670&h=456&f=jpeg&s=88869)
作者简介：叶茂，芦苇科技web前端开发工程师，代表作品：口红挑战网红小游戏、服务端渲染官网、微信小程序粒子系统。擅长网站建设、公众号开发、微信小程序开发、小游戏、公众号开发，专注于前端领域框架、交互设计、图像绘制、数据分析等研究。 一起并肩作战： yemao@talkmoney.cn 访问 www.talkmoney.cn 了解更多

