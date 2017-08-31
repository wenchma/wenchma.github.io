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