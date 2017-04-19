---
layout: post
title: "Keepalived check and notify scripts"
categories: tech
tags: keepalived
date: 2017-04-19 11:02:14
---

## Keepalived

Keepalived is a Linux implementation of the VRRP (Virtual Router Redundancy Protocol) protocol to make IPs highly available.
Keepalived check and notify scripts can be used to check anything you want to ensure the Master is on the right node
and take action if a state change.

## Check scripts

Check script has two reutrn value: 0 for everything is fine, 1 or other than 0 means something went wrong.

For example:

```bash
vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  fall 2
  rise 2
  weight 20
}
```

This script defines haproxy check to check whether haproxy daemon is running.
The check interval is 2 seconds, check fail and succeed twice for KO and OK.


The check script is used in a `vrrp_instance`:

```
vrrp_instance api_vip {
  state MASTER
  interface eth0
  virtual_router_id 5
  priority 100
  advert_int 1
  virtual_ipaddress {
    192.168.1.100/32 dev eth0
  }
  track_script {
    chk_haproxy
  }
}
```

The `track_script` returns other code than 0 two times, the VRRP instance will change the state to `FAULT`,
or the instance will change the state to `running` if return code 0 two times.

## Notify scripts

Keepalived tasks some action depending on the VRRP state.


```
vrrp_instance api_vip {
  [...]
  notify /usr/local/bin/notify-haproxy.sh
}
```

The script is called after any state change with the following parameters:

* $1 = “GROUP” or “INSTANCE”
* $2 = name of group or instance
* $3 = target state of transition (“MASTER”, “BACKUP”, “FAULT”)

```bash
#!/bin/bash

TYPE=$1
NAME=$2
STATE=$3

case $STATE in
        "MASTER") /usr/sbin/service haproxy start
                  exit 0
                  ;;
        "BACKUP") /usr/sbin/service haproxy stop
                  exit 0
                  ;;
        "FAULT")  /usr/sbin/service haproxy stop
                  exit 0
                  ;;
        *)        echo "unknown state"
                  exit 1
                  ;;
esac
```

We can implement haproxy daemon is running on only master node, stop on slave nodes.
