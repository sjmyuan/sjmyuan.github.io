---
title: 山寨版Chrome插件- Awesome Highlighter
date: 2020-03-22 12:00 +0800
---
![](https://tva1.sinaimg.cn/large/00831rSTly1gd2kj5nofkj30c807sa9y.jpg)

首先声明Awesome Highlighter是[Super Simple Highlighter](https://chrome.google.com/webstore/detail/super-simple-Highlighter/hhlhjgianpocpoppaiihmlpgcoehlhio)的山寨版本。
这里的山寨主要是指想法和部分交互行为，内部逻辑均为本人自己开发(也没处抄，Super Simple Highlighter不开源)

# 动机

在网上学习时经常会有一些知识点需要做笔记，通常我会把它们放到Evernote或[idea-note](https://github.com/sjmyuan/idea-note)，时间长了我也记不清楚有没有做过这个笔记了，经常会碰到同一个网页重复细读的情况。
而且记录到其他软件需要不停的进行切换，很不方便。

其实我使用过一段时间[Evernote Web Clipper](https://chrome.google.com/webstore/detail/evernote-web-clipper/pioclpoplcdbaefihamjohnefbikjilc)， 很好用，既可以保存整个网页也可以保存网页片段。
但我更倾向于自己拥有自己的数据，现在已全面转向idea-note。再加上这个插件并不能在浏览同一个网页时对已记录的信息进行渲染提示，我就没有继续使用它了。

通过一番搜索我发现了[Super Simple Highlighter](https://chrome.google.com/webstore/detail/super-simple-Highlighter/hhlhjgianpocpoppaiihmlpgcoehlhio)， 非常好用的一个插件，界面也很好看。
但它有两个缺点，一个是它不开源，第二个是这个插件的作者已经不维护了。这就使得我期望的云存储功能注定无法实现。

所以在大概搞清楚页面Highlight的原理后我果断决定，山寨它！

# 思路

要做一个Highlight插件有几个难点

1. 如何Highlight？
2. 如何在页面中对Highlight进行定位？
3. 如何删除Highlight？

来看看我是怎么解决的

## 如何Highlight？

要解决这个问题，我们得先搞清楚Highlight是什么。说白了就是把你选中的一段文字高亮，高亮可以通过改变背景颜色，字体颜色或两者都有来实现。

那么我们的问题就变成了三个子问题
1. 如何在页面上定义一段文字？
2. 如何知道用户在页面上选了哪段文字？
3. 如何改变一段文字的字体颜色或背景颜色？ 

对于第一个问题，HTML里有一个Range的概念，它有四个属性

* startContainer
* startOffset
* endContainer
* endOffset

通过这四个属性我们可以在页面中唯一确定一段文字，说白了就是我知道你在哪个Element的哪个位置开始，在哪个Element的哪个位置结束，而在页面中Element是有顺序的，定位一个Element也很容易。

对于第二个问题，`window.getSelection()` 返回的就是一个Range数组，我们很容易就能拿到用户选择的片段

难点在于第三个问题，位于`startContainer`和`endContainer`之间的Element，我们只需要用带指定颜色的Element替换掉原来的就可以了，但该怎么处理`startContainer`和`endContainer`呢？
这两个Element都只有一部分被选择，我们需要先对它们进行一个切分，然后把切分后的在Range内的Element进行替换，看下面这个例子

假设我们的页面是这个样子

```html
<li>"This is a Highlight example"</li>
```

我们在页面上选择了`is a Highlight` 这几个单词，那么Highlight后的HTML会变成这个样子

```html
<li>
  "This"
  <mark style="background-color: yellow;">"is a Highlight"</mark>
  "example"
</li>
```

## 如何在页面中对Highlight进行定位？

通过上个问题我们已经知道Range是由两个Element决定的，而我们要对Highlight进行定位就是对Range进行定位，最后演变成要对一个Element进行定位。

通常我们可以通过id或class达到目的，但对于这个场景来说，我们要面对的是任意一个网页，我们无法预知网页中都有哪些Element，更不要说这个网页的Element有没有id或class了。

所以这里我想到了两种解决方法

1. 以Element路径来定位

   这种方法需要记录下来从body到当前Element的路径，这样我们只要重复这个路径就可以找到这个Element，这个也是Super Simple Highlighter使用的方法(我从它的存储结构里反推过来的)

   该方法定位速度很快，缺点就是路径查找算法实现复杂(脑子里模拟一遍后，直接放弃了，哈哈)

2. 以内容来定位

   这种方法就十分简单，我只要记录下来当前Element的内容，然后看整个页面里有相同内容的节点有几个，再看看我要定位的节点排第几。 这样我通过内容和一个index就可以定位这个Element了。

   该方法比较简单直接，缺点就是需要遍历页面里的所有Element，在Highlight较多时速度让人捉急。

这个插件目前实现的是第二种方法，保存Highlight时它会同时保存一个range index用于定位`startContainer`和`endContainer`

```javascript
export interface RangeIndex {
  startNodeIndex: number,
  startOffset: number,
  startNodeContent: string | null,
  endNodeIndex: number,
  endOffset: number,
  endNodeContent: string | null
}
```

## 如何删除Highlight？

有人可能会说删除最简单了，把内存里的Highlight信息删掉不就完了。这里我们需要注意一点，我们每添加一个Highlight，整个页面的结构就会被改变，因为我们要对Element进行切分替换。
而Highlight的定位完全依赖于当前页面的DOM结构，一旦结构改变定位就错乱了。

所以我们不能简单的对Highlight进行删除，我们需要保证页面的修改行为是一致的。这就要求我们只能对页面进行追加修改，不能删掉中间的任何修改。

所以这里对于删除hightlight，我们要在当前最新的页面上去把替换掉的Element再替换回去，看下面的例子

```html
<li>
  "This"
  <mark style="background-color: yellow;">is a Highlight</mark>
  "example"
</li>
```

删除`is a Highlight` 后，DOM结构变成

```html
<li>
  "This"
  "is a Highlight"
  "example"
</li>
```

你会看到DOM是不会恢复到没有加Highlight之前的样子的，因为可能已经有其他的hightlight依赖于`This` 或`example`了。

# 功能

* 选择文字后右键添加Highlight
* 鼠标移到Highlight上后删除Highlight
* 查看当前页面所有的Highlight，并可以复制单条或导出全部
* 重新打开页面后渲染以前的Highlight
* 定制Highlight的显示样式
* 备份所有的配置和Highlight信息到文件
* 从文件恢复所有的配置和Highlight信息

# 效果图

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2j2oxe0wj30zk0m8qb0.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2j2ts7myj30zk0m8jua.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2j2yi9srj30zk0m8tb2.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2j31y6yhj30zk0m8abd.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2j358kpfj30zk0m8wfg.jpg)
