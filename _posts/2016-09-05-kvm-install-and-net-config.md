---
layout: post
title: "KVM安装及网络配置"
categories: tech
tags: kvm
date: 2016-09-05 16:21:20
---

## 说明：

**该文档简要描述**：

1. Ubuntu环境安装KVM； 
2. KVM里安装虚机及网络配置两个问题

**环境**：

宿主机：Ubuntu 12.10-x86_64，虚机：RHEL6.4-x86_64


## 具体步骤：

1. 安装KVM
   (参考链接：http://tech.ddvip.com/2013-01/1359458551189749.html，下面是需要注意的问题)

 
	安装之后，执行下面的命令看KVM是否安装成功：
	kvm-ok
	输出信息：
	INFO: /dev/kvm exists
	KVM acceleration can be used
	如果提示信息为：
	INFO: KVM (vmx) is disabled by your BIOS(KVM [vmx]被你的BIOS禁用)
	HINT: Enter your BIOS setup and enable Virtualization Technology (VT)
	则需要进入的BIOS设置界面，启用虚拟化技术[VT]，设置步骤为：
	进入BIOS后，选择ADVANCED，然后至 PROCESSOR CONFIGURATION进去找到，
	INTEL (R) VIRTUALIZATION TECHNOLOGY ，设置成ENABLE，保存退出 。


	KVM安装好并链接到KVM服务器（sudo virt-manager -c qemu:///system kvmhost）之后，每次打开KVM管理界面时，可以使用命令：virt-manager

 
2. 网络配置
    如果宿主机不做配置，那么KVM管理的虚机将默认采用NAT的方式与宿主机连接。为了使虚机拥有独立的外网ip地址，应采用桥接方式连接，步骤如下：
    
  	a.修改宿主机配置文件 ： /etc/network/interfaces
     原文件内容：
     # interfaces(5) file used by ifup(8) and ifdown(8) 
     auto lo 
     iface lo inet loopback

     添加网桥设置内容，修改后文件：
     # interfaces(5) file used by ifup(8) and ifdown(8) 
     auto lo 
     iface lo inet loopback 

     auto br0 
     iface br0 inet dhcp 
        bridge_ports eth0 
	 bridge_stp off 
        bridge_fd 0 
        bridge_maxage 0

    b. 重启网络：service network restart （或者：sudo /etc/init.d/networking restart）
    重启之后 ifconfig 会发现宿主机已经有了网桥(br0)配置如下：

    ![](img/br0.jpg)

    c. 使用 `brctl show` 查看网桥列表，发现 br0已经和宿主机网卡绑定：

    ![](img/btctl.jpg)

    至此，KVM的虚机已经可以且默认使用网桥的方式与宿主机连接。

3. KVM创建虚机添加多块网卡
	使用 virt-manager，打开管理器，前几步参照：http://tech.ddvip.com/2013-01/1359458551189749.html

	在第五步，要选中 `Customize configuration before install` 选项，再 `Finish`，同时在下面的`Advanced options`选项，可以看到虚机与宿主机是网桥连接。

	![](img/config-vm.jpg)

	点finish之后，进入下面的虚机配置界面

	![](img/setting.jpg)

选择左下角的 Add Hardware，弹窗中选择添加 Network（即添加了一个网卡），配置好后继续安装即可。
虚机安装好后，Toolbar上（右上角），为多个网卡连接网络即可。
