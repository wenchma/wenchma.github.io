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


A Tap interface is a software-only interface. Instead of passing frames to and from a physical Ethernet card, the frames are read and written by a user space program. The kernel makes the Tap interface available via the /dev/tapN device file, where N is the index of the network interface.

A Macvtap interface combines the properties of  these two; it is an virtual interface with a tap-like software interface. A Macvtap interface can be created using the ip command:

```bash
$ sudo ip link add link eth0 name macvtap1 type macvtap mode vepa | bridge | private | passthru

$ sudo ip netns exec ns1 ip link show macvtap1
9: macvtap1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN 
mode DEFAULT group default qlen 500
    link/ether fe:4d:6a:7e:2c:de brd ff:ff:ff:ff:ff:ff
```

The device file corresponding to the new macvtap interface with index 9 is /dev/tap9. This device file is 
created by udev.

```bash
$ ls -l /dev/tap9 
crw------- 1 root root 248, 1  4月  6 13:09 /dev/tap9
```

A user space program can open this device file and use it to send and receive Ethernet frames over it. When the kernel transmits a frame via the interface macvtap1, instead of sending it to a physical Ethernet card,  it makes it available for reading from this file by the user space program. Correspondingly, when the user space program writes the content of an Ethernet frame to the file /dev/tap9, the kernel’s networking code sees the frame as if it had been received via the device macvtap1.

Macvtap is implemented in the Linux kernel, and must be configured when compiling the kernel, either as a module or as a built-in feature. The setting can be found under Device Drivers → Network device support → MAC-VLAN based tap driver. The tap driver is dependent on `MAC-VLAN support` in the same category, so you need to enable that too.

## MacVTap modes

A Macvtap device can function in one of four modes: Virtual Ethernet Port Aggregator (VEPA) mode, Bridge mode, Private mode, and passthru mode. The modes determine how the tap endpoints communicate between each other.

### 1. Virtual Ethernet Port Aggregator mode

In this mode, which is the default, data between endpoints on the same lower device are sent via the lower device (Ethernet card) to the physical switch the lower device is connected to. This mode requires that the switch supports ‘Reflective Relay’ mode, also known as `Hairpin` mode. Reflective Relay means the switch can send back a frame on the same port it received it on. Unfortunately, most switches today do not yet support this mode.

![](/img/hairpin.jpg)


### 2. Bridge mode

When the MacVTap device is in Bridge mode, the endpoints can communicate directly without sending the data out via the lower device. When using this mode, there is no need for the physical switch to support Reflective Relay mode.

![](/img/bridge.jpg)


### 3. Private mode

In Private mode the nodes on the same MacVTap device can never talk to each other, regardless if the physical switch supports Reflective Relay mode or not, because the broadcast and multicast  traffic are dropped by MacVTap device. Use this mode when you want to isolate the virtual machines connected to the endpoints from each other, but not from the outside network.

![](/img/private.jpg)


### 4. passthru mode
 
This feature attaches a virtual function of a SRIOV(Single root I/O virtualization) capable physical NIC directly to a VM without losing the migration capability.  The traffic processing in kernel MacVTap is skipped, hardware takes the processing instead, so the Host CPU resources are released.  All packets are sent to the VF/IF of the configured network device. SRIOV capable NIC supports MacVTap passthrough and PCI passthrough. MacVTap passthrough works only for MacVTap net device, PCI passthrough works for any PCI device, aims for Guest OS directly using Host PIC hardware for efficiency. Depending on the capabilities of the device additional prerequisites or limitations may apply; for example, on Linux this requires kernel 2.6.38 or newer.
