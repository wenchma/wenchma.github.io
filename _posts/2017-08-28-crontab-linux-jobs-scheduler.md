---
layout: post
title: "Crontab - Linux jobs scheduler"
categories: tech
tags: crontab linux
date: 2017-08-28 17:48:39
---


## Overview

In unix and unix-like operating systems, we can use `crontab` to schedule commands to be exectued periodically.

To see what crontabs are running:

```
$ crontab -l
* 11 * * 5 /bin/date >> /tmp/date_cron.log
```

To edit the list of cronjobs you can run:

```
$ crontab -e
```

## 1. Crontab Format

Cronjobs are written in the following format:

```
* * * * * /bin/execute/this/script.sh
```

The five stars represents different date parts in the following order:

* minute (from 0 to 59)
* hour (from 0 to 23)
* day of month (from 1 to 31)
* month (from 1 to 12)
* day of week (from 0 to 6) (0=Sunday)

The above cronjob means the script is being executed every minute of every hour, every day of month, every month, every day of week.

## 2. Crontab Examples

1. Execute every Friday 1AM
```
0 1 * * 5 /bin/execute/this/script.sh
```

2. Execute on workdays 1AM
```
0 1 * * 1-5 /bin/execute/this/script.sh
```

3. Execute 10 min past after every hour on the 1st of every month
```
10 * 1 * * /bin/execute/this/script.sh
```

4. Execute every 10 min after every hour on the 1st of every month
```
0,10,20,30,40,50 * 1 * * /bin/execute/this/script.sh
or
*/10 * 1 * * /bin/execute/this/script.sh
```

## 3. Special words

For the first (minute) field, you can also put in a keyword instead of a number:

```
@reboot     Run once, at startup
@yearly     Run once  a year     "0 0 1 1 *"
@annually   (same as  @yearly)
@monthly    Run once  a month    "0 0 1 * *"
@weekly     Run once  a week     "0 0 * * 0"
@daily      Run once  a day      "0 0 * * *"
@midnight   (same as  @daily)
@hourly     Run once  an hour    "0 * * * *"
```

```
@daily /bin/execute/this/script.sh
```

## 4. Storing the crontab output

```
*/10 * * * * /bin/execute/this/script.sh >> /var/log/script_output.log 2>&1
```

> Note. 2>&1 tells linux to store stderr in stdout.

## 5. Mailing the crontab output

```
MAILTO="yourname@yourdomain.com"
```

Need to install mailx first:

```
$ sudo apt-get install mailx
```

Just mail one cronjob output:

```
*/10 * * * * /bin/execute/this/script.sh 2>&1 | mail -s "Cronjob ouput" yourname@yourdomain.com
```

## 6. Special Character 

* % - `%` is a special character to the crontab, which gets translated to a newline

```
25 9 * * * mysqldump  > `date '+\%Y\%m\%d\%H\%M\%S'`.sql
```