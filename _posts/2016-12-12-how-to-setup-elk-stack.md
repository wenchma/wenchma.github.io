---
layout: post
title: "How to setup ELK Stack"
categories: tech
tags: elasticsearch logstash
date: 2016-12-12 17:08:35
---

![ELK](/img/elk.png)

## Overview

The ELK stack consists of Elasticsearch, Logstash, and Kibana used to centralize the logs.
Logstash is an open source tool for collecting, parsing, and storing logs for future use.
Kibana is a web interface that can be used to search and view the logs that Logstash has indexed.
Both of these tools are based on Elasticsearch, which is used for storing logs.

The ELK Stack has four main components:

* Logstash: The server component of Logstash that processes incoming logs
* Elasticsearch: Stores all of the logs
* Kibana: Web interface for searching and visualizing logs, which will be proxied through Nginx
* Filebeat: Installed on client servers that will send their logs to Logstash,
  Filebeat serves as a log shipping agent that utilizes the lumberjack networking protocol to communicate with Logstash.

![ELK infra](/img/elk-infrastructure.png)

Pre-check the Ubuntu Server release:

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04 LTS
Release:	16.04
Codename:	xenial
```

## Install Java 8

Elasticsearch and Logstash require Java, so we will install a recent version of Oracle Java 8 because that is what Elasticsearch recommends.

```bash
$ sudo add-apt-repository -y ppa:webupd8team/java

$ sudo apt-get update

$ sudo apt-get -y install oracle-java8-installer

$ java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```

## Install Elasticsearch

This needs to add elk's package source list, then install it by pkg manager.

import the Elasticsearch public GPG key into apt:

```bash
$ wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

create source list and install:

```
$ echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list

$ sudo apt-get update

$ sudo apt-get -y install elasticsearch
```

We should restrict outside access to ELK server, so can't read our data or shutdown our Elasticsearch cluster through the HTTP API.

Modify the configuration: /etc/elasticsearch/elasticsearch.yml

```bash
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: localhost
#
# Set a custom port for HTTP:
#
# http.port: 9200
#
```