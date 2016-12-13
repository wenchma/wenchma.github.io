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

then run the ELK server:

```bash
$ sudo service elasticsearch restart

$ sudo update-rc.d elasticsearch defaults 95 10

$ curl localhost:9200
{
  "name" : "The Blank",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "4akK1qP2S1OntBkv5OCpHQ",
  "version" : {
    "number" : "2.4.2",
    "build_hash" : "161c65a337d4b422ac0c805f284565cf2014bb84",
    "build_timestamp" : "2016-11-17T11:51:03Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.2"
  },
  "tagline" : "You Know, for Search"
}

```

## Install logstash

The public GPG key is same as ELK, so just create source list and install.

```bash
$ echo 'deb http://packages.elastic.co/logstash/2.4/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash-2.4.x.list
deb http://packages.elastic.co/logstash/2.4/debian stable main

$ sudo apt-get update

$ sudo apt-get install logstash
```

### Configure Logstash

Here we use Filebeat to ship logs from Client to ELK Server, so create an SSL certificate and key pair to verify the identity of ELK Server.

```bash
$ sudo mkdir -p /var/lib/logstash/private

$ sudo chown logstash:logstash /var/lib/logstash/private

$ sudo chmod go-rwx /var/lib/logstash/private



```

If you don't setup DNS, change `/CN=192.168.0.11` to your real server's private IP address. To avoid `TLS handshake error`, 
add IP to the subjectAltName (SAN) field of the SSL certificate to generate the cert.

```bash
$ sudo vi /etc/ssl/openssl.cnf


[ v3_ca ]


# Extensions for a typical CA

subjectAltName=IP:192.168.0.11

$ sudo openssl req -config /etc/ssl/openssl.cnf -x509  -batch -nodes -newkey rsa:2048 -keyout /var/lib/logstash/private/logstash-forwarder.key -out /var/lib/logstash/private/logstash-forwarder.crt -subj /CN=192.168.0.11
```

remember that copy the cert to all of servers whose logs will be sent to logstash by filebeat.

Logstash configuration consists of three sections: inputs, filters, and outputs.

#### 1. Create a conf file called 02-beats-input.conf and set up the "filebeat" input: 

```bash
sudo vi /etc/logstash/conf.d/02-beats-input.conf

input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/var/lib/logstash/private/logstash-forwarder.crt"
    ssl_key => "/var/lib/logstash/private/logstash-forwarder.key"
    }
}
```

The `filebeat` input is listening on tcp port 5044.

#### 2. Create "filebeat" filter named 10-syslog-filter.conf to add a filter for syslog.

```bash
$ sudo vi /etc/logstash/conf.d/10-syslog-filter.conf 

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}
```

This filter looks for logs that are labeled as "syslog" type (by Filebeat), and it will try to use `grok` to parse incoming syslog logs to make it structured
and query-able.

#### 3. Create "filebeat" output named 30-elasticsearch-output.conf

```bash
$ sudo vi /etc/logstash/conf.d/30-elasticsearch-output.conf 

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    sniffing => true
    manage_template => false
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
    document_type => "%{[@metadata][type]}"
  }
}

```

This output basically configures Logstash to store the beats data in Elasticsearch which is running at localhost:9200,
in an index named after the beat used (filebeat, in our case).

Test the Logstash configuration:

```bash
$ sudo service logstash configtest
log4j:WARN No appenders could be found for logger (io.netty.util.internal.logging.InternalLoggerFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Configuration OK
```

Just ignore the log4j WARN and the configuration is OK.

Then run the followng cmd:

```bash
# cd /opt/logstash/bin && ./logstash -f /etc/logstash/conf.d/02-beats-input.conf
```

You will find that the logstash has started a pipeline and processing the syslogs. Once you are sure that logstash is processing the syslogs,
combine `02-beats-input.conf`, `10-syslog-filter.conf` and `30-elasticsearch-output`.conf as `logstash.conf` in the directory /etc/logstash/

Restart logstash to reload new configuration.

```bash
$ sudo service logstash restart

$ sudo update-rc.d logstash defaults 96 9
```
未完待续......