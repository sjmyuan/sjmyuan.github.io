---
title: Python 工程管理
tags:
  - Python
categories:
  - Python Tutorial
---

接触Python也有五六年了，一直都是小打小闹，撑死了也就一个几十行的小脚本，
大型工程的管理也就知道需要一个requirement.txt去管理依赖,从来没有实践过。
前段时间用Python开发了一个图床工具[simple-image-tool](https://github.com/sjmyuan/simple-image-tool)，
在这里分享一下用到的工程管理工具[Poetry](https://python-poetry.org/)(主页很酷炫)

# 简单介绍

当我们谈到工程管理时，通常我们关心的基本需求有

* 依赖管理
* 打包
* 自定义任务

Python也有一些工具可以满足其中一些需求，但显然太复杂了，让我们看看开发Poetry的动机

> Packaging systems and dependency management in Python are rather convoluted and hard to understand for newcomers. Even for seasoned developers it might be cumbersome at times to create all files needed in a Python project: setup.py, requirements.txt, setup.cfg, MANIFEST.in and the newly added Pipfile.
> So I wanted a tool that would limit everything to a single configuration file to do: dependency management, packaging and publishing.

Poetry可以满足我们的这些基本需求，且所有配置都放在一个叫`pyproject.toml`的文件里，个人觉得使用体验和npm/yarn差不多。

# 安装

* 独立安装(推荐)

  ```bash
  curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
  ```

* pip 安装

  ```bash
  pip install --user poetry
  ```

  这种方式会安装poetry的所有依赖，可能会有冲突

安装完成后可以查看版本

```bash
poetry --version
```

也可通过命令行更新版本

```bash
poetry self update
```

如果想要命令自动补全，可以根据自己的shell类型执行下面的命令

```bash
# Bash
poetry completions bash > /etc/bash_completion.d/poetry.bash-completion

# Bash (Homebrew)
poetry completions bash > $(brew --prefix)/etc/bash_completion.d/poetry.bash-completion

# Fish
poetry completions fish > ~/.config/fish/completions/poetry.fish

# Fish (Homebrew)
poetry completions fish > (brew --prefix)/share/fish/vendor_completions.d/poetry.fish

# Zsh
# After this command, add fpath+=~/.zfunc before compinit in ~/.zshrc
poetry completions zsh > ~/.zfunc/_poetry

# Zsh (Homebrew)
poetry completions zsh > $(brew --prefix)/share/zsh/site-functions/_poetry

# Zsh (Oh-My-Zsh)
# After the command, enable poetry plugin in ~/.zshrc
mkdir $ZSH/plugins/poetry
poetry completions zsh > $ZSH/plugins/poetry/_poetry

# Zsh (prezto)
poetry completions zsh > ~/.zprezto/modules/completion/external/src/_poetry
```

# 创建工程

```bash
poetry new poetry-demo
```

执行完这个命令后，我们将得到如下的工程结构

```
poetry-demo
├── pyproject.toml
├── README.rst
├── poetry_demo
│   └── __init__.py
└── tests
    ├── __init__.py
    └── test_poetry_demo.py
```

`pyproject.toml` 是整个工程的配置文件，其内容大体如下

```toml
[tool.poetry]
name = "poetry-demo"
version = "0.1.0"
description = ""
authors = ["Sébastien Eustace <sebastien@eustace.io>"]

[tool.poetry.dependencies]
python = "*"

[tool.poetry.dev-dependencies]
pytest = "^3.4"

[tool.poetry.scripts]
my-script = "my_module:main"
```

* tool.poetry

  工程的基本信息

* tool.poetry.dependencies

  工程发布后需要的依赖

* tool.poetry.dev-dependencies

  工程开发时需要的依赖

* tool.poetry.scripts

  自定义任务，可以通过`poetry run` 执行

如果想要交互式的创建工程，可以执行`poetry init`。


# 依赖管理

* 添加依赖

  ```bash
  poetry add <dependency>[@<version>]

  # poetry add requests@latest
  ```

* 删除依赖
* 锁定版本
* 安装所有依赖
* 只安装production 依赖

# 打包发布

# 自定义任务
