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

### 2. How to create openrc for a tenant

```
$ openstack project create --description "Project for mars" wenchma
$ openstack user create --password 40bb211707b92bf96e3 mars
$ openstack role add --project wenchma --user xtrail user

# update tenant resources quota, -1 means no limit
$ neutron quota-update --tenant-id project_wenchma_id --network -1 --subnet -1 --port -1 --router -1 --floatingip -1 --security-group -1 --security-group-rule -1
$ nova quota-update  project_wenchma_id  --instances -1 --cores -1 --ram -1 --fixed-ips -1
```

### 3. maximum open files limited

* Find open files limit per process: `ulimit -n`
* Count all opened files by all process: `lsof | wc -l` or `cat /proc/sys/fs/file-nr`
* Get maximum open files count allowed per system: `cat /proc/sys/fs/file-max`

Using `ulimit -n` to set per shell based maximum open files limit.
To make this value persistent to edit `/etc/security/limits.conf` and restart the system:
```
*               soft    nofile          10240
*               hard    nofile          10240
root            soft    nofile          10240
root            hard    nofile          10240
```

check maximum open files limit for memcached service:

```
# ps -ef |grep memcached
root     11684  9824  0 08:31 pts/0    00:00:00 grep --color=auto memcached
memcache 22789     1  0 Sep01 ?        00:23:11 /usr/bin/memcached -m 64 -p 11211 -u memcache -l 0.0.0.0
# cat /proc/22789/limits 
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        0                    unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             64115                64115                processes 
Max open files            1024                 1024                 files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       64115                64115                signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us
```

check current open files for memcached service:
```
# lsof -p 22789 | wc -l
291

# ls /proc/22789/fd/ | wc -l
274

```


Memecached conf file:

```
# Run memcached as a daemon.
-d

# memory
-m 64

# Default connection port is 11211
-p 11211

# Run the daemon as root. The start-memcached will default to running as root if no
-u memcache

# Specify which IP address to listen on. The default is to listen on all IP addresses
# This parameter is one of the only security measures that memcached has, so make sure
# it's listening on a firewalled interface.
-l 127.0.0.1

# Limit the number of simultaneous incoming connections. The daemon default is 1024
# -c 1024

# Lock down all paged memory. Consult with the README and homepage before you do this
# -k

# Return error when memory is exhausted (rather than removing items)
-M

# Maximize core file limit
# -r
```