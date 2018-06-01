---
layout: post
title: "OpenShift Deployment"
categories: tech
tags: openshift k8s docker
date: 2018-05-22 17:37:14
---

## OpenShift installation

### Preparation

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

### Installation

#### Modify docker conf
vi /etc/sysconfig/docker, append the following line:
```
INSECURE_REGISTRY='--insecure-registry 172.30.0.0/16'
```

Account: developer/developer

failed: iptables: Bad rule (does a matching rule exist in that chain?)

```
# systemctl stop firewalld -> restart docker
When firewalld is started or restarted it will remove the DOCKER chain from iptables, preventing Docker from working properly.

# setenforce 0
# tar -zxvf openshift-origin-server-v3.9.0-linux.tar.gz
# cd openshift-origin-server-v3.9.0-191fece-linux-64bit/
# cp k* o* /usr/local/bin/

# openshift start &   (always throw error)
```

**Run in the container**:

(edit /etc/hosts, add master.example.com map 10.35.0.2, default is 127.0.0.1, 登录时老跳到127认证，出错)
```
# oc cluster up --public-hostname=master.example.com
Using nsenter mounter for OpenShift volumes
Using 127.0.0.1 as the server IP
Starting OpenShift using openshift/origin:v3.9.0 ...
OpenShift server started.

The server is accessible via web console at:
    https://master.example.com:8443

You are logged in as:
    User:     developer
    Password: <any value>

To login as administrator:
    oc login -u system:admin
```

