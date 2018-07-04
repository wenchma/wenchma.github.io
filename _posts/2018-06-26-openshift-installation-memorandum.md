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

**实现主机相互之间SSH无密钥访问**
```
# ssh-keygen -f /root/.ssh/id_rsa -N ''

# for host in master.example.com \
node1.example.com \
node2.example.com; \
do ssh-copy-id -i ~/.ssh/id_rsa.pub $host; \
done
```
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
template_service_broker : Verify that TSB is running -------------------------------------------- 182.84s
Run health checks (install) - EL ---------------------------------------------------------------- 71.81s
openshift_web_console : Pause for the web console deployment to start --------------------------- 30.10s
openshift_service_catalog : oc_process ---------------------------------------------------------- 12.79s
openshift_hosted : Ensure OpenShift pod correctly rolls out (best-effort today) ----------------- 12.26s
openshift_master : restart master api ----------------------------------------------------------- 10.27s
openshift_excluder : Install docker excluder - yum ---------------------------------------------- 9.84s
restart master api ------------------------------------------------------------------------------ 9.55s
openshift_buildoverrides : Set buildoverrides config structure ---------------------------------- 7.55s
Initialize openshift.node.sdn_mtu --------------------------------------------------------------- 7.12s
openshift_hosted : Ensure OpenShift pod correctly rolls out (best-effort today) ----------------- 6.72s
Gathering Facts --------------------------------------------------------------------------------- 6.28s
openshift_manageiq : Configure role/user permissions -------------------------------------------- 5.76s
openshift_version : Get available RPM version --------------------------------------------------- 5.57s
Gather Cluster facts ---------------------------------------------------------------------------- 4.10s
openshift_node : Update journald setup ---------------------------------------------------------- 3.97s
openshift_master_facts : Set master facts ------------------------------------------------------- 3.75s
tuned : Ensure files are populated from templates ----------------------------------------------- 3.56s
openshift_facts --------------------------------------------------------------------------------- 3.55s
openshift_master : Re-gather package dependent master facts ------------------------------------- 3.42s
```

## Post-Configuration

**1. Create login user on master**

```# htpasswd -c /etc/origin/master/htpasswd admin```

**2. Grant the administrator permission of cluster to user**

```
# oc login -u system:admin

# ocadm policy add-cluster-role-to-user cluster-admin 
```

**3. Access openshift web console**

The hostname of master is accessable in the internal domain, so need to set hostname map in `/etc/hosts`, then  
browse `https://master.exmpale.com:8443`

## Jenkins and OpenShift integration

**Step 1**
Jenkins -> 增加构建步骤 -> Inject environment variables :

```
APP_HOSTNAME=192.168.100.228
APP_NAME= myapp
USER_NAME=jenkins
OSE_SERVER=https://192.168.100.228:8443
CERT_PATH=/var/lib/jenkins/secrets/ca.crt
DEVEL_PROJ_NAME=wenchma
WORKSPACE=/root/.jenkins/workspace
```

**Step 2**

