---
layout: post
title: "OpenStack Troubleshooting"
categories: tech
tags: openstack
date: 2017-08-30 18:22:08
---

This post will record the OpenStack troubleshooting, that will benefit the future.


### 1. Unable to ping and ssh instance

This is because the security group blocks the traffic, run the following cmds to add security group rules:

```
$ nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
$ nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
```