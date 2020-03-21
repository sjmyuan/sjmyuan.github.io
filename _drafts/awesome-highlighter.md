---
title: 山寨版Chrome插件- Awesome Highlighter
---

首先声明Awesome Highlighter是[Super Simple Highlighter](https://chrome.google.com/webstore/detail/super-simple-highlighter/hhlhjgianpocpoppaiihmlpgcoehlhio)的山寨版本。
这里的山寨主要是指想法和部分交互行为，内部逻辑均为本人自己开发(也没处抄，Super Simple Highlighter不开源)

# 动机

在网上学习时经常会有一些知识点需要做笔记，通常我会把它放到Evernote或[idea-note](https://github.com/sjmyuan/idea-note)，时间长了我也记不清楚有没有记过这个笔记了，经常会碰到同一个网页重复细读的情况.
而且记录到其他软件还是没有直接在浏览器操作方便，需要不停的进行切换.

其实我使用过一段时间[Evernote Web Clipper](https://chrome.google.com/webstore/detail/evernote-web-clipper/pioclpoplcdbaefihamjohnefbikjilc)， 很好用，既可以保存整个网页也可以保存网页片段。
但我更倾向于自己拥有自己的数据，现在已全面转向idea-note。再加上这个插件并不能在浏览同一个网页时对已记录的信息进行渲染提示，我就没有继续使用它.

通过一番搜索我发现了[Super Simple Highlighter](https://chrome.google.com/webstore/detail/super-simple-highlighter/hhlhjgianpocpoppaiihmlpgcoehlhio), 非常好用的一个插件，界面也很好看。
但它有两个缺点，一个是它不开源，第二个是这个插件的作者已经不维护了。这就使得我期望的云存储功能注定无法实现。

在大概搞清楚页面highlight的原理后果断决定，山寨它!

# 思路

要做一个highlight插件有两个难点

1. 如何highlight？
2. 如何在页面中对highlight进行定位？
3. 如何删除highlight？

来看看我是怎么解决的

## 如何highlight？

要解决这个问题，我们得先搞清楚highlight是什么。说白了就是把你选中的一段文字高亮，高亮可以通过改变背景颜色或字体颜色或两者都有来实现。

那么我们的问题就变成了三个子问题
1. 如何在页面上定义一段文字？
2. 如何知道用户在页面上选了哪段文字？
3. 如何改变一段文字的字体或背景 

对于第一个问题，chrome里有一个Range的概念，它有四个属性

* startContainer
* startOffset
* endContainer
* endOffset

通过这四个属性我们可以在页面中唯一确定一段文字，说白了就是我知道你开始在哪个element的哪个位置，结束在哪个element的哪个位置，而在页面中element是有顺序的，定位一个element也很容易。

对于第二个问题，`window.getSelection()` 返回的就是一个Range数组，我们很容易就能拿到用户选择的片段

难点在于第三个问题，位于`startContainer`和`endContainer`之间的element，我们只需要用带指定颜色的elememnt替换掉原来的就可以了，但该怎么处理`startContainer`和`endContainer`呢？
这两个element都只有一部分被选择，我们需要先对它们进行一个切分，然后把切分后的在Range内的element进行替换。

## 如何在页面中对highlight进行定位？

## 如何删除highlight？

# 功能

# 效果图
