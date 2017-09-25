---
layout: post
title: "Best Practice in Vyatta"
categories: tech
tags: sdn
date: 2017-09-25 18:06:35
---

```
__     __           _   _        
\ \   / /   _  __ _| |_| |_ __ _ 
 \ \ / / | | |/ _` | __| __/ _` |
  \ V /| |_| | (_| | |_| || (_| |
   \_/  \__, |\__,_|\__|\__\__,_|
        |___/                    
```


## Firewall Configuration

Vyatta firewall functionality provides the following features:  

* Packet filtering for traffic that traverses the router by using the in and out keywords on an interface.
* Definable criteria for packet-matching rules, including source IP address, destination IP address, source port, destination port, IP protocol,  
  and Internet Control Message Protocol (ICMP) type.
* General detection on IP options, such as source routing and broadcast packets.
* Ability to set the firewall globally for stateful or stateless operation.
* Advanced firewall capabilities include stateful failover, zone-based firewalls, and time-based firewalls.

Firewall rules perform the following actions:

* Accept, which means that traffic is allowed and forwarded
* Drop, which means that traffic is silently discarded
* Reject, which means that traffic is discarded with an ICMP Port Unreachable message

Applying firewall rules to interfaces:

* In. If you apply the instance as in, the firewall filters packets that enter the interface and traverse the Vyatta system. You can apply one in packet filter.

* Out. If you apply the instance as out, the firewall filters packets that leave the interface. These can be packets that traverse the Vyatta system or that   
  originated on the system. You can apply one out packet filter.

* Local. If you apply the instance as local, the firewall filters packets that are destined for the Vyatta system. One firewall instance can be applied as a local  
  packet filter.
