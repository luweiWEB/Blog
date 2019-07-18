# 通过阅读源码来提高js知识

原文传送门：[《Improve Your JavaScript Knowledge By Reading Source Code》](https://www.smashingmagazine.com/2019/07/javascript-knowledge-reading-source-code/)

![](https://user-gold-cdn.xitu.io/2019/7/18/16c02c8b911b0891?w=200&h=200&f=jpeg&s=7305)

原本作者：Carl Mungazi，是位于伦敦的能源创业公司Limejump的前端开发人员。他花时间深入挖掘所有JavaScript的深度。



> 简介：当你还处于编程生涯的初期阶段时，深入研究开源库和框架的源代码可能是一项艰巨的任务。在这篇文章中，Carl Mungazi分享了他如何克服恐惧并开始使用源代码来提高他的知识和技能。他还使用Redux来演示他如何破坏图书馆。



你还记得你第一次深入挖掘经常使用的库或框架的源代码吗？对我而言，那一刻是我三年前作为前端开发人员的第一份工作。

我们刚刚完成了用于创建电子学习课程的内部遗留框架的重写。在重写开始时，我们花时间研究了许多不同的解决方案，包括Mithril，Inferno，Angular，React，Aurelia，Vue和Polymer。因为我是一个非常初学者（我刚从新闻转向网络开发），我记得每个框架的复杂性让人感到害怕，而不是理解每个框架的工作方式。

当我开始更深入地研究我们选择的框架Mithril时，我的理解增长了。从那以后，我对JavaScript的了解 - 以及一般的编程 - 得到了很大的帮助，我花了很多时间深入研究我每天在工作或我自己的项目中使用的库的内容。在这篇文章中，我将分享一些您可以采用自己喜欢的图书馆或框架并将其用作教育工具的方法。



以下是我第一次阅读代码的介绍是通过Mithril的hyperscript函数

![*我第一次阅读代码的介绍是通过Mithril的hyperscript函数。*](https://user-gold-cdn.xitu.io/2019/7/18/16c02c8b49fed4a5?w=2000&h=1542&f=jpeg&s=197560)



### 阅读源代码的好处

阅读源代码的主要好处之一是增加了你可以学习的知识数量。当我第一次看到Mithril的代码库时，我对虚拟DOM的含义有一个模糊的概念。当我学习完后，我知道虚拟DOM是一种技术，它涉及创建描述用户界面应该是什么样的对象树。然后使用DOM API将该树转换为DOM元素`document.createElement`。通过创建描述用户界面的未来状态的新树，然后将其与旧树中的对象进行比较来执行更新。

我在各种文章和教程中也都阅读过这些内容，它对我很有帮助，并且在我们发布的应用程序的中能够观察它的运作，对我来说非常有启发性。它还告诉我在比较不同的框架时要问哪些问题。例如，我现在会问诸如“每个框架执行更新的方式如何影响性能和用户体验？”等问题，而不是关注GitHub上的Stars。

另一个好处是增加你对优秀应用程序架构的好感和理解。虽然大多数开源项目通常与其存储库遵循相同的结构，但每个项目都各有差异。Mithril的结构是非常平坦的，如果你熟悉它的API，你可以猜测有关文件夹，如代码`render`，`router`和`request`。另一方面，React的结构反映了它的新架构。维护者将负责UI更新（`react-reconciler`）的模块与负责呈现DOM元素（`react-dom`）的模块分开。

这样做的好处之一是，现在开发人员可以通过挂钩来编写自己的[自定义渲染器](https://github.com/chentsulin/awesome-react-renderer)`react-reconciler`。我最近研究过的模块捆绑包Parcel也有像React `packages`这样的文件夹。关键的模块已命名`parcel-bundler`，它包含负责创建捆绑包，启动热模块服务器和命令行工具的代码。



![](https://user-gold-cdn.xitu.io/2019/7/18/16c02c8b49de9f07?w=2000&h=1240&f=jpeg&s=179222)

*不久之后，你正在阅读的源代码将引导您进入JavaScript规范。*



另一个好处 —— 令我感到惊讶的是：你可以更轻松地阅读那些定义JavaScript这门语言如何工作的官方规范。我第一次读规范是在我调查的区别`throw Error`和`throw new Error`（扰流警报是none）。我调查了这个因为我注意到Mithril用于`throw Error`实现它的`m`功能，我想知道使用它是否比`throw new Error`更好。从那以后，我还了解到，逻辑运算符`&&`和`||` [不一定返回布尔值](https://tc39.es/ecma262/#prod-LogicalORExpression)，发现这个 `==` 运算是如何强制转换值的规则 ，以及还有`Object.prototype.toString.call({}) 返回 '[object Object]'`的[理由](http://www.ecma-international.org/ecma-262/#sec-object.prototype.tostring) 。



### 阅读源代码的技巧

有很多方法可以处理源代码。我发现最简单的方法是从您选择的库中选择一种方法并记录调用它时会发生什么。不要记录每一步，而要尝试确定其整体流程和结构。

我最近按这个方式做了`ReactDOM.render`的记录，也因此学到了很多关于React Fiber及其实现背后的一些原因。值得庆幸的是，由于React是一个流行的框架，我在同一个问题上遇到了很多其他开发人员撰写的文章，这让我少走很多弯路。

这次深入探讨还向我介绍了[合作调度](https://developer.mozilla.org/en-US/docs/Web/API/Background_Tasks_API)的概念，`window.requestIdleCallback`方法和[链接列表](https://github.com/facebook/react/blob/v16.7.0/packages/react-reconciler/src/ReactUpdateQueue.js#L10)的[真实示例](https://github.com/facebook/react/blob/v16.7.0/packages/react-reconciler/src/ReactUpdateQueue.js#L10)（React通过将它们放入队列中来处理更新，队列是优先级更新的链接列表）。执行此操作时，建议使用库创建一个非常基本的应用程序。这使得调试时更容易，因为你不必处理由其他库引起的堆栈跟踪。

如果我没有进行深入审查，我将打开`/node_modules`我正在处理的项目中的文件夹，或者我将转到GitHub存储库。当我遇到错误或有趣的功能时，通常会发生这种情况。在GitHub上阅读代码时，一定要确保你正在阅读的代码是最新的版本。你可以通过单击用于更改分支的按钮并选择“tags”来查看具有最新版本标记的提交中的代码。因为代码库和框架一直都在迭代更新，你当然也不希望了解一些可能在下一版本中删除的内容，所以及时留意最新的版本。

另一种复杂点的阅读源代码的方式，我喜欢称之为“粗略一瞥”。在我开始阅读代码的早期，我安装了*express.js*，打开了它的`/node_modules`文件夹并安装了它的依赖。如果看了`README`文件没有给到我很好理解的话，我会直接阅读源码。这样做让我得到了以下这个有趣的发现：

- Express依赖于两个模块，这两个模块合并对象但以非常不同的方式这样做。`merge-descriptors`只添加直接在源对象上直接找到的属性，它还合并不可枚举的属性，同时`utils-merge`只迭代对象的可枚举属性以及在其原型链中找到的属性。`merge-descriptors`使用`Object.getOwnPropertyNames()`和`Object.getOwnPropertyDescriptor()`同时`utils-merge`使用`for..in`;
- `setprototypeof`模块提供了一种设置实例化对象原型的跨平台方式;
- `escape-html` 是一个78行模块，用于转义一串内容，以便可以在HTML内容中进行插值。

虽然这些发现并不能马上运用，但是让你对代码库或框架使用的依赖关系有一个大致的了解。

在调试前端代码时，浏览器的调试工具是你最好的朋友。除此之外，它们可以让你随时中断程序运行并检查当前的状态，再决定跳过函数的执行或进入或退出程序。如果代码已经压缩过，可能无法很好地进行调试。我个人就喜欢把没有压缩的代码复制到`/node_modules`文件夹中的相关文件中，再将其进行解析。

![](https://user-gold-cdn.xitu.io/2019/7/18/16c02c8b4c7ad6e4?w=2000&h=998&f=png&s=248018)



### 案例研究：Redux的Connect连接功能

React-Redux是一个用于管理React应用程序状态的库。在处理诸如此类的流行库时，我首先会搜索一些写过有关其实现的文章。在本案例研究中，我遇到了这篇[文章](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation)。这是阅读源代码的另一个好处。研究阶段通常会引导你阅读这样的文章，这些文章会改善你自己的思考和理解。

`connect`是一个React-Redux函数，它将React组件连接到应用程序的Redux存储。他是怎么做的呢？根据[文档](https://react-redux.js.org/api/connect)，它做了以下事情：

> “...返回一个新的，连接的组件类，它包装你传入的组件。”

看完之后，我会问下列问题：

- 我是否知道函数接受输入的任何模式或概念，然后返回包含其他功能的相同输入？
- 如果我知道任何这样的模式，我将如何根据文档中给出的解释实现这一点？

通常，下一步是创建一个使用的非常基本的示例应用程序`connect`。但是，在这种情况下，我选择使用我们在[Limejump](https://limejump.com/)上构建的新React应用程序，因为我想`connect`在最终将进入生产环境的应用程序的上下文中理解。

我关注的组件看起来像这样：

```javascript
class MarketContainer extends Component {
 // code omitted for brevity
}

const mapDispatchToProps = dispatch => {
 return {
   updateSummary: (summary, start, today) => dispatch(updateSummary(summary, start, today))
 }
}

export default connect(null, mapDispatchToProps)(MarketContainer);
```



它是一个容器组件，包裹着四个较小的连接组件。你在[文件](https://github.com/reduxjs/react-redux/blob/v7.1.0/src/connect/connect.js)中遇到的第一个导出`connect`方法是注释：*connect是connectAdvanced上的外观*。没有走得太远，我们就有了第一个学习时刻：**一个观察立面设计模式的机会**。在文件的末尾，我们看到`connect`导出一个名为的函数的调用`createConnect`。它的参数是一堆已被解构的默认值，如下所示：

```javascript
export function createConnect({
 connectHOC = connectAdvanced,
 mapStateToPropsFactories = defaultMapStateToPropsFactories,
 mapDispatchToPropsFactories = defaultMapDispatchToPropsFactories,
 mergePropsFactories = defaultMergePropsFactories,
 selectorFactory = defaultSelectorFactory
} = {})
```



同样，我们遇到了另一个学习时刻：**导出调用函数**和**解构默认函数参数**。解构部分是一个学习时刻，因为代码编写如下：

```javascript
export function createConnect({
 connectHOC = connectAdvanced,
 mapStateToPropsFactories = defaultMapStateToPropsFactories,
 mapDispatchToPropsFactories = defaultMapDispatchToPropsFactories,
 mergePropsFactories = defaultMergePropsFactories,
 selectorFactory = defaultSelectorFactory
})
```



它会导致发生这个错误。`Uncaught TypeError: Cannot destructure property 'connectHOC' of 'undefined' or 'null'.` 这是因为这个函数没有默认参数可供使用。

**注意**：*有关此内容的更多信息，请阅读David Walsh的文章。根据你对语言的了解，一些学习时刻可能看起来微不足道，因此最好将注意力放在您以前从未见过或需要了解更多信息的事情上。*

`createConnect`它本身在函数体中没有任何作用。它返回一个名为的函数`connect`，我在这里使用的函数：

```javascript
export default connect(null, mapDispatchToProps)(MarketContainer)
```



它需要四个参数，都是可选的，前三个参数都通过一个`match`函数来帮助根据参数是否存在以及它们的值类型来定义它们的行为。现在，因为提供的第二个参数`match`是导入的三个函数之一`connect`，我必须决定要遵循哪个线程。

如果这些参数是函数，用于包装第一个参数的[代理函数](https://github.com/reduxjs/react-redux/blob/v7.1.0/src/connect/wrapMapToProps.js#L29)`connect`，`isPlainObject`用于检查普通对象的实用程序或`warning`揭示如何设置调试器以[中断所有异常](https://developers.google.com/web/tools/chrome-devtools/javascript/breakpoints#exceptions)的模块，有学习时刻。在匹配函数之后，我们来了`connectHOC`一个函数，它接受我们的React组件并将它连接到Redux。它是另一个返回的函数调用，`wrapWithConnect`它实际上处理将组件连接到存储的函数。

看看`connectHOC`它的实现，我可以理解为什么它需要`connect`隐藏它的实现细节。它是React-Redux的核心，包含不需要通过暴露的逻辑`connect`。尽管我原本打算在这个地方结束对它的深度探讨，我也会继续，这也将是我翻查之前发现的参考资料的最佳时机，因为有些资料对对代码库的解释非常详细。



### 摘要

阅读源代码一开始是很困难的，但跟任何事情一样，随着时间的推移它会慢慢变得更容易。我们的目标不是说要理解每一行代码，而是要通过阅读进而得到不同的视角和新的知识。关键是既对整个过程进行深思熟虑，也要对所有事情充满好奇。

例如，我发现`isPlainObject`函数很有趣，因为它使用它`if (typeof obj !== 'object' || obj === null) return false`来确保给定的参数是一个普通的对象。当我第一次阅读它的实现过程，我想知道为什么它没有使用`Object.prototype.toString.call(opts) !== '[object Object]'`，这样会产生更少的代码并能区分对象和对象子类型，比如Date对象。但是，阅读下一行意识到，在开发人员几乎无法用connect去返回一个Date对象，比如，这个是由`Object.getPrototypeOf(obj) === null` 进行检查校验的。

另一个很吸引人的是在`isPlainObject`中的这段代码：

```javascript
while (Object.getPrototypeOf(baseProto) !== null) {
 baseProto = Object.getPrototypeOf(baseProto)
}
```



在谷歌搜索的时候，有些会引导我进入[这个](https://stackoverflow.com/questions/51722354/the-implementation-of-isplainobject-function-in-redux/51726564#51726564) StackOverflow社区 或 Redux [issue](https://github.com/reduxjs/redux/pull/2599#issuecomment-342849867) 查看关于该代码如何处理的案例，例如检查iFrame的对象来源。



#### 另外，还有一些有利于阅读源码的文章

- [《如何通过逆向工程进行框架设计》](https://blog.angularindepth.com/level-up-your-reverse-engineering-skills-8f910ae10630)，Max Koretskyi，Medium
- [《如何阅读代码》](https://github.com/aredridel/how-to-read-code/blob/master/how-to-read-code.md) ，Aria Stewart，GitHub

![Finish](https://user-gold-cdn.xitu.io/2019/1/25/16882a45941d3026?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

【作者简介】：**土拨鼠**，芦苇科技web前端开发工程师，代表作品：飞花亭小程序、续航基因、YY表情红包、YY叠方块直播竞赛小游戏。擅长网站建设、公众号开发、微信小程序开发、小游戏、公众号开发，专注于前端框架、服务端渲染、SEO技术、交互设计、图像绘制、数据分析等研究。

欢迎和我们一起并肩作战： web@talkmoney.cn

访问 www.talkmoney.cn 了解更多


