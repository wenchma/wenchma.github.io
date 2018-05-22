---
layout: post
title: "OpenShift Deployment"
categories: tech
tags: openshift k8s docker
date: 2018-05-22 17:37:14
---

## OpenShift installation

### preparation

Centos 系统，kernel 版本不能太低, 否则需要升级一下kernel, 下面是我升级的版本,
```
# cat /etc/redhat-release 
CentOS Linux release 7.0.1406 (Core)
# uname -a
Linux i-lwfsocpz 4.16.6-1.el7.elrepo.x86_64 #1 SMP Sun Apr 29 16:50:56 EDT 2018 x86_64 x86_64 x86_64 GNU/Linux
```

升级执行如下,
```
官方源： http://elrepo.reloumirrors.net/kernel/el7/x86_64/RPMS/
# yum install -y http://elrepo.reloumirrors.net/kernel/el7/x86_64/RPMS/kernel-ml-4.16.6-1.el7.elrepo.x86_64.rpm

# rmp -qa | grep kernel
# rpm -e kernel-3.10.0-123.el7.x86_64
# vi /etc/default/grub

修改成 GRUB_DEFAULT=0
# grub2-mkconfig -o /boot/grub2/grub.cfg   //重新编译内核启动文件，以后升级完内核也要执行一次
```