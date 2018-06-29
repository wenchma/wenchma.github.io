---
layout: post
title: "Socat: port forwarding"
categories: tech
tags: socat
date: 2018-05-08 09:43:43
---

socat is `Socket CAT`, 主要作用是在两个数据流之间建立双向数据通道，支持的协议和链接方式有：IP, TCP, UDP, IPV6, PIPE, EXEC, System, Open, proxy, OpenSSL, Socket...  

### [Download](http://www.dest-unreach.org/socat/download/) or install as follows:

```
# yum install socat

```

### Port forwarding

Forward locahost port to remote host:

```
# socat TCP4-LISTEN:5432,reuseaddr,fork TCP4:172.30.80.155:5432
```

TCP4-LISTEN： 本地建立一个TCP ipv4 协议的监听端口  
reuseaddr: 绑定本地一个端口  
fork: 设置多链接模式，即当一个链接建立后，自动复制一个同样的端口再进行监听. 

如果远程访问不了(connection refused), 需要加一条防火墙规则：
```
# iptables -A OS_FIREWALL_ALLOW -p tcp -m state --state NEW --dport 5432 -j ACCEPT

# iptables  -nL OS_FIREWALL_ALLOW --line-number
Chain OS_FIREWALL_ALLOW (1 references)
num  target     prot opt source               destination         
...

12   ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:5432
```
