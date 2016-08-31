---
layout: post
title: "Management of LVM instance"
categories: tech
tags: 
date: 2016-08-31 15:42:13
---

实验环境，假设本机系统中已有一个磁盘，现增加三个磁盘作LVM。


### 1. 第一次使用LVM时一定要使用vgscan, 以生成/etc/lvmtab和/etc/lvmtab.d.

```
#vgscan
```

### 2. 使用fdisk命令划分VLM分区，分区类型为"8e".

```
#fdisk /dev/sdb
#fdisk /dev/sdc
#fdisk /dev/sdd
```

### 3. 在以上三个磁盘中生Physical Volume。

```
#pvcreate /dev/sdb1
#pvcreate /dev/sdc1
#pvcreate /dev/sdd1
```


### 4. 创建一个Volume Group。

```
#vgcreate Volume Group name（VG1） /dev/sdb1 /dev/sdc1 /dev/sdd1
如果是要扩展一个Volume Group，使用以下命令.
#vgextend Volume Group name（VG1）/dev/sdX
```

# 5. 创建一个 Logical Volume.

```
#lvcreate -L SIZE Volume Group name（VG1）-n Logical Volume name(lv1)
扩展一个现有的Logical Volume:
#e2fsadm -L +SIZE /dev/Volume Group name（VG1）/Logical Volume name(lv1)
```


### 6. 创建文件系统。

```
#mkfs.ext3 /dev/VG1/lv1
```

### 7. 挂载文件系统。

```
#mkdir /mnt/lv
#mount /dev/VG1/lv1 /mnt/lv
```


### 8. 删除一个Logical Volume。

```
1) 先卸载/dev/VG1/lv1的挂载点。
#umunt /dev/VG1/lv1
2) 删除Logical Volume。
#lvremove /dev/VG1/lv1

3) 报错：Unable to deactivate logical volume ，删除失败，可执行
  
＃lsof /dev/VG1/lv1
  
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF      NODE NAME
tgtd            4150 root   11u   BLK  253,2      0t0 255606978 /dev/cinder-volumes/../dm-2
  
#service tgtd stop
```


### 9. 删除一个Volume Group 

在删除Volume Group之前，必须先将Volume Group内的Logical Volume先删
除，然后将Volume Group停止作用，然后再删除Volume Group。

 ```
1、先卸载/dev/VG1/lv1的挂载点。
#umunt /dev/VG1/lv1
2、删除Logical Volume。
#lvremove /dev/VG1/lv1
3、停止Volume Group作用。
#vgchange -a n VG1
4、删除Volume Group。
#vgremove VG1
```
