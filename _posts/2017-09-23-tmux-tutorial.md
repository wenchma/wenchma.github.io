---
layout: post
title: "Tmux Tutorial"
categories: tech
tags: terminal
date: 2017-09-23 07:44:37
---

## Overview

tmux is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen.  
tmux may be detached from a screen and continue running in the background, then later reattached.  
Each `session` has one or more windows linked to it, a `window` occupies the entire screen and may be split into rectangular `panes`.

![](/img/tmux.jpg)

## Installation

For ubuntu:
```
$ sudo apt-get install tmux
```

For Mac:
```
$ brew install tmux
```

## Usage

### Shortcut keys

Control tmux from an attached client by using combination key `Ctrl-b` by default, followed by the following command key:

* Basic

    ? show help info

* Session

    * s 列出所有会话
    * $ 重命名当前的会话
    * d 断开当前的会话

* Windows

    * c 创建一个新窗口
    * , 重命名当前窗口
    * w 列出所有窗口
    * % 水平分割窗口
    * " 竖直分割窗口
    * n 选择下一个窗口
    * p 选择上一个窗口
    * 0~9 选择0~9对应的窗口

* Panes

    * % 创建一个水平窗格
    * " 创建一个竖直窗格
    * h 将光标移入左侧的窗格*
    * j 将光标移入下方的窗格*
    * l 将光标移入右侧的窗格*
    * k 将光标移入上方的窗格*
    * q 显示窗格的编号
    * o 在窗格间切换
    * } 与下一个窗格交换位置
    * { 与上一个窗格交换位置
    * ! 在新窗口中显示当前窗格
    * x 关闭当前窗格> 要使用带“*”的快捷键需要提前配置，配置方法可以参考上文的“在窗格间移动光标”一节。——译者注

* Others

    * t 在当前窗格显示时间
