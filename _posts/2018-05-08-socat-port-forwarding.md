---
layout: post
title: "Socat: port forwarding"
categories: tech
tags: socat
date: 2018-05-08 09:43:43
---

socat is `Socket CAT`, 主要作用是在两个数据流之间建立双向数据通道，支持的协议和链接方式有：IP, TCP, UDP, IPV6, PIPE, EXEC, System, Open, proxy, OpenSSL, Socket...  

## [Download](http://www.dest-unreach.org/socat/download/) or install as follows:

```
# yum install socat

```

## Port forwarding

Forward locahost port to remote host:

```
# socat TCP4-LISTEN:5432,reuseaddr,fork TCP4:172.30.80.155:5432
```