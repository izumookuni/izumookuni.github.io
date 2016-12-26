---
layout: post
title: 用Sublime Text运行Scala脚本
date: 2016-12-26 17:11:45 +08:00
tags: 
    - Scala
    - Scala Script
    - Sublime Text
---

最近在学Python，偶然发现Scala 也可以用来写脚本（Script）。

众所周知，Scala是基于Java的，对依赖包的处理很是麻烦，需要在`scala -cp`后面加入依赖包的路径，不然运行时会报错。但每次运行Scala脚本都要输入一次依赖包路径，又有点得不偿失，违背了使用Scala脚本的初衷。

Sublime Text是一个代码编辑器，也是HTML和散文先进的文本编辑器，具有漂亮的用户界面和强大的功能，例如代码缩略图，Python的插件，代码段等。还可自定义键绑定，菜单和工具栏。（百度百科原话）

Sublime Text是Windows平台上少有的默认支持Scala语法高亮的文本编辑器，非常适用于日常的Scala脚本编写。

下面就开始介绍如何使用Sublime Text来运行Scala脚本。

---

####配置

默认已经安装好Sublime Text和Scala。

在Scala 的安装目录`%SCALA_HOME%\bin`下新建一个文件，命名为`scalas.bat`（也可以起别的名字，但后缀名必须为`.bat`），输入以下代码：

    @echo off
    SetLocal EnableDelayedExpansion
    cd %~dp1
    set file=.
    for /f "delims=" %%i in ('"dir /a/s/b/on *.jar"') do (
      set file=!file!;%%~fi
    )
    scala -cp "!file!" %~nx1

打开Sublime Text，点击`Tools`->`Build System`->`New Build System...`，在打开的文件中输入以下代码：

    {
      "shell_cmd": "scalas \"$file\"",
      "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
      "selector": "source.scala"
    }

`scalas`对应之前的`scalas.bat`。

将文件保存为`Scala Script.sublime-build`（或任意喜欢的名字，但后缀名必须为`.sublime-build`）。

自此，配置完成。

####运行

编写Scala脚本并保存，然后将运行脚本所需的依赖包放到Scala脚本所在的目录下（建议在目录下新建一个`lib`文件夹，把所有依赖包都放在里面，方便管理）。

在Sublime Text中选择`Tools`->`Build System`->`Scala Script`，然后点击`Tools`->`Build`（快捷键`Ctrl + B`），系统开始运行Scala脚本。

完毕。

---

理论上这个方法也可以作为配置其他语言的参考，具体代码可以自己研究。
