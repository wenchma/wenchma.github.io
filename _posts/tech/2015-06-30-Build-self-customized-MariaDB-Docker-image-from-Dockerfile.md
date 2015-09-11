---
layout: post
title:  "Build self-customized MariaDB Docker image from Dockerfile"
date:   2015-06-30 15:30:23
categories: tech
tags: docker
---

Docker 可以说是自OpenStack后现在最火的容器技术，可以方便快捷的完成应用程序的打包和部署，本例运行在Ubuntu 14.04 (LTS)虚拟机上，每个运行的容器实例(如nova instances)之间是相互隔离的。
Docker images 是容器技术的基础，Docker有两种方式来更新和构建镜像:

1.  We can update a container created from an image and commit the results to an image by `docker commit` command.
2.  We can use a `Dockerfile` to specify instructions to create an image.

让我们看看如何通过 Dockerfile 构建一个包含mariadb application 的Docker 镜像。

## 1. Install Docker

	$ wget -qO- https://get.docker.com/ | sh

详细的安装步骤可以参考[官方安装文档][docker-installation-on-ubuntu].

## 2. Download Ubuntu base image from [Docker Hub][Docker-Hub]

	$ docker run ubuntu:14.04
	Unable to find image 'ubuntu:14.04' locally
	14.04: Pulling from ubuntu
	428b411c28f0: Already exists 
	435050075b3f: Already exists 
	9fd3c8c9af32: Already exists 
	6d4946999d4f: Already exists 
	Digest: sha256:59662c823007a7a6fbc411910b472cf4ed862e8f74603267ddfe20d5af4f9d79
	Status: Downloaded newer image for ubuntu:14.04
	$ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
	ubuntu              latest              6d4946999d4f        2 weeks ago         188.3 MB
	ubuntu              14.04               6d4946999d4f        2 weeks ago         188.3 MB

This command downloads the specified image and runs it in a container.

## 3. 创建Dockerfile

创建一个工作目录，如 dockerfiles ，然后创建一个名为 Dockerfile 的文本文件，编写内容如下：

	# Sets the base image
	FROM ubuntu:14.04
	
	# Set the Author of the generated image
	MAINTAINER wenchma <wenchma@gmail.com>
	
	# Set the environment variables inside container
	ENV MYSQL_ROOT_PASSWORD mypassword
	ENV MYSQL_DATADIR /var/lib/mysql
	
	# Set the MariaDB public key and repository
	RUN apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
	RUN echo 'deb http://sfo1.mirrors.digitalocean.com/mariadb/repo/10.0/ubuntu trusty main' | tee /etc/apt/sources.list.d/mariadb.list
	
	# update the MariaDB repo and install the packages
	RUN apt-get update
	RUN apt-get install -y -q mariadb-server mariadb-client
	
	# set MariaDB data dir and log dir
	RUN mkdir -p /var/lib/mysql && mkdir -p /var/log/mysql && chown mysql:mysql /var/log/mysql
	
	# set the port in container to expose to host
	EXPOSE 3306