Jenkins -> 增加构建步骤 -> 执行shell -> 输入
```
oc login -u$USER_NAME -p$USER_PASSWD --server=$OSE_SERVER

oc project $DEVEL_PROJ_NAME

#Is this a new deployment or an existing app? Decide based on whether the project is empty or not
#If BuildConfig exists, assume that the app is already deployed and we need a rebuild

BUILD_CONFIG=`oc get bc | tail -1 | awk '{print $1}'`

if [ -z "$BUILD_CONFIG" -o "$BUILD_CONFIG" == "NAME" ]; then

# no app found so create a new one
  echo "Create a new app"
  oc new-app http://192.168.100.226/wench/openshift-example/raw/master/application-template-stibuild.json
 
  echo "Find build id"
  BUILD_ID=`oc get builds | tail -1 | awk '{print $1}'`
  rc=1
  attempts=75
  count=0
  while [ $rc -ne 0 -a $count -lt $attempts ]; do
    BUILD_ID=`oc get builds | tail -1 | awk '{print $1}'`
    if [ "$BUILD_ID" == "NAME" -o "$BUILD_ID" == "" ]; then
      count=$(($count+1))
      echo "Attempt $count/$attempts"
      sleep 5
    else 
      rc=0
      echo "Build Id is :" ${BUILD_ID}
    fi 
  done

  if [ $rc -ne 0 ]; then
    echo "Fail: Build could not be found after maximum attempts"
    exit 1
  fi 
else

  # Application already exists, just need to start a new build
  echo "App Exists. Triggering application build and deployment"
  oc start-build ${BUILD_CONFIG}
  sleep 2
  BUILD_ID=`oc get builds | tail -1 | awk '{print $1}'`
  
fi

echo "Waiting for build to start"
rc=1
attempts=25
count=0
while [ $rc -ne 0 -a $count -lt $attempts ]; do
  status=`oc get -o template build ${BUILD_ID} --template={{.status.phase}}`
  if [[ $status == "Failed" || $status == "Error" || $status == "Canceled" ]]; then
    echo "Fail: Build completed with unsuccessful status: ${status}"
    exit 1
  fi
  if [ $status == "Complete" ]; then
    echo "Build completed successfully, will test deployment next"
    rc=0
  fi
  
  if [ $status == "Running" ]; then
    echo "Build started"
    rc=0
  fi
  
  if [ $status == "Pending" ]; then
    count=$(($count+1))
    echo "Attempt $count/$attempts"
    sleep 5
  fi
done

# stream the logs for the build that just started
oc build-logs $BUILD_ID



echo "Checking build result status"
rc=1
count=0
attempts=100
while [ $rc -ne 0 -a $count -lt $attempts ]; do
  status=`oc get -o template build ${BUILD_ID} --template={{.status.phase}}`
  if [[ $status == "Failed" || $status == "Error" || $status == "Canceled" ]]; then
    echo "Fail: Build completed with unsuccessful status: ${status}"
    exit 1
  fi

  if [ $status == "Complete" ]; then
    echo "Build completed successfully, will test deployment next"
    rc=0
  else 
    count=$(($count+1))
    echo "Attempt $count/$attempts"
    sleep 5
  fi
done

if [ $rc -ne 0 ]; then
    echo "Fail: Build did not complete in a reasonable period of time"
    exit 1
fi



echo "Checking for successful test deployment at $HOSTNAME:30543"
set +e
rc=1
count=0
attempts=100
while [ $rc -ne 0 -a $count -lt $attempts ]; do
  if curl -s --connect-timeout 2 $APP_HOSTNAME:30543 >& /dev/null; then
    rc=0
    break
  fi
  count=$(($count+1))
  echo "Attempt $count/$attempts"
  sleep 5
done
set -e

if [ $rc -ne 0 ]; then
    echo "Failed to access test deployment, aborting roll out."
    exit 1
fi


################################################################################
##Include development test scripts here and fail with exit 1 if the tests fail##
################################################################################
```

**Step 3**
Jenkins -> 构建环境 -> Inject passwords to the build as environment variables

增加`USER_PASSWD` 变量

## Troubleshooting

**1. /etc/docker/certs.d/docker-registry.default.svc:5000/node-client-ca.crt: no such file or directory**

Solution:
```
# cp /etc/origin/master/client-ca-bundle.crt /etc/origin/node/client-ca.crt
```

**2. Run etcdctl in openshift master

```
# export ETCDCTL_API=3 
# export ETCDCTL_KEY=/etc/origin/master/master.etcd-client.key
# export ETCDCTL_CERT=/etc/origin/master/master.etcd-client.crt
# export ETCDCTL_CACERT=/etc/origin/master/master.etcd-ca.crt

# etcdctl -w=table member list
+------------------+---------+-----------------+------------------------------+------------------------------+
|        ID        | STATUS  |      NAME       |          PEER ADDRS          |         CLIENT ADDRS         |
+------------------+---------+-----------------+------------------------------+------------------------------+
| 2824c7191a56f0fa | started | 192.168.100.228 | https://192.168.100.228:2380 | https://192.168.100.228:2379 |
+------------------+---------+-----------------+------------------------------+------------------------------+

# etcdctl -w=table endpoint status
+------------------------------+------------------+---------+---------+-----------+-----------+------------+
|           ENDPOINT           |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+------------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://192.168.100.228:2379 | 2824c7191a56f0fa |  3.2.18 |  8.4 MB |      true |         2 |    1647230 |
+------------------------------+------------------+---------+---------+-----------+-----------+------------+
```