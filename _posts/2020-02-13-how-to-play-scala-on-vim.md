---
title: 如何在Vim上把Scala玩的飞起？
date: 2020-02-13 00:02 +0800
---
这篇博客只适合那些已经入坑Neovim的读者. 对于想入坑的读者，也欢迎尝试入坑，挺过刚开始的各种报错，尝试过各种眼花缭乱的插件，最终化繁为简甚至自写插件，你将看到一片新大陆。

# 动机

对于初学者来说，做Scala 开发的首选工具是IntelliJ Idea，其强大的快捷键，索引和重构功能让人爱不释手。

但支持这些功能是有代价的，那就是慢，超出你想象的慢。IntelliJ需要大量的时间来帮你做索引才能让你体验它那强大的功能。
还有另外一个代价只针对某些人，那就是贵，很不幸我本人就是被针对的，如果没有公司的购买，我是掏不起这个钱的。

为了让世界不在等待IntelliJ编译的煎熬中度过，也为了给公司和自己省点钱，我们需要一个IntelliJ的替代方案。

# 方案

我曾经尝试过[Ensime](https://ensime.github.io), 体验并不是很好，现在它已经被[Metals](https://scalameta.org/metals/)替代了，下图就是Metals目前支持的编辑器

![](https://tva1.sinaimg.cn/large/0082zybply1gbtd67ifwtj30jq0lwdhs.jpg)

Metals 只是一个Language Server，要在编辑器上使用必须得有一个对应的插件支持它才行。对于NeoVim来说这个插件就是[coc.nvim](https://github.com/neoclide/coc.nvim).

我不得不在这里给coc.nvim一个大大的赞，它简直是Vim插件里的一颗超新星，重新激活了我对Vim插件的兴趣。其功能十分强大，整合了多数常用的功能，基本可以将其他插件卸载了。

那么下面我们就看看如何在NeoVim搭建基于coc.nvim + Metals的Scala开发环境

首先我们需要安装Metlas的CLI程序

```sh
$ curl -L -o coursier https://git.io/coursier

$ ./coursier bootstrap \
  --java-opt -Xss4m \
  --java-opt -Xms100m \
  --java-opt -Dmetals.client=vim-lsc \
  org.scalameta:metals_2.12:0.8.0 \
  -r bintray:scalacenter/releases \
  -r sonatype:snapshots \
  -o /usr/local/bin/metals-vim -f
```

然后在vimrc里安装vim-scala和coc.nvim插件，这里vim-scala主要用来做代码渲染。

```vim
Plug 'derekwyatt/vim-scala'
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

为了让操作更加便捷，我们需要定义一些快捷键，这里只给出常用快捷键的定义，全部配置请参考[Metals文档](https://scalameta.org/metals/docs/editors/vim.html#installing-cocnvim)

```vim
" Use `[c` and `]c` for navigate diagnostics
nmap <silent> [c <Plug>(coc-diagnostic-prev)
nmap <silent> ]c <Plug>(coc-diagnostic-next)

" Remap keys for gotos
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Remap for do action format
nnoremap <silent> F :call CocAction('format')<CR>

" Search workspace symbols
nnoremap <silent> <space>s  :<C-u>CocList -I symbols<cr>

" Fix autofix problem of current line
nmap <leader>qf  <Plug>(coc-fix-current)
```

插件安装好后打开Vim, 安装Metals 在coc.nvim上的插件

```sh
:CocInstall coc-metals
```

Ok, 环境配置完毕，现在可以愉快的写代码了！

# 使用

## 导入Scala工程

当我们打开`.sbt` 或`.scala`文件时，Scala工程会被自动加载. 当`.sbt`被修改时，工程会被重新加载。
如果想要手动触发，可以在`:CocCommand`中选择命令`metals.build-import`。

![](https://tva1.sinaimg.cn/large/0082zybply1gbu17xhxpig30dc0clkjo.gif)

## 跳转到定义位置

用`gd` 来找到类型定义的位置

![](https://tva1.sinaimg.cn/large/0082zybply1gbu18wz81rg30dc0ccnpg.gif)

## 跳转到使用位置

用`gr` 来找到所有使用当前类型的代码

![](https://tva1.sinaimg.cn/large/0082zybply1gbu193gi6dg30dc0ccx6q.gif)

## 跳转到实现位置

用`gi` 来找到当前类型的所有子类

![](https://tva1.sinaimg.cn/large/0082zybply1gbu1958nv5g30dc0ccu0x.gif)


## 查询类型

用`<space>s` 来查询类型

![](https://tva1.sinaimg.cn/large/0082zybply1gbu199qf3yg30dc0ccx6s.gif)

## 自动导入

用`<leader>qf` 来自动修复当前代码，最常用场景为自动添加`import`

![](https://tva1.sinaimg.cn/large/0082zybply1gbu19dyo5wg30dc0cb4qs.gif)

## 代码格式化

用`F` 格式化当前文件，默认使用scalafmt进行格式化

![](https://tva1.sinaimg.cn/large/0082zybply1gbu19gqy4lg30dc0cb1kz.gif)

## 定位错误

* 下一个错误: `]c`
* 上一个错误: `[c`

![](https://tva1.sinaimg.cn/large/0082zybply1gbu19koen7g30dc0cbkjo.gif)

## 代码补全

通常情况下当你输入`.`时，当前类型的可用方法就会弹出来，我们只需要使用Tab来进行选择并用Enter确定

![](https://tva1.sinaimg.cn/large/0082zybply1gbu1xeyjagg30dc0c3npg.gif)

# 总结

从以上的例子中你会发现，目前Metals在Vim上只支持最基本的代码跳转，代码补全以及代码格式化等功能，对代码重构的支持还比较弱，且无法进行Debug。
但幸运的是重度重构和Debug在Scala代码中发生的概率都很小，Metals完全可以满足你的日常开发需求。
