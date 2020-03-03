+++
author = "Weiran"
title = "使用Git中的submodule模块"
date = "2020-03-02"
summary = "使用git submodule来管理项目中的依赖。"
math = true
categories = [
    "应用"
]
tags = [
    "git",
]
+++

>之前在搭建Hugo博客的时候，想要把项目上传至Github方便多台机器开发，但是遇到了上传修改过的主题文件夹的问题。这是由于git的submodule导致的。

<h2>1. 问题描述</h2>

我的hugo项目的结构是这样的：

/hugo-blog ---> https://github.com/ryotakx/hugo-blog.git

/hugo-blog/themes/hugo-notepadium ---> https://github.com/cntrump/hugo-notepadium.git # submodule主题原作者的仓库

/hugo-blog/public ---> https://github.com/ryotakx/ryotakx.github.io.git

当我试图push hugo-blog的时候，无法一并将我对于主题的改动上传，在github上themes/hugo-notepadium指向了原作者的repository。实际上外部项目并不关心子模块的修改与否，只通过`.gitmodules`拉取子模块的信息。下面记录一下子模块的正确使用方法。

<h2>2. 更新子模块</h2>

1. 当「当前项目下的子模块内容」和「当前项目记录的子模块的版本」不一致的时候，更新到后者的版本：

```bash
git submodule update
```

2. 当前主项目记录的子模块版本没有发生变化而远程分支更新的时候：

```bash
git submodule update --remote
```

亦或者：

```bash
cd submodule-1
git pull origin master
```

当主项目的子项目特别多时，可以使用 git submodule foreach 执行：

```bash
git submodule foreach 'git pull origin master'
```

<h2>3. 子模块内容的变动</h2>

1. 子模块文件夹内的内容发生了没有提交的内容变动：

通常是在开发的时候一并修改了子模块内容，此时在主项目中用git status可以确认这个状态。主项目的git add/commit是不会对子项目产生影响的。这时候应该进入子模块，按照子模块内部的git版本控制来提交代码。当提交完成的时候进入情况2。

```bash
cd submodule-1
git add .
git commit -m "update submodule"
```

2. 子模块文件夹内的内容发生了版本的变化：

此时在主项目中用git status可以确认子模块的commit状态。此时可以在主项目中使用git add/commit 来提交子模块的版本改动，但是实际上提交的改动是子模块的版本信息，而不是子模块的实际内容。

在我的例子中，我对于原作者远程仓库的子模块做了修改，但是修改只在我的本地分支上完成。此时如果将主模块上传github，子模块指向的是一个不存在的版本仓库。因此，合适的做法是一开始就fork一份原作者的仓库。所有子模块的修改push完成之后，再push主项目的修改。主项目中的`.gitmodules`中保存着子项目的远程地址，用于标记对主项目来说的子项目的位置。因此这个值应该和子项目的远程地址保持一致。在我的例子中，应该是我fork过后的项目地址。

<h2>4. Clone包含子模块的项目</h2>

在和远程同步成功之后，如果要clone整个项目的话：

1. 递归克隆整个项目：

```bash
mkdir hugo-blog
git clone https://github.com/ryotakx/hugo-blog.git --recursive
```

2. 先克隆父项目再更新子模块：

```bash
mkdir hugo-blog
git clone https://github.com/ryotakx/hugo-blog.git
git submodule init
git submodule update
```

<h2>5. 参考 </h2>

1. 子模块的使用：[知乎](https://zhuanlan.zhihu.com/p/87053283)
2. git中文文档：[git](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)






