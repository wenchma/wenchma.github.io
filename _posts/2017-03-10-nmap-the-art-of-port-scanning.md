---
layout: post
title: "Nmap: The Art of Port Scanning"
categories: tech
tags: network security
date: 2017-03-10 14:44:31
---

Nmap ("Network Mapper") is an open source tool for network exploration and security auditing.

Mainly have the following functins:

* Determine what hosts are available on the network
* Determine what services those hosts are offering
* Determine what operating systems they are running
* Determine what type of packet filters/firewalls are in use
* Others, version detection

It was originally written by [Gordon Lyon](https://en.wikipedia.org/wiki/Gordon_Lyon) and it can answer the following questions easily:

1. What computers did you find running on the local network?

2. What IP addresses did you find running on the local network?

3. What is the operating system of your target machine?

4. Find out what ports are open on the machine that you just scanned?

5.Find out if the system is infected with malware or virus.

6. Search for unauthorized servers or network service on your network.

7.Find and remove computers which don’t meet the organization’s minimum level of security.


## Installation

```
$ sudo apt-get install nmap
```

other installation, refer to [Install Guide](https://nmap.org/book/install.html)

## Nmap Usage

1. Scan a host or an ip address

   ```
   $ nmap 127.0.0.1

   Starting Nmap 6.40 ( http://nmap.org ) at 2017-03-10 15:07 CST
   Nmap scan report for localhost (127.0.0.1)
   Host is up (0.00024s latency).
   Not shown: 987 closed ports
   PORT     STATE SERVICE
   22/tcp   open  ssh

   ## Fast mode TCP scan with more info ##
   $ nmap -F -sT -v nmap.org
   ```

2. Scan multiple ip addresses or network

   ```
   $ nmap 192.168.0.1 192.168.0.2 192.168.0.3

   $ nmap 192.168.0.1,2,3

   ## Scan an IP range
   $ nmap 192.168.0.1-100

   $nmap 192.168.0.*

   ## Scan with entire network
   $ nmap 192.168.0.0/24

   ```

3. Excluding hosts/networks

   ```
   $ nmap 192.168.0.0/24 --exclude 192.168.0.9

   $ nmap 192.168.1.0/24 --exclude 192.168.1.9,192.168.1.10
   ```

4. Enable OS detection, version detection, script scanning, and traceroute

   ```
   $ nmap -A 127.0.0.1
   ```

5. Scan specific ports

   ```
   nmap -p [port] hostName
   ## Scan port 80
   nmap -p 80 192.168.0.1
	 
   ## Scan TCP port 80
   nmap -p T:80 192.168.0.1
	 
   ## Scan UDP port 53
   nmap -p U:53 192.168.0.1
 
   ## Scan two ports ##
   nmap -p 80,443 192.168.0.1
 
   ## Scan port ranges ##
   nmap -p 80-200 192.168.0.1
	 
   ## Combine all options ##
   nmap -p U:53,111,137,T:21-25,80,139,8080 192.168.0.1
   nmap -p U:53,111,137,T:21-25,80,139,8080 server1.cyberciti.biz
   nmap -v -sU -sT -p U:53,111,137,T:21-25,80,139,8080 192.168.0.254
	 
   ## Scan all ports with * wildcard ##
   nmap -p "*" 192.168.0.1
	 
   ## Scan top ports i.e. scan $number most common ports ##
   nmap --top-ports 5 192.168.0.1
   nmap --top-ports 10 192.168.0.1
   ```

## Scan results

```
Starting Nmap 6.40 ( http://nmap.org ) at 2017-03-10 15:07 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00024s latency).
Not shown: 987 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
631/tcp  open  ipp
1023/tcp open  netvenuechat
2049/tcp open  nfs
3260/tcp open  iscsi
3306/tcp open  mysql
5900/tcp open  vnc
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```

| State           | Description                                                                    |
|-----------------|--------------------------------------------------------------------------------|
| open            | port is open, packets arrived destination, application is listened on the port |
| closed          | port is closed, packets arrived destination, no app is listened                |
| filtered        | packets not arrived destination, filtered by firewall or IDS                   |
| unfilitered     | packets arrived destination, but cannot determine port state                   |
| open/filtered   | no return from port, occur in UDP, IP, FIN, NULL and Xmas scanning             |
| closed/filtered | only occur in IP ID idle scanning                                              |
