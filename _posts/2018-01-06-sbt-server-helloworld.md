---
title: sbt server hello world
excerpt: ""
---
# Contents
{:.no_toc}

* Toc
{:toc}

## 前言
sbt 在今天发布了[v1.1.0 版本](https://github.com/sbt/sbt/releases/tag/v1.1.0)，新增了sbt-server 功能，支持[Language Server Protocol 3.0](https://microsoft.github.io/language-server-protocol/specificatio). 
eed3si9n 写了一个[neovim 插件](https://github.com/eed3si9n/LanguageClient-neovim) 支持sbt-server, 我按照这篇[文档](http://eed3si9n.com/sbt-server-with-neovim) 进行了安装尝试

## 安装
* 安装neovim 插件

  ~~~ vim
  Plug 'autozimu/LanguageClient-neovim', { 'do': ':UpdateRemotePlugins' }
  ~~~

* 下载[sbt-server-stdio.js](https://gist.githubusercontent.com/eed3si9n/0ee26a15218f1d4031b451dd61315d6c/raw/5693fbcafbb9a71f1ac5a9d13ace94df3b091cbc/sbt-server-stdio.js) 到本地，
  例如~/.vim/sbt-server-stdio.js

* 在neovim中添加如下配置

  ~~~ vim
  let g:LanguageClient_autoStart = 1 " 启动后自动连接sbt-server

  let g:LanguageClient_serverCommands = {
       "\ 'scala': ['node', expand('~/.vim/sbt-server-stdio.js')] " 指向sbt-server-stdio.js的本地位置
       "\ }

  nnoremap <silent> gd :call LanguageClient_textDocument_definition()<CR> "将快捷键gd 定义为跳转到声明位置
  ~~~

## 测试

* 声明查找

  * 执行gd 后提示server 无法启动

    vim 插件是通过LanguageClient_serverCommands 建立与sbt-server 的socket 链接，这要求启动vim 时sbt-server 已经启动或在sbt-server 启动后执行LanguageClientStart 命令。
    我在启动sbt后再打开vim就解决了这个问题。

  * 执行gd 后提示Not found

    sbt-server 是通过已编译的文件对声明进行查找，在我第一次执行gd 时整个工程并没有进行compile, 所以出现了Not found 错误。在sbt 中执行compile 后终于可以正确跳转啦。

  * 在测试代码中执行gd 无效

    当我在测试代码中执行gd 时仍然提示Not found，我发现sbt compile 只会编译功能代码，测试代码只会在sbt test时编译，可能是这个原因导致的gd 无效，需要进一步定位解决。

* 错误提示

  当我编辑完源文件并保存后，sbt 会自动进行compile，也可以在vim 中显示编译错误，并给出错误的行和列

## 评价

从功能上讲sbt-server 很强大，可以支持目前IDE 的大多数功能。但鉴于刚刚发布，vim 插件还有待完善，和ensime 在使用体验上还有一定差距。
