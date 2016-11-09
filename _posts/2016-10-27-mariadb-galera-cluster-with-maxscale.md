---
layout: post
title: "MariaDB Galera Cluster With MaxScale"
categories: tech
tags: mariadb maxscale
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


## Part I - MariaDB

### Installation

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

### Configuration

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

#### Configuring the Remaining Nodes

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

### Open the firewall for MariaDB on all nodes

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

### Start the cluster

First, stop the MariaDB service on all Three Servers so that cluster can be brought online.

`sudo systemctl stop mysql`

systemctl doesn't display the outcome, so run the following command to make sure we succeeded,

sudo systemctl status mysql

````
Oct 27 14:55:15 node1 systemd[1]: Stopped MariaDB database server.
```

Second, bring up first node, need to use a special startup script ( `galera_new_cluster` ) to start first node.
This script allows systemd to pass the `--wsrep-new-cluster` parameter.
`systemctl start mysql` would fail because there are no nodes running for the first node to connect with.

```
$ sudo galera_new_cluster

node1:~$ mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
Enter password: 
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```

Finally, bring up the remaining two nodes:

**node2**:

```
node3:~$ sudo systemctl start mariadb
node2:~$ mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
Enter password: 
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
```

**node3**:

```
node3:~$ sudo systemctl start mariadb
node3:~$ mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
Enter password: 
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```

verify mariadb cluster memebers:

```
$ mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_incoming_addresses'"
Enter password: 
+--------------------------+----------------------------------------------------------------+
| Variable_name            | Value                                                          |
+--------------------------+----------------------------------------------------------------+
| wsrep_incoming_addresses | 192.168.100.201:3306,192.168.100.202:3306,192.168.100.203:3306 |
+--------------------------+----------------------------------------------------------------+
```

### Adjust the Debian Maintenance User

Currently, Ubuntu and Debian's MariaDB servers do routine maintenance such as log rotation as a special maintenance user.
When we installed MariaDB, the credentials for that user were randomly generated, stored in /etc/mysql/debian.cnf,
and inserted into the MariaDB's mysql database.
When bringing up our cluster, the password from the first node was replicated to the other nodes,
so the value in debian.cnf no longer matches the password in the database. 

So copy first node's `debian.cnf` to the remaining nodes.
The file looks like as follows:

```
# cat /etc/mysql/debian.cnf 
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = Err7Z0NeieDbZW39
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = Err7Z0NeieDbZW39
socket   = /var/run/mysqld/mysqld.sock
basedir  = /usr
```
check the password is correct, if not, update datatbase.

### Verify Replication

On node1, you can create one database and a talbe, then insert into a row data;
insert into another row data on node2 and node3, so verify the data is the same
on all three nodes.


## Part II - MaxScale

### Installation

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

### Configuration

First, create a new MaxScale user, this user will login and get the credentials from the privileges table
(you authenticate against MaxScale technically) and monitor each instance.

```
MariaDB [(none)]> create user 'maxscale'@'%' identified by 'maxscale';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant select on mysql.db to 'maxscale'@'%';
Query OK, 0 rows affected (0.02 sec)

MariaDB [(none)]> grant select on mysql.user to 'maxscale'@'%';
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> grant show databases on *.* to 'maxscale'@'%';
Query OK, 0 rows affected (0.01 sec)
```

If `maxscale` system user doesn't exist, create it first:

```
# groupadd -g 999 maxscale
# grep maxscale /etc/group
maxscale:x:999:
# useradd -g 999 -u 999 maxscale
# grep maxscale /etc/passwd
maxscale:x:999:999::/home/maxscale:
```

Finally below is the configuration /etc/maxscale.cnf:

```
# MaxScale documentation on GitHub:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Documentation-Contents.md

# Global parameters
#
# Complete list of configuration options:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Getting-Started/Configuration-Guide.md

[maxscale]
threads=1

# Server definitions
#
# Set the address of the server to the network
# address of a MySQL server.
#

[node1]
type=server
address=node1
port=3306
protocol=MySQLBackend

[node2]
type=server
address=node2
port=3306
protocol=MySQLBackend

[node3]
type=server
address=node3
port=3306
protocol=MySQLBackend

# Monitor for the servers
#
# This will keep MaxScale aware of the state of the servers.
# MySQL Monitor documentation:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Monitors/MySQL-Monitor.md

[Galera Monitor]
type=monitor
module=galeramon
servers=node1,node2,node3
user=maxscale
passwd=maxscale
monitor_interval=10000

# Service definitions
#
# Service Definition for a read-only service and
# a read/write splitting service.
#

# ReadConnRoute documentation:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Routers/ReadConnRoute.md