## 4. 构建Docker MariaDB 镜像

	$ docker build -t wenchma/ubuntu:mariadb .
	Sending build context to Docker daemon 20.15 MB
	Sending build context to Docker daemon 
	Step 0 : FROM ubuntu:14.04
	 ---> 6d4946999d4f
	Step 1 : MAINTAINER wenchma <wenchma@gmail.com>
	 ---> Using cache
	 ---> ce6687ba3415
	Step 2 : ENV MYSQL_ROOT_PASSWORD mypassword
	 ---> Using cache
	 ---> 711c5d00c490
	Step 3 : ENV MYSQL_DATADIR /var/lib/mysql
	 ---> Using cache
	 ---> 0aca6d1fa456
	Step 4 : RUN apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
	 ---> Using cache
	 ---> f14b6903246f
	Step 5 : RUN echo 'deb http://sfo1.mirrors.digitalocean.com/mariadb/repo/10.0/ubuntu trusty main' | tee /etc/apt/sources.list.d/mariadb.list
	 ---> Running in 8caa96d12fa1
	deb http://sfo1.mirrors.digitalocean.com/mariadb/repo/10.0/ubuntu trusty main
	 ---> 12231367c510
	Removing intermediate container 8caa96d12fa1
	Step 6 : RUN apt-get update
	 ---> Running in cbd89fde87d3
	Get:1 http://sfo1.mirrors.digitalocean.com trusty InRelease [2488 B]
	Ign http://archive.ubuntu.com trusty InRelease
	Ign http://archive.ubuntu.com trusty-updates InRelease
	
	......
	
	Reading package lists...
	 ---> 19bd40f631e4
	Removing intermediate container cbd89fde87d3
	Step 7 : RUN apt-get install -y -q mariadb-server mariadb-client
	 ---> Running in 9376a559e85e
	Reading package lists...
	Building dependency tree...
	Reading state information...
	The following extra packages will be installed:
	  libaio1 libdbd-mysql-perl libdbi-perl libhtml-template-perl
	  libmariadbclient18 libmysqlclient18 libreadline5 mariadb-client-10.0
	  mariadb-client-core-10.0 mariadb-common mariadb-server-10.0
	  mariadb-server-core-10.0 mysql-common psmisc
	Suggested packages:
	  libclone-perl libmldbm-perl libnet-daemon-perl libplrpc-perl
	  libsql-statement-perl libipc-sharedcache-perl libterm-readkey-perl tinyca
	  mailx mariadb-test
	The following NEW packages will be installed:
	  libaio1 libdbd-mysql-perl libdbi-perl libhtml-template-perl
	  libmariadbclient18 libmysqlclient18 libreadline5 mariadb-client
	  mariadb-client-10.0 mariadb-client-core-10.0 mariadb-common mariadb-server
	  mariadb-server-10.0 mariadb-server-core-10.0 mysql-common psmisc
	0 upgraded, 16 newly installed, 0 to remove and 11 not upgraded.
	Need to get 13.9 MB of archives.
	After this operation, 137 MB of additional disk space will be used.
	Get:1 http://sfo1.mirrors.digitalocean.com/mariadb/repo/10.0/ubuntu/ trusty/main mysql-common all 10.0.20+maria-1~trusty [8388 B]
	......
	Get:16 http://archive.ubuntu.com/ubuntu/ trusty/main libhtml-template-perl all 2.95-1 [65.5 kB]
	Preparing to unpack .../psmisc_22.20-1ubuntu2_amd64.deb ...
	Unpacking psmisc (22.20-1ubuntu2) ...
	Selecting previously unselected package mariadb-server-core-10.0.
	......
	Configuring mariadb-server-10.0
	-------------------------------
	
	While not mandatory, it is highly recommended that you set a password for the 
	MariaDB administrative "root" user.
	
	If this field is left blank, the password will not be changed.
	
	New password for the MariaDB "root" user: 
	Use of uninitialized value $_[1] in join or string at /usr/share/perl5/Debconf/DbDriver/Stack.pm line 111.
	Use of uninitialized value $val in substitution (s///) at /usr/share/perl5/Debconf/Format/822.pm line 83, <GEN6> line 1.
	Use of uninitialized value $val in concatenation (.) or string at /usr/share/perl5/Debconf/Format/822.pm line 84, <GEN6> line 1.
	Unpacking mariadb-server-10.0 (10.0.20+maria-1~trusty) ...
	Selecting previously unselected package libhtml-template-perl.
	Preparing to unpack .../libhtml-template-perl_2.95-1_all.deb ...
	......
	Configuring mariadb-server-10.0
	-------------------------------
	
	While not mandatory, it is highly recommended that you set a password for the 
	MariaDB administrative "root" user.
	
	If this field is left blank, the password will not be changed.
	
	New password for the MariaDB "root" user: 
	Use of uninitialized value $_[1] in join or string at /usr/share/perl5/Debconf/DbDriver/Stack.pm line 111.
	invoke-rc.d: policy-rc.d denied execution of stop.
	invoke-rc.d: policy-rc.d denied execution of start.
	Setting up mariadb-client (10.0.20+maria-1~trusty) ...
	Processing triggers for ureadahead (0.100.0-16) ...
	Setting up mariadb-server (10.0.20+maria-1~trusty) ...
	Processing triggers for libc-bin (2.19-0ubuntu6.6) ...
	 ---> 3367c0b221b9
	Removing intermediate container 9376a559e85e
	Step 8 : RUN mkdir -p /var/lib/mysql   && mkdir -p /var/log/mysql   && chown mysql:mysql /var/log/mysql
	 ---> Running in af3cdc6b1a8b
	 ---> 5715c0920490
	Removing intermediate container af3cdc6b1a8b
	Step 9 : EXPOSE 3306
	 ---> Running in ecacf80ce76e
	 ---> 6849bb03001e
	Removing intermediate container ecacf80ce76e
	Successfully built 6849bb03001e

## 5. 验证我们构建的镜像

列出镜像列表：

	$ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
	wenchma/ubuntu      mariadb             6849bb03001e        9 seconds ago       463.5 MB
	ubuntu              14.04               6d4946999d4f        2 weeks ago         188.3 MB
	ubuntu              latest              6d4946999d4f        2 weeks ago         188.3 MB

用我们新构建的镜像创建一个容器并测试一下MariaDB：

	$ docker run -it wenchma/ubuntu:mariadb /bin/bash
	root@d31eca25d0a9:/# ls
	bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
	root@d31eca25d0a9:/# ls /var/lib/mysql/
	aria_log.00000001  aria_log_control  debian-10.0.flag  ib_logfile0  ib_logfile1  ibdata1  mysql  performance_schema
	root@d31eca25d0a9:/# /etc/init.d/mysql status
	 * MariaDB is stopped.
	root@d31eca25d0a9:/# /etc/init.d/mysql start 
	 * Starting MariaDB database server mysqld                                                                                                                                                            [ OK ] 
	 * Checking for corrupt, not cleanly closed and upgrade needing tables.
	root@d31eca25d0a9:/# /etc/init.d/mysql status
	 * /usr/bin/mysqladmin  Ver 9.1 Distrib 10.0.20-MariaDB, for debian-linux-gnu on x86_64
	Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.
	
	Server version		10.0.20-MariaDB-1~trusty-log
	Protocol version	10
	Connection		Localhost via UNIX socket
	UNIX socket		/var/run/mysqld/mysqld.sock
	Uptime:			9 sec
	
	Threads: 1  Questions: 705  Slow queries: 0  Opens: 167  Flush tables: 1  Open tables: 31  Queries per second avg: 78.333
	root@d31eca25d0a9:/# mysql
	Welcome to the MariaDB monitor.  Commands end with ; or \g.
	Your MariaDB connection id is 45
	Server version: 10.0.20-MariaDB-1~trusty-log mariadb.org binary distribution
	
	Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	MariaDB [(none)]> show databases;
	+--------------------+
	| Database           |
	+--------------------+
	| information_schema |
	| mysql              |
	| performance_schema |
	+--------------------+
	3 rows in set (0.00 sec)

	docker run 这种方式启动的container, 如果你退出后container也就关闭了.
	docker start <container-id> # start your stopped container.
	# list 所有状态的container
	$ docker ps -a
	CONTAINER ID        IMAGE                                                              COMMAND                CREATED             STATUS                      PORTS               NAMES
	37c688837ad4        ubuntu:14.04                                                       "/bin/bash"            41 minutes ago      Exited (0) 32 seconds ago                       berserk_cori           
	f957fa8f119b        ubuntu:14.04                                                       "/usr/sbin/sshd -D"    42 minutes ago                                                      evil_thompson          
	35f84153ecbd        ubuntu:14.04                                                       "/usr/sbin/sshd-D"     42 minutes ago                                                      furious_darwin         
	0a86cb72698a        wenchma/ubuntu:mariadb                                             "/bin/bash"            58 minutes ago      Exited (0) 44 minutes ago                       condescending_leakey   
	d31eca25d0a9        wenchma/ubuntu:mariadb                                             "/bin/bash"            2 weeks ago         Exited (0) 2 weeks ago                          grave_engelbart        
	503013aeaf75        f14b6903246f10fc59f83f42154f3854991dc2dd5bbf14e367555bcf42959c4c   "/bin/sh -c 'add-apt   2 weeks ago         Exited (127) 2 weeks ago                        focused_carson
	$ docker start 37c688837ad4
	37c688837ad4
	$ docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
	37c688837ad4        ubuntu:14.04        "/bin/bash"         56 minutes ago      Up 4 seconds                            berserk_cori
	$ docker exec 37c688837ad4 ifconfig
	eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:1a  
	          inet addr:172.17.0.26  Bcast:0.0.0.0  Mask:255.255.0.0
        	  inet6 addr: fe80::42:acff:fe11:1a/64 Scope:Link
	          UP BROADCAST RUNNING  MTU:1500  Metric:1
	          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0 
	          RX bytes:648 (648.0 B)  TX bytes:738 (738.0 B)

	lo        Link encap:Local Loopback  
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          inet6 addr: ::1/128 Scope:Host
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0 
	          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

	docker run 是启动一个container然后执行命令，docker exec 是在一个running container执行命令.

自此，我们的镜像就构建成功了，当然你也可以`push` 你的镜像到[Docker Hub][Docker-Hub], 这里面有全世界开发者贡献的镜像。

[Docker-Hub]:  https://hub.docker.com/
[docker-installation-on-ubuntu]:      http://docs.docker.com/installation/ubuntulinux/
