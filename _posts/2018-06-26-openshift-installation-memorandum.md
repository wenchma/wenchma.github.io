---
layout: post
title: "OpenShift installation Memorandum"
categories: tech
tags: openshift k8s docker
date: 2018-06-26 16:25:15
---

![](/img/os-arch.png)

## Prerequisites

* CentOS - CentOS Linux release 7.5.1804 (Core)
* kernel - 4.17.2-1.el7.elrepo.x86_64
* Docker - 1.13.1
* Opensfhit - 3.9.0
* openshift-ansible -  release-3.9
* ansible >= 2.4.5.0
* hosts - master + node1 + node2

## Installation

**Inventory hosts**:

```
# cat /etc/ansible/hosts
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
ansible_ssh_user=root
ansible_become=true
deployment_type=origin
openshift_release="3.9"
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
openshift_web_console_nodeselector={'region':'infra'}
openshift_docker_options='--selinux-enabled --insecure-registry 172.30.0.0/16'

[masters]
openshfit-1.example.com

[nodes]
openshfit-1.example.com
openshfit-2.example.com openshift_node_labels="{'region': 'infra'}" openshift_node_group_name="node-config-infra"
openshfit-3.example.com openshift_node_labels="{'region': 'infra'}" openshift_node_group_name="node-config-infra"

[etcd]
openshfit-1.example.com

```

**Run ansible playbooks**:

```
# git clone https://github.com/openshift/openshift-ansible.git
# cd openshift-ansible
# git checkout release-3.9

# ansible-playbook playbooks/prerequisites.yml

# ansible-playbook playbooks/deploy_cluster.yml

INSTALLER STATUS *******************************************************************************************************************************************************************************************
Initialization             : Complete (0:00:37)
Health Check               : Complete (0:01:12)
etcd Install               : Complete (0:00:29)
Master Install             : Complete (0:02:10)
Master Additional Install  : Complete (0:00:26)
Node Install               : Complete (0:01:21)
Hosted Install             : Complete (0:00:56)
Web Console Install        : Complete (0:00:39)
Service Catalog Install    : Complete (0:04:30)

Tuesday 26 June 2018  06:47:44 +0000 (0:00:00.042)       0:12:26.216 ********** 
=============================================================================== 
template_service_broker : Verify that TSB is running ---------------------------------------------------------------------------------------------------------------------------------------------- 182.84s
Run health checks (install) - EL ------------------------------------------------------------------------------------------------------------------------------------------------------------------- 71.81s
openshift_web_console : Pause for the web console deployment to start ------------------------------------------------------------------------------------------------------------------------------ 30.10s
openshift_service_catalog : oc_process ------------------------------------------------------------------------------------------------------------------------------------------------------------- 12.79s
openshift_hosted : Ensure OpenShift pod correctly rolls out (best-effort today) -------------------------------------------------------------------------------------------------------------------- 12.26s
openshift_master : restart master api -------------------------------------------------------------------------------------------------------------------------------------------------------------- 10.27s
openshift_excluder : Install docker excluder - yum -------------------------------------------------------------------------------------------------------------------------------------------------- 9.84s
restart master api ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 9.55s
openshift_buildoverrides : Set buildoverrides config structure -------------------------------------------------------------------------------------------------------------------------------------- 7.55s
Initialize openshift.node.sdn_mtu ------------------------------------------------------------------------------------------------------------------------------------------------------------------- 7.12s
openshift_hosted : Ensure OpenShift pod correctly rolls out (best-effort today) --------------------------------------------------------------------------------------------------------------------- 6.72s
Gathering Facts ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 6.28s
openshift_manageiq : Configure role/user permissions ------------------------------------------------------------------------------------------------------------------------------------------------ 5.76s
openshift_version : Get available RPM version ------------------------------------------------------------------------------------------------------------------------------------------------------- 5.57s
Gather Cluster facts -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 4.10s
openshift_node : Update journald setup -------------------------------------------------------------------------------------------------------------------------------------------------------------- 3.97s
openshift_master_facts : Set master facts ----------------------------------------------------------------------------------------------------------------------------------------------------------- 3.75s
tuned : Ensure files are populated from templates --------------------------------------------------------------------------------------------------------------------------------------------------- 3.56s
openshift_facts ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 3.55s
openshift_master : Re-gather package dependent master facts ----------------------------------------------------------------------------------------------------------------------------------------- 3.42s
```
