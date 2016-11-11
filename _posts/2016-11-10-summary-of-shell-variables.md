---
layout: post
title: "Shell变量总结"
categories: tech
tags: shell
date: 2016-11-10 16:55:45
---

Shell 编程有丰富的变量使用规则以实现各种各样的需求，这里做了一下总结，以备以后用着方便，内容主要来源于网络和平时工作经验.

**特殊符号意义**：

* $n - $1 the first parameter,$2 the second...
* $# - The number of command-line parameters.
* $0 - The name of current program.
* $? - Last command or function's return value.
* $$ - The program's PID.
* $! - Last program's PID.
* $@ - Save all the parameters. 