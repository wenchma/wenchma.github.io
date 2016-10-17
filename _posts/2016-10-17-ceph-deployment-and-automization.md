---
layout: post
title: "Ceph Deployment and Automization"
categories: tech
tags: ceph
date: 2016-10-17 15:45:50
---


## Ceph Overview

Ceph 是一个PB级的统一存储集群系统, 支持的存储包括: 

* Object Storage: 有原生的API，而且也兼容Swift和S3的API
* Block Storage: 支持可配置、snapshot, clone
* File System: Posix接口，支持snaphot

Ceph 的底层核心是RADOS(Reliable, autonomous, distributed object storage),
有两个守护进程: Monitor (维护整个Ceph集群的全局状态); OSD (bject Storage Device，提供存储资源).

Ceph 还有着高性能, 高可靠性, 高可扩展性的特点，被许多厂商青睐, 云计算领域被广泛采用.