#[Read-Only Service]
#type=service
#router=readconnroute
#servers=server1
#user=myuser
#passwd=mypwd
#router_options=slave

# ReadWriteSplit documentation:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Routers/ReadWriteSplit.md

[Read-Write Service]
type=service
router=readwritesplit
servers=node1,node2,node3
user=maxscale
passwd=maxscale
max_slave_connections=100%

# This service enables the use of the MaxAdmin interface
# MaxScale administration guide:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Reference/MaxAdmin.md

[MaxAdmin Service]
type=service
router=cli

[Read-Write Listener]
type=listener
service=Read-Write Service
protocol=MySQLClient
port=4006

[MaxAdmin Listener]
type=listener
service=MaxAdmin Service
protocol=maxscaled
socket=default

```

### Verify your MaxScale

```
# systemctl start maxscale
# systemctl status maxscale
● maxscale.service - MariaDB MaxScale Database Proxy
   Loaded: loaded (/usr/lib/systemd/system/maxscale.service; disabled; vendor preset: enabled)
   Active: active (running) since Wed 2016-11-02 03:40:02 UTC; 13min ago
  Process: 24017 ExecStart=/usr/local/bin/maxscale --user=maxscale (code=exited, status=0/SUCCESS)
  Process: 24011 ExecStartPre=/usr/bin/install -d /var/run/maxscale -o maxscale -g maxscale (code=exited, status=0/SUCCESS)
 Main PID: 24019 (maxscale)
    Tasks: 5
   Memory: 1.7M
      CPU: 761ms
   CGroup: /system.slice/maxscale.service
           └─24019 /usr/local/bin/maxscale --user=maxscale

Nov 02 03:40:02 node1 maxscale[24019]: Loaded module MySQLClient: V1.1.0 from /usr/local/lib/maxscale/libMySQLClient.so
Nov 02 03:40:02 node1 maxscale[24019]: Listening connections at 0.0.0.0:4006 with protocol MySQL
Nov 02 03:40:02 node1 maxscale[24019]: Loaded module maxscaled: V2.0.0 from /usr/local/lib/maxscale/libmaxscaled.so
Nov 02 03:40:02 node1 maxscale[24019]: Listening connections at /tmp/maxadmin.sock with protocol MaxScale Admin
Nov 02 03:40:02 node1 maxscale[24019]: MaxScale started with 1 server threads.
Nov 02 03:40:02 node1 maxscale[24019]: Started MaxScale log flusher.
Nov 02 03:40:02 node1 systemd[1]: Started MariaDB MaxScale Database Proxy.
Nov 02 03:40:02 node1 maxscale[24019]: Server changed state: node1[node1:3306]: new_slave. [Running] -> [Slave, Synced, Running]
Nov 02 03:40:02 node1 maxscale[24019]: Server changed state: node2[node2:3306]: new_master. [Running] -> [Master, Synced, Running]
Nov 02 03:40:02 node1 maxscale[24019]: Server changed state: node3[node3:3306]: new_slave. [Running] -> [Slave, Synced, Running]
```

The `maxadmin` command may be used to confirm that MaxScale is running and the services, listeners etc have been correctly configured.

```
MaxScale> list servers
Servers.
-------------------+-----------------+-------+-------------+--------------------
Server             | Address         | Port  | Connections | Status              
-------------------+-----------------+-------+-------------+--------------------
node1              | node1           |  3306 |           0 | Slave, Synced, Running
node2              | node2           |  3306 |           0 | Master, Synced, Running
node3              | node3           |  3306 |           0 | Slave, Synced, Running
-------------------+-----------------+-------+-------------+--------------------

MaxScale> list services
Services.
--------------------------+----------------------+--------+---------------
Service Name              | Router Module        | #Users | Total Sessions
--------------------------+----------------------+--------+---------------
Read-Write Service        | readwritesplit       |      1 |     1
MaxAdmin Service          | cli                  |      2 |     4
--------------------------+----------------------+--------+---------------

MaxScale> list listeners
Listeners.
---------------------+--------------------+-----------------+-------+--------
Service Name         | Protocol Module    | Address         | Port  | State
---------------------+--------------------+-----------------+-------+--------
Read-Write Service   | MySQLClient        | *               |  4006 | Running
MaxAdmin Service     | maxscaled          | default         |     0 | Running
---------------------+--------------------+-----------------+-------+--------
```

[1]: https://en.wikipedia.org/wiki/Split-brain_(computing)
[2]: https://linux.die.net/man/1/rsync
[3]: https://mariadb.com/kb/en/mariadb-enterprise/building-mariadb-maxscale-from-source-code/