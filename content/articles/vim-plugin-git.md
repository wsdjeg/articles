---
title: "使用 git 管理 vim 插件"
date: 2018-03-20T17:53:00+08:00
tags: ["git", "vim" ]
---

vim 有很多**管理插件**的插件，例如 [vundle](https://github.com/VundleVim/Vundle.vim)、[vim-plug](https://github.com/junegunn/vim-plug)、[dein](https://github.com/Shougo/dein.vim) 等等。我最早使用的是 vundle，后来迁移到了 vim-plug 上，中间也试用过 dein，但没有发现亮点。这些插件的**核心功能**就是从 GitHub **下载**或者**更新**插件。而实现这些核心功能并不复杂，git 就是很好的工具。

# 插件的目录结构

所谓的 vim 插件不过一个**特殊的文件夹**，其主要的结构如下：

- **autoload/**	插件公共代码，vim 在**执行 viml 的时候**自动载入。`:h autoload`
- **colors/**	配色主题定义文件。`:h colorscheme`
- **ftplugin/**	专用代码，以**文件类型加下划线**开头，遇到对应文件自动执行。`:h write-filetype-plugin`
- **plugin/**	通用代码，自动执行。`:h write-plugin`
- **syntax/**	语法高亮定义文件。`:h mysyntaxfile`

一般的插件都有 `autload` 和 `plugin` 两个目录，简单的插件可能只有一个 `plugin` 目录，支持多种语言的插件会有一个 `ftplugin` 目录。vim 还支持很多其他功能的目录，大家可以通过 `:h runtimepath` 查看详细的说明文档。

# vim 如何加载插件
在 vim 上执行 `:set runtimepath`，会有以下输出：

```
runtimepath=~/.vim,/usr/share/vim/vimfiles,/usr/share/vim/vim80,/usr/share/vim/vimfiles/after,~/.vim/after
```

这是一个逗号分割的路径列表，每个路径都是一个**插件**。vim 会依次执行相应的 viml 文件，载入各种定义文件。在没有插件管理器的年代，安装插件就是将插件对应的目录复制到 `~/.vim`。但是，直接使用 `~/.vim` 会将不同插件的目录混合到一起，所以很难去更新特定插件（这是不是跟 linux 的软件包管理器很相似呢？）。

这时候，各种插件管理器就粉墨登场了，而它们的实质就是在**修改这个 `runtimepath` 变量**。

# 最简单的插件『管理器』
首先，创建一个 `~/.vim` 目录，将创建 `~/.vim/vimrc` 文件。然后，切换到 `~/.vim` 执行 `git init` 完成准备工作。

编辑 `~/.vim/vimrc` 文件，添加如下内容：

```
for name in systemlist('ls ~/.vim/vendor')
	let path = '~/.vim/vendor/'.name
	execute 'set runtimepath+='.path
endfor
```

这个 `systemlist` 的功能跟 `system` 很像，都是执行 shell 命令，并获取命令的输出结果。但是 `systemclist` 会将返回结果逐行拆分，生成一个 `list` 列表。这个 `ls ~/.vim/vendor` 就是列出 `~/.vim/vendor` 目录下的所有目录名称。接着，脚本遍历这个列表，构造出插件的实际目录。最后，通过 `execute` 执行 `set runtimepath+=$path` 将插件的目录追加到 `runtimepath` 变量中。之后，vim 就会自动加载 `~/.vim/vendor` 下的插件了。

这是一个只有 4 行的插件管理器。其实它不是一个**合格**的管理器，因为它只有**加载功能**，没有**管理功能**。我们用 git 实现这个管理功能。

# 添加插件

```
git submodule add https://github.com/tpope/vim-fugitive vendor/vim-fugitive
```

# 更新插件

```
git submodule foreach git pull origin master
```

# 删除插件

```
git submodule deinit vendor/vim-fugitive/
git rm -rf vendor/vim-fugitive/
```

最后，大家可以使用 `git commit` 提交自己的改动，并使用 `git push` 将自己的配置推送到 GitHub 等在线仓库保存。

# 结语

使用 git 管理 vim 插件具有简单、清晰、不依赖第三方插件的特点。但是，用户需要了解 git 的 **submodule** 概念、需要了解 vim 的 **runtimepath** 概念、需要**手动执行**插件下载后的后续操作（如 `composer update`、`:UpdateRemotePlugins` 等），门槛比较高，只适合有一定基础的 vim 用户。
