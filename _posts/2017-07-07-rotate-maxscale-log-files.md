---
layout: post
title: "Rotate Maxscale log files"
categories: tech
tags: logrotate
date: 2017-07-07 14:40:54
---

## Overview

MaxScale write log data into four log files with varying degrees of detail. The default behavior of MaxScale is to grow the log files indefinitely,
we must take action to prevent this.
There are two methods to rotate log files.

## Method 1

Run `maxadmin` command to rotate a single log file or all the log files.

```bash
$ maxadmin -pmariadb
    MaxScale> flush log error
    MaxScale> flush log trace
    MaxScale>
```

or 

`$ maxadmin -pmariadb flush logs`

When the logfile is rotated, the current log file is closed and a new log file, with an increased sequence number in its name, is created.

## Method 2

Follow Linux logrotate mechanism by adding a configuration file to the /etc/logrotate.d directory.

The configuration file would look like the following:

```
/var/log/maxscale/*.log {
	monthly
	rotate 5
	missingok
	nocompress
	sharedscripts
	postrotate
	# run if maxscale is running
	if test -n "`ps acx|grep maxscale`"; then
		/usr/bin/maxadmin -pmariadb flush logs
	fi
	endscript
}
```

If avoid password appear in the conf file, use the following conf:

```
/var/log/maxscale/*.log {
    weekly
    rotate 7
    missingok
    nocompress
    sharedscripts
    compress
    postrotate
        kill -USR1 `cat /var/run/maxscale/maxscale.pid` 
    endscript
}
```

For details, please refer to [maxscale-rotating](https://mariadb.com/kb/en/mariadb-enterprise/mariadb-maxscale-14/maxscale-administration-tutorial/#rotating)