---
layout: post
title: "DB2 Common Commands"
categories: tech
tags: db2
date: 2016-09-20 17:49:26
---


## Common Commands

### 1. Connect to database(default port: 50000)

```
DB2 CONNECT TO db_test user username using password
```

### 2. Database Operation

```
   # Shutdwon the connection
   DB2 CONNECT RESET

   #终止数据库运行的命令并断开数据库连接:
   DB2 TERMINATE

   # list DB2 instance
   DB2ILIST

   # 列出DB2当前实例
   DB2 GET INSTANCE

   #列出当前活动的数据库
   DB2 LIST ACTIVE DATABASES

   #列出所有系统数据库目录
   DB2 LIST DATABASE DIRECTORY
 
   # 列出所有DBMS节点目录
   DB2 LIST NODE DIRECTORY
 
   # 列出应用程序信息
   DB2 LIST APPLICATIONS
 
   # 列出详细信息
   DB2 LIST APPLICATIONS SHOW DETAIL
 
   # 列出指定数据库的应用程序连接
   DB2 LIST APPLICATIONS FOR DB db_test

   #列出指定数据库的应用程序连接的详细信息
   DB2 LIST APPLICATIONS FOR DB PjjTest SHOW DETAIL

   #列出DB2分区信息
   DB2 LIST DATABASE PARTITION GROUPS
 
   # 列出详细信息
   DB2 LIST DATABASE PARTITION GROUPS SHOW DETAIL
 
   # 列出数据库表空间
   DB2 LIST TABLESPACES
 
   # 列出详细信息
   DB2 LIST TABLESPACES SHOW DETAIL
 
   # 列出 DB2 命令行处理器选项设置
   DB2 LIST COMMAND OPTIONS

   # 修改数据库密码命令
   # 最好不要通过修改操作系统更改密码。而使用CONNECT 命令
   CONNECT TO database-alias
   [USER username [{USING password
   [NEW new-password CONFIRM confirm-password] |
   CHANGE PASSWORD}]]
   这里用的是system user pass, type in
   $ passwd username

   $ su - $DB2USER -c "db2 ATTACH TO dbserver USER $DB2USER USING $DB2PASSWORD ; $1"
   "db2 drop database $APP"

   $ netstat -an | grep LISTEN |grep 50001
   db2 get dbm cfg | grep SVC
   db2set -all
   db2 get dbm cfg | grep -i authentication

   # db2 limit impl:
   SELECT * FROM TABLE1 FETCH FIRST 100 ROWS ONLY

   # db2 list application [show detail which applications have open connections]
```

### 3. SQL Operations

```
   # Check DB2 verison
   select * from sysibm.sysvERSIONS
   #create database
   DB2 CREATE DATABASE db_test
   # delete database
   DB2 DROP DATABASE db_test
   # 快速建表: Create sql语句文件

	IF isvalid(gpl3) then 
	   drop table gpl3;
	end if
	create table gpl3
	( id integer  not null,
	 zjhm varchar(3) not null,
	 sjbm varchar(1) null,
	primary key(id)
	);
   
   !db2 -tf (sql语句所在文件的路径)

   DB2? # 显示所有DB2命令
　　 DB2?COMMAND # 显示命令信息
　　 DB2?SQLnnnn # 显示这个SQLCODE的解释信息
　　 DB2?DB2nnnn# 显示这个DB2错误的解释信息

   # force stop DB2 server
   DB2STOP FORCE

   # 建库 example
   CREATE DATABASE db_test AUTOMATIC STORAGE YES  ON 'D:\' DBPATH ON 'D:\' USING CODESET GBK TERRITORY CN COLLATE USING SYSTEM PAGESIZE 4096;

   /home/db2inst1> db2set DB2COMM=SSL,TCPIP 
   /home/db2inst1> db2set -all 
   [i] DB2COMM=SSL,TCPIP

   db2 log messages: db2diag.log
```

## gskit change passwd:

```
$ gsk8capicmd_64 -keydb -changepw -db "dbclient.kdb" -pw "oldPassword" -new_pw "newPassword" -stash
# must specify "-stash", if not, Support for one or more communications protocols specified in the DB2COMM environment variable failed to start successfully.
```


## Steps to remove DB2 from Unix/Linux:

**1) Remove DB**

```
   (1)su - db2inst1
   (2)db2 list db directory
   (3)db2 drop db <db name>
   (4)db2 force application all  - Stop all database applications
   (5)db2stop  -  stop db2 database
   (6)db2 terminate - stop db2 instance
```

**2) Remove Instance**

```
	(1)su - root
	(2)cd <db2 dir>/instance
	(3)./db2ilist
	(4)./db2idrop <instance name>
```

**3) Remove das**

```
	(1)su - root
	(2)cd <db2 dir>/instance
	(3)./daslist
	(4)./dasdrop <das user>
```

**4) Uninstall**

```
	(1)su - root
	(2)cd <db2 dir>/install
	(3)./db2_deinstall -a
```

**5) Remove users ( db2inst1,db2fenc1,dasusr1)**

```
	userdel -r <username>
	please lookinto the file /etc/passwd before and after you deleted users
```