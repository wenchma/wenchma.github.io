---
layout: post
title: "MacVTap Deep Dive"
categories: tech
tags: macvtap
date: 2016-09-07 22:23:19
---

## MacVTap Overview:

`Macvtap` is essentially a combination of the Macvlan driver and a Tap device. 
The Macvlan driver is a separate Linux kernel driver that the Macvtap driver depends on. 
Macvlan makes it possible to create virtual network interfaces that “cling on” a physical network interface.
Each virtual interface has its own MAC address distinct from the physical interface’s MAC address.
Frames sent to or from the virtual interfaces are mapped to the physical interface, which is called the lower interface.

![](/img/vtap1.jpg)
