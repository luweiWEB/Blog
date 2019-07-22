#  [wxs ](<https://developers.weixin.qq.com/miniprogram/dev/reference/wxs/>)

### [**官方解释**](<https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxs/>)

1. WXS 与 JavaScript 是不同的语言，有自己的语法，并不和 JavaScript 一致。
2. WXS 的运行环境和其他 JavaScript 代码是隔离的，WXS 中不能调用其他 JavaScript 文件中定义的函数，也不能调用小程序提供的API。
3. WXS 函数不能作为组件的事件回调。
4. 由于运行环境的差异，在 iOS 设备上小程序内的 WXS 会比 JavaScript 代码快 **2 ~ 20 倍**。在 android 设备上二者运行效率无差异



#### **使用方法**
---

1. wxs 代码可以写在wxml  文件中 的<wxs>标签内， 或者 以 .wxs 为后缀名的文件内。(`ps: 一般建议写在 .wxs 文件中`）

2. 每个 .wxs 文件 或者 <wxs> 标签都是一个单独的模块， 当我们想在外部引入其中的私有变量或者函数时， 需要 module.exports 实现。

**示例代码：**

1. 首先在tools.wxs 文件中这么编写

```js
 // /pages/tools.wxs
 var foo = "'hello world' from tools.wxs";
 var bar = function (d) {
  return d;
}
 module.exports = {
  FOO: foo,
  bar: bar,
};
module.exports.msg = "some msg";
```

2. 然后在 wxml 页面中引用

```html
<wxs src="./tools.wxs" module="tools" />
<view>{{tools.FOO}}</view>
<view>{{tools.bar(5)}}</view>
<view>{{tools.msg}}</view>
```

3. 页面中会显示



![result](https://upload-images.jianshu.io/upload_images/14442426-c013a23eb7d0103b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

 ### **注意事项**

---

 wxs 跟js 相比还是有很多限制的。 

比如：

- 它**不支持 es6 语法**， 所以我们平常编码过程中使用的 解构， 箭头函数...都是不支持的。

- 定义变量只能用 var 或者不写 代表全局。因为 let  ，cons是 es6 的

- 数据类型 wxs 语法是没有 **symbol  null undefined **的。 其他的数据类型都支持。

  具体都有： 

  - `number` ： 数值
  - `string` ：字符串
  - `boolean`：布尔值
  - `object`：对象
  - `function`：函数
  - `array` : 数组
  - `date`：日期
  - `regexp`：正则

  

  **`判断wxs中的数据类型`**

  我们知道 在 js 中判断数据类型可以用 typeof  && Object.prototype.toString.call（）

  ```js
  typeof undefined === 'undefined'   // true
  typeof true      === 'bollean'    // true
  typeof 25        === 'number'    // true
  typeof 'shit'      === 'string' // true
  typeof { name: 'mars'} === 'object'  // true
  
  // 以及 es6中的Symbol 
  typeof Symbol()  === 'symbol'    // true
  
  
  typeof function a() {} === 'function'  // true
  
  ```

  以上6种数据类型都有与之同名的字符串与之对应。 但是 mull是 不再其中 的

  ```js
  typeof null === 'object'    // true
  ```

  我们知道当 遇到 Array Date Object... 时  typeof 都会识别为 `object`

  此时需要 `Object.prototype.toString.call（）` 

  ![fuck](https://upload-images.jianshu.io/upload_images/14442426-6fa43e4fe1ce2b5f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

  但是在wxs 中 有属性 **`constructor`**  会返回相应数据类型的字符串

  **如图：** 

  ![business](https://upload-images.jianshu.io/upload_images/14442426-9d5aad4431efb51d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

  [更多详细介绍](<https://developers.weixin.qq.com/miniprogram/dev/reference/wxs/06datatype.html>)戳

- `<wxs>` 模块只能在定义模块的 WXML 文件中被访问到。使用 `<include>` 或 `<import>`时，`<wxs>` 模块不会被引入到对应的 WXML 文件中。

- `<template>` 标签中，只能使用定义该 `<template>` 的 WXML 文件中定义的 `<wxs>` 模块。

### **使用场景**
---

> 因为 wxml 的双括号数据绑定中对表达式的支持不够完善，因此我们可以用wxs 来增强wxml 的表达式。 也就是可以在 wxml 中写函数。

接下来讲两个实际的应用场景的例子

1.  展示天气进行数据展示

```js
// index.wxs 
// 湿度判断
humidity: function(h) {
    if (h) {
      return '湿度 ' + h + '%'
    }
    return h
  },
      
  // 风的等级判断
  windLevel: function(level) {
    if (level === '1-2') {
      return '微风'
    } else {
      return level + '级'
    }
  },
      
  // 风的类型
  wind: function(code, level) {
    if (!code) {
      return '无风'
    }

    if (level) {
      level = level.toString().split('-')
      level = level[level.length - 1]
      return code + ' ' + level + '级'
    }
    return code
  },
```



2. 因为后台返回给我们的数据数组是时间戳, 但是要处理成我们想要的时间格式，比如：2019-07-17

一般处理是遍历数组然后对数组中的每个时间戳对象调用时间转化函数。

但是在wxs 中 我们的转换函数可以这么写

```js
// utils.wxs
var formatTime = function (date) {
  var date = getDate(date)
  var year = date.getFullYear();
  var month = date.getMonth() + 1;
  var day = date.getDate();


  return ([year, month, day].map(formatNumber).join("-") + " " + [hour, minute].map(formatNumber).join(':'));
}
var formatNumber = function (n) {
  n = n.toString();
  return n[1] ? n : "0" + n;
}

module.exports = {
  formatTime: formatTime,
}

```

```html
// pages/index/index.html
<wxs src='./utils.wxs' module="utils">
  <block wx:for="{{list}}" wx:key="index"></block>
  <view >{{utils.formateTime(item.time)}}</view>
```

最终实现效果如下：

![图片](https://upload-images.jianshu.io/upload_images/14442426-c5f6f51ee3750102.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





【完】



![joy](https://upload-images.jianshu.io/upload_images/14442426-371a4c33ffd54e4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



【作者简介】 Mars  芦苇科技web前端开发工程师 喜欢 看电影 ，撸铁 还有学习。擅长  微信小程序开发， 系统管理后台。访问 [www.talkmnoney.cn](http://www.talkmoney.cn )了解更多。

作者主页：

[github](<https://github.com/Marszht>)

[segmentfault](<https://segmentfault.com/u/mars_5ad9c4d43eed5>)











