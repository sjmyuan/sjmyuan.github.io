---
title: Python 如何优雅的处理命令行参数
tags:
- Python
categories:
- Python Tutorial
---

以前一直热衷于使用shell脚本来解决问题，主要看中其简单快速，有各种轮子可以使用.
但是随着要解决的问题越来越复杂，使用shell的便捷性在慢慢降低，
例如复杂字符串生成时引号双引号的转义，变量的使用，以及在脚本重复使用时入参的解析，帮助文档的生成等。

其实之前也使用Python解决过一些问题，当时还是觉得还是比shell重一些，毕竟要遵守更多的语言规则。
但就目前我面临的问题看，已经到了shell的临界点，是时候重拾Python了。

本文中我将介绍一个让我重新使用Python的实际问题及其解决方案。

# 需求 

一个正常的命令行工具都会附带使用说明，遇到没用过的命令，敲一个 `--help` 已成为本能。

那么作为命令行程序的开发者我们该如何生成这个使用说明呢，或者先不说别的，我们该如何识别`--help` 命令呢?

以前使用shell的时候，我通常会这样写

```bash
usage(){
  echo "usage: $0 <command> [<args>] "
  echo ""
  echo "COMMANDS"
  echo "  open              create a new note or open the existion note"
  echo "  git-sync          flush all the notes to git"
  echo "  s3-sync           flush all the notes to given s3 bucket"
  echo "  ls                list all the notes in 3 days"
  echo "  search            serach all the notes which match the given pattern"
  echo "  goto              goto the given project folder"
  echo "  add               add existing file to idea-note"
}

if [ $# -gt 0 ]
then
  COMMAND="$1"
  shift
  case $COMMAND in

    .....

    help)
      usage
      ;;
  esac
else
  usage
  exit 1
fi
```

真的是生写出来的啊，更别说子命令的还要另写，你能理解我修改了功能后要更新使用说明的痛苦么?
想要理解这种痛苦的可以参观一下完整的shell脚本[idea-note](https://github.com/sjmyuan/idea-note/blob/master/bin/idea).

在开发[simple-image-tool](https://github.com/sjmyuan/simple-image-tool)时，我决定停止这种痛苦，使用Python!

对于这个命令行工具，我期望的使用说明如下

* 入口

```sh
$ simple-image-tool --help

usage: simple-image <command> [<args>]

    The most commonly used commands are:
       upload      Upload image to remote server
       browse      Browse all images on remote server


Simple image tool to upload/browse image on remote server.

positional arguments:
  {upload,browse}  Supported subcommand

optional arguments:
  -h, --help       show this help message and exit
```

* 子命令

```sh
$ simple-image-tool upload --help

usage: simple-image upload [-h] [--resolution {480,720,1080,-1}]
                           [--domain DOMAIN] [--open]
                           image bucket

Resize image and upload it to given s3 bucket

positional arguments:
  image                 The image which will be uploaded, if the value is -,
                        the image in clipboard will be used.
  bucket                The s3 bucket which host the image

optional arguments:
  -h, --help            show this help message and exit
  --resolution {480,720,1080,-1}
                        The resolution of uploaded image, default is -1 which
                        means keep the original size
  --domain DOMAIN       The domain of image server, if this value is given,
                        the full image url will be generated
  --open                Open the image in browser after uploaded the image
```

```sh
$ simple-image-tool browse --help

usage: simple-image browse [-h] [--port PORT] domain bucket

Browse all the images in the s3 bucket

positional arguments:
  domain       The domain of image server, if this value is given, the image
               url will use this domain
  bucket       The s3 bucket which host the image

optional arguments:
  -h, --help   show this help message and exit
  --port PORT  The http server port
```

# 解决方案

从上面的需求中我们可以拆分出一些具体的需求点

* 入口命令和子命令都需要支持`--help` 输出说明文档
* 能够生成命令调用方法和说明
* 既需要支持可选参数，也需要支持必填参数
* 能够限制参数的类型和输入范围
* 能够添加参数说明

下面我们看看在Python中如何解决这些问题，注意本解决方案是基于Python 3.7.2 完成的.

这里我们采用的是Python的argparse 库，安装Python是已安装该库，我们只需要导入即可

```python
import argparse
```

## 入口命令
