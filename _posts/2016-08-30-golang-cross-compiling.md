---
layout: post
title: "Golang Cross Compiling"
categories: tech
tags: golang
date: 2016-08-30 16:51:23
---

# Prerequisite

在安装Golang的时候, 只是编译了local operating system 需要的库文件; 而需要跨平台交叉编译, 需要在Golang中增加对其他平台的支持,
`make.bash` 就是来做这件事的.

## Environment variables `make.bash` supports

* GOARCH: The target architecture for installed packages and tools. Examples are `amd64`, `386`, `arm`, `ppc64`.
* GOOS: The target operating system for installed packages and tools. Examples are `linux`, `darwin`, `windows`, `netbsd`.
* CGO_ENABLED: Controls cgo usage during the build. Set it to 1 to include all cgo related files, `.c` and `.go` file with "cgo"
  build directive in the build.

## Run make.bash

```bash
$ cd /usr/local/go/src/

$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 ./make.bash

$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 ./make.bash

$ CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 ./make.bash
```

# Cross Compiling

```bash
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build

$ CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build

$ CGO_ENABLED=0 GOOS=windows GOARCH=386 go build
```
