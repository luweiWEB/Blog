# 你可能不知道的一些css小知识

> css是一门功能强大、具备完整生态的复杂语言。它拥有很多的技巧， 但是生活工作中我们可能不怎么会接触到，这包括一些实际上挺实用的技巧。在此，特地列举一些css里比较容被忽略的小知识，希望能对你有所帮助。

## 1.  椭圆的实现

跟圆形的实现一样，这里也是用到`border-radius`属性，但是你可能不知道，`border-radius`是一个简写属性， `border-radius`可以单独为四个角分别设置水平和垂直半径，只要用到一个正斜杠即可。在斜杠前指定前四个值，作为各自的水平半径，在斜杠后指定后四个值作为各自的垂直半径（可以简写）。故椭圆可以如下的代码实现：

```css
div {
    width: 200px;
    height: 300px;
    border: 1px solid #000;
	border-radius: 150px / 100px;  
}
```

```html
<div></div>
```

这样子可以直接得到一个椭圆。实际上，它还支持百分比值，所以`border-radius`属性部分的代码还可以简化成这样子：

```css
border-radius: 50%;
```

那如果仅需要实现一个上面部分的半椭圆呢？由前面的结论，实际上我们可以很容易地写出半椭圆的`border-radius`属性部分的css代码：

```css
border-radius: 50% / 100% 100% 0 0;
```

这等价于：

```css
border-radius: 50% 50% 50% 50% / 100% 100% 0 0;
```

结果如下：

![](https://s2.ax1x.com/2019/08/04/ecAvkj.png)



## 2.  绝对定位和固定定位的元素可通过同时设置bottom&top或者left&right的值，来间接地设置该元素的宽度

代码如下：

```css
.parent {
    position: relative;
    border: 1px dashed red;
    height: 100px;
    width: 100px;
}
.child {
    position: absolute;
    bottom: 10px;
    height: 10px;
    left: 10px;
    right: 10px;
    background: gray;
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>
```

结果如下：

![](https://s2.ax1x.com/2019/08/04/eceFnf.png)

实际上，就算在父元素的宽高不是直接指定、而是由其内嵌的内容撑起来的情况下，也可以生效。

## 3.  一般情况下固定定位的父元素，无论其层级设置得多高且无论其内嵌的子元素的层级设置得多低，该元素都不能将这些子元素覆盖

老规矩，上代码示例：

```css
.parent {
    position: fixed;
    z-index: 999;
    background: red;
    height: 100px;
    width: 100px;
}
.child {
    position: relative;
    z-index: -999;
	width: 50px;
    height: 50px;
    background: gray;
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>

```

老规矩，贴结果图：

![](https://s2.ax1x.com/2019/08/04/ecmTL6.png)

## 4. 运用currentColor

 `currentColor`是一个比较特殊的属性，它是css的第一个变量。顾名思义，它并没有绑定具体的颜色值，而是在使用中被解析为color。例如以下的水平分割线的颜色值：

```css
div {
    color: red;
    border: 1px solid red;
}
hr {
    height: .5em;
    background: currentColor;
}

```

```html
<div>
    <h1>My First Heading</h1>
    <hr>
    <p>My first paragraph.</p>
</div>

```

结果如下，水平分割线成功被赋予了同样的颜色值：

![](https://s2.ax1x.com/2019/08/04/ecKzl9.png)

## 5. 利用flexbox和外边距实现垂直水平居中

实现一个元素垂直水平居中的方法有很多，但是以下这种算是比较好的一种方法，十分简捷。

只需在父类和子类分别加上这两个属性：

```css
.parent {
	display: flex;   	
}
.child {
	margin: auto;
}

```

```html
<div class="parent">
    <div class="child"></div>
</div>

```

即可实现子元素垂直水平居中。

## 6. 绝对定位的元素设置为行内元素时不会生效

不少人以为绝对定位的元素可以随便设置其`display`属性，实际上，绝对定位的元素会形成一个块级框，它无法被设置为一个行内元素。你可以对一个元素同时设置以下样式试试效果：

```css
div {
    display: inline;
    position: absolute;
    width: 200px;
    height: 200px;
    background: red;
}

```

会发现，给他设置的宽高依旧会生效。这已经说明了问题。

## 7. 更好地实现文本两端对齐的效果

相信一些人可能会在相应的样式类中设置如下属性：

```css
text-align: justify;

```

但这里的效果是不好的，容易造成单词间间距过大的现象，影响阅读。

我们可以仅仅在相应的样式类中设置如下属性，达到期待中的效果：

```css
hyphens: auto;

```

它会在末尾进行单词的断层，大大缩短了单词间的过大间距。

## 8. 实现波浪形下划线

我们将借助`background`这个简写属性来实现它。可先试着写如下代码：

```css
p {	
	font-size: 30px;
    display: inline;
    background: linear-gradient(red 0 100%) no-repeat;
    background-size: 100% 1px;
    background-position: 0 85%;
}

```

```html
<p>
    This is my first paragraph. This is my first paragraph. This is my first paragraph.
</p>

```

会看到结果如下：

![](https://s2.ax1x.com/2019/08/05/ecJKgO.png)

这并不是我们想要的结果。首先它穿过了字形的descender。我们应该让下划线在遇到字母时自动断开，那样子的效果将会更加舒服。可以巧妙地利用`text-shadow`属性实现这一步，在上面的样式类里加上这样一个属性段：

```css
    text-shadow: 2px 0 white, -2px 0 white;

```

结果如下所示：

可看到效果变成了这样：

![](https://s2.ax1x.com/2019/08/05/ecJVER.png)

怎样实现类似文本输入纠错时的波浪型下划线？这里需要用到两层渐变，最终的代码如下：

```css
p {	
	font-size: 30px;
    display: inline;
	background: linear-gradient(-45deg, transparent 40%,  red, red 60%, transparent 0) 0 1em, linear-gradient(45deg, transparent 40%,  red, red 60%, transparent 0) .1em 1em;
	background-repeat: repeat-x;
	background-size: .2em .1em;
	text-shadow: 2px 0 white, -2px 0 white;
}

```

实现的效果如下：

![](https://s2.ax1x.com/2019/08/05/ec63NT.png)

附： 部分内容源自Lea Verou大神的《CSS揭秘》一书。



![](https://s2.ax1x.com/2019/08/05/ec6dD1.gif)

【作者简介】 TRIS，芦苇科技web前端开发工程师，喜欢唱歌、看动漫、看小说。擅长微信小程序开发，系统管理后台。访问www.talkmnoney.cn了解更多。