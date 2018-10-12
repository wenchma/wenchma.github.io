---
layout: post
title: "Superset 开发环境部署"
categories: tech
tags: superset
date: 2018-10-10 09:27:51
---

搭建Superset 开发环境，以进行Superset的二次开发。

## Setup Steps:

###  OS info
```
# uname -a
Linux superset 3.10.0-327.28.3.el7.x86_64 #1 SMP Thu Aug 18 19:05:49 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
```

### 1.  Install requirements

```
# yum upgrade python-setuptools

# yum install gcc gcc-c++ libffi-devel python-devel python-pip python-wheel openssl-devel libsasl2-devel openldap-devel

# yum -y install epel-release
# yum install python-pip

# pip install virtualenv

# wget https://codeload.github.com/apache/incubator-superset/tar.gz/0.27.0
# tar -zxvf 0.27.0
# cd incubator-superset-0.27.0

# 安装过程中版本冲突， pip uninstall pkg==version1, then pip install==version2
# python setup.py install

安装成功会提示：
......
Using /usr/lib64/python2.7/site-packages
Finished processing dependencies for superset==0.27.0
# which superset
/usr/bin/superset
```

### 2. Create admin user for superset


```
# fabmanager  create-admin --app superset
Username [admin]: admin
User first name [admin]: admin
User last name [user]: Ma
Email [admin@fab.org]: wench.ma@hnair.com
Password:                           # superset
Repeat for confirmation:   # superset
Recognized Database Authentications.
Admin User admin created.

```

### 3. 初始化Superset数据

初始化数据库
```
# superset db upgrade
```

*加载测试examples 数据*

```
# superset load_examples
Loading examples into <SQLA engine=u'sqlite:////root/.superset/superset.db'>
Creating default CSS templates
Loading energy related dataset
Creating table [wb_health_population] reference
2018-10-09 03:55:15,380:INFO:root:Creating database reference
2018-10-09 03:55:15,471:INFO:root:Database.get_sqla_engine(). Masked URL: sqlite:////root/.superset/superset.db
Loading [World Bank's Health Nutrition and Population Stats]
Creating table [wb_health_population] reference
2018-10-09 03:55:43,613:INFO:root:Creating database reference
......

```

*创建默认的role and permissions*

```
# superset init
2018-10-09 03:58:21,503:INFO:root:Creating database reference
2018-10-09 03:58:21,521:INFO:root:Syncing role definition
2018-10-09 03:58:21,752:INFO:root:Syncing Admin perms
2018-10-09 03:58:21,933:INFO:root:Syncing Alpha perms
2018-10-09 03:58:22,613:INFO:root:Syncing Gamma perms
2018-10-09 03:58:23,264:INFO:root:Syncing granter perms
2018-10-09 03:58:23,765:INFO:root:Syncing sql_lab perms
2018-10-09 03:58:24,491:INFO:root:Fetching a set of all perms to lookup which ones are missing
2018-10-09 03:58:24,639:INFO:root:Creating missing datasource permissions.
2018-10-09 03:58:24,650:INFO:root:Creating missing database permissions.
2018-10-09 03:58:24,743:INFO:root:Creating missing metrics permissions
2018-10-09 03:58:24,791:INFO:root:Cleaning faulty perms
```

### 4. 启动Superset

```
# superset/bin/superset runserver
2018-10-09 05:00:01,372:INFO:root:The Gunicorn 'superset runserver' command is deprecated. Please use the 'gunicorn' command instead.
Starting server with command: 
gunicorn -w 2 --timeout 60 -b  0.0.0.0:8088 --limit-request-line 0 --limit-request-field_size 0 superset:app

[2018-10-09 05:00:01 +0000] [23005] [INFO] Starting gunicorn 19.9.0
[2018-10-09 05:00:01 +0000] [23005] [INFO] Listening at: http://0.0.0.0:8088 (23005)
[2018-10-09 05:00:01 +0000] [23005] [INFO] Using worker: sync
[2018-10-09 05:00:01 +0000] [23010] [INFO] Booting worker with pid: 23010
[2018-10-09 05:00:01 +0000] [23011] [INFO] Booting worker with pid: 23011
```
也可指定`-d` 进入Debug模式。

登录后首页无法正常显示如下，
![](/img/superset-err.jpg =150x300)

需要安装nodejs 环境，重新build:

```
# yum install nodejs
# npm install -g yarn
# cd superset/assets
# yarn
# yarn run build
# cd ../../
# python setup.py install
```

然后再启动superset, 一切正常。
后台启动:
```
# superset/bin/superset runserver >/dev/null 2>&1 &
```

## Troubleshooting

### 1. Superset table view 中列名随机排序，非用户所需

修改如下：
```python
    def process_metrics(self):
        # metrics in TableViz is order sensitive, so metric_dict should be
        # OrderedDict
        self.metric_dict = OrderedDict()
        fd = self.form_data
        for mkey in METRIC_KEYS:
```

> Note. collections.OrderedDict 会记录key-value添加的顺序，但初始化时传入多个参数，顺序是随机的。

### 2. Superset metrics 显示为中文有问题

```
'ascii' codec can't decode byte 0xe6 in position 94: ordinal not in range(128)
Traceback (most recent call last):
  File "/root/incubator-superset-0.27.0/superset/viz.py", line 394, in get_df_payload
    df = self.get_df(query_obj)
  File "/root/incubator-superset-0.27.0/superset/viz.py", line 201, in get_df
    if not df.empty:
AttributeError: 'NoneType' object has no attribute 'empty'
```
手动重现, 发现不是Sqlite 的原因, python 要升级到3：
```
$ python
Python 2.7.15 (default, Jun 17 2018, 13:05:56) 
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.39.2)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> "时间".encode("utf8")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 0: ordinal not in range(128)
```

> Note. The default string encoding of python 2.7 is ascii, it conflicts with unicode, the python3
  can distinguish the unicode string and byte array, default encoding is not ascii.

### 3. 连接MySQL 失败：

要安装相关的依赖包(centos7 要用OS官方的源，不要用mysql官方源):
```
pip3 install mysqlclient
```