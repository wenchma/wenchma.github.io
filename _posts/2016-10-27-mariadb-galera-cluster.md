---
layout: post
title: "MariaDB Galera Cluster"
categories: tech
tags: mariadb
date: 2016-10-27 17:01:42
---

## Overview

In order to support high availability to MariaDB database by distributing requests to different servers,
MariaDB provides Galera cluster deployment natively, so this can avoid single point of failure.


Generally, cluster has two modes: active-passive and active-active. 

* In active-passive cluster: all writes are done on a single active node and then copied to one or more passive nodes
  that are poised to take over only in the event of an active server failure.
  Some active-passive clusters also allow `SELECT` operations on passive nodes. 

* In an active-active cluster: every node is read-write and a change made to one is replicated to all.

In this post, we will configure an active-active MariaDB Galera cluster.

## Prerequisites

### Minimal cluster size

In order to avoid a [split-brain][1] condition, the minimum recommended number of nodes in a cluster is 3.
Blocking state transfer is yet another reason to require a minimum of 3 nodes in order to enjoy service availability
in case one of the members fails and needs to be restarted. While two of the members will be engaged in state transfer,
the remaining member(s) will be able to keep on serving client requests.

### rsync

[rsync][2] is the fastest and recommended method for synchronizing data between members,
especially for large datasets since it copies binary data. 
This is used only for the state transfer that happens when a node comes online.
It requires a `READ LOCK` during the whole process.

### Swap Size Requirements

A node maybe go into state transfer failure when it cannot apply writesets during receiving or sending a state transfer.
Thus need to cache the writesets in memory, if the system runs out of memory either the state transfer will fail
or the cluster will block waiting for the state transfer to end, may require manual restoration of the administrative tables.


## Installation

**Environment topology**

```
node1 192.168.100.201
node2 192.168.100.202
node3 192.168.100.203
```

MariaDB 10.1 isn't included in the default Ubuntu 16.04 repositories, so add external repo.
Run the following commands on all the three nodes:

```
$ sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
Executing: /tmp/tmp.j4s70d9kmK/gpg.1.sh --recv-keys
--keyserver
hkp://keyserver.ubuntu.com:80
0xF1656F24C74CD1D8
gpg: requesting key C74CD1D8 from hkp server keyserver.ubuntu.com
gpg: key C74CD1D8: public key "MariaDB Signing Key <signing-key@mariadb.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)

$ sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://nyc2.mirrors.digitalocean.com/mariadb/repo/10.1/ubuntu xenial main'

$ sudo apt-get update

# For server versions prior to 10.1 :
$ sudo apt-get install mariadb-galera-server

# For 10.1 versions :
$ sudo apt-get install mariadb-server

$ sudo apt-get install rsync
```

## Configuration

Add all of the configuration on the first node, then copy it to the others.
By default, MariaDB loads the `/etc/mysql/my.cnf` conf file, and `/etc/mysql/conf.d` directory for additional configuration.
This post, we will modify the default conf file.

```
ubuntu@node1:~$ sudo vi /etc/mysql/my.cnf
```

Add or update the following configuration into the file:

```
[galera]
# Mandatory settings
# WriteSet replication
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_name="wenchma"
wsrep_cluster_address="gcomm://node1,node2,node3"

# Do not change this value, as it affects performance and consistency.
# The binary log can only use row-level replication.
binlog_format=row

# Galera cluster must work with transactional storage engines
default_storage_engine=InnoDB

# Ensure that the InnoDB locking mode for generating auto-increment values
# is set to interleaved lock mode, which is designated by a 2 value.
innodb_autoinc_lock_mode=2

wsrep_sst_method=rsync

# Galera Node Configuration
# IP address and the name of the current server
wsrep_node_address=192.168.100.201
wsrep_node_name=node1

#
# Allow server to accept connections on all interfaces.
#
bind-address=0.0.0.0
```

### Configuring the Remaining Nodes

Copy the configuration from the first node, only update the `Galera Node Configuration` to use the IP address or resolvable domain name of current node.

** node2**:

```
...
# Galera Node Configuration
wsrep_node_address=192.168.100.202
wsrep_node_name=node2
```

** node3**:

```
...
# Galera Node Configuration
wsrep_node_address=192.168.100.203
wsrep_node_name=node3
```

## Open the firewall for MariaDB on all nodes

```
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

Galera can make use of four ports:

* 3306 For MySQL client connections and State Snapshot Transfer that use the mysqldump method.
* 4567 For Galera Cluster replication traffic, multicast replication uses both UDP transport and TCP on this port.
* 4568 For Incremental State Transfer.
* 4444 For all other State Snapshot Transfer.

Open the ports with the following command:

```
$ sudo ufw allow 3306,4567,4568,4444/tcp
$ sudo ufw allow 4567/udp
```

## Start the cluster

First, stop the MariaDB service so that cluster can apply the confand be brought online.

Stop MariaDB on all Three Servers:

`sudo systemctl stop mysql`

systemctl doesn't display the outcome, so run the following command to make sure we succeeded,

sudo systemctl status mysql

`Oct 27 14:55:15 node1 systemd[1]: Stopped MariaDB database server.`


## Installation of MaxScale

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8167EE24
Executing: /tmp/tmp.0eSIWOtBEE/gpg.1.sh --keyserver
keyserver.ubuntu.com
--recv-keys
8167EE24
gpg: requesting key 8167EE24 from hkp server keyserver.ubuntu.com
gpg: key 8167EE24: public key "MariaDBManager" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
$ sudo add-apt-repository 'deb http://downloads.mariadb.com/MaxScale/latest/ubuntu xenial main'
```

用上面的ubuntu repo的方式不能找到deb package, 那就自己[build maxscale from source][3],

```
# install required pkgs
sudo apt-get install git build-essential libssl-dev libaio-dev ncurses-dev \
bison flex cmake perl libtool libcurl4-openssl-dev libpcre3-dev tcl tcl-dev uuid uuid-dev

# Close source codes
git clone https://github.com/mariadb-corporation/MaxScale

mkdir build
cd build

# Configure maxscale build
cmake ../MaxScale -DBUILD_TESTS=Y

# Compile, test and install MaxScale
make
make test
sudo make install
```


[1]: https://en.wikipedia.org/wiki/Split-brain_(computing)
[2]: https://linux.die.net/man/1/rsync
[3]: https://mariadb.com/kb/en/mariadb-enterprise/building-mariadb-maxscale-from-source-code/