---
title: 新轮子- Copy as Markdown
date: 2020-03-01 23:38 +0800
---
闲来无事就造造轮子，今天为大家介绍一个Chrome插件[Copy as Markdown](https://chrome.google.com/webstore/detail/copy-as-markdown/pcmnmggfchmeohmflkfocnkackgcnlln?authuser=0&hl=en)

# 动机

在网上看资料时经常需要摘抄一些重要的知识点，使用普通的复制粘贴往往得到的是一坨没有任何格式的文字。
而现在的笔记系统大多数都支持Markdown(欢迎使用[idea-note](https://github.com/sjmyuan/idea-note))， 如果从网页复制的内容能够直接转换为Markdown那就太好了。

# 思路

其实我们从网页复制的内容是`txt/html`格式，如果我们能把HTML转换成Markdown那不就完事了么。

经过一番Google后, 我发现了[turndown](https://github.com/domchristie/turndown)。如果我们想得到GitHub格式的Markdown，还需要[turndown-plugin-gfm](https://github.com/domchristie/turndown-plugin-gfm).

剩下的就是在Chrome插件里先把HTML转换成Markdown然后放到粘贴板了，大体步骤如下

![](https://tva1.sinaimg.cn/large/00831rSTly1gceu3g6u2uj30fb0bzt96.jpg)

# 功能

1. 用户可以直接点击工具栏的`Copy as Markdown`图标进行复制
2. 用户可以点击右键菜单中的`Copy as Markdown`进行复制，注意该选项只有在用户选择HTML之后才能看到。
3. 用户可以按`Ctrl+Shift+Y(Windows)`或`Command+Shift+Y(Mac)`进行复制

# 效果

![](https://tva1.sinaimg.cn/large/00831rSTly1gcev4bogejj30ts0mf0y6.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gceu9pdhu9j30vw0gg0yw.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gceuav8gkzj317w096q47.jpg)
