---
layout: post
title: "kong and cassandra deployment"
categories: tech
tags: kong
date: 2016-08-01 11:10:31
---

## Cassandra Installation

This is to configure database for `kong`, it requires Cassandra 2.2.x as its datastore.
Cassandra is a NoSQL database based on Java, developed by Facebook, then contributed as
open source, has storing huge amounts of data, centerless design, consistent hash, BloomFilter
highlights.

Cassandra 2.2.x requires Java 7.x.x

```bash
$ bin/cassandra -f
Cassandra 2.0 and later require Java 7u25 or later.
```

If your system has existed Java env in use, simply deploy a docker Cassandra container, no install, no config.

```bash
$ docker pull cassandra:2.2
2.2: Pulling from library/cassandra
357ea8c3d80b: Pull complete 
25eb4fd61cd9: Pull complete 
a21cf6fac262: Pull complete 
e5070e58285f: Pull complete 
0b2931b3c576: Pull complete 
0af4d9286476: Pull complete 
d441751fd050: Pull complete 
3542e2b56a66: Pull complete 
a95069fffaef: Pull complete 
a35b36a6da85: Pull complete 
Digest: sha256:8b6e48f7560a156a87d2f308103bbde2176148e82e0ff826335420a27c9da67c
Status: Downloaded newer image for cassandra:2.2

$ docker run -d --name kong-database -p 9042:9042 cassandra:2.2
639b2fdee14366231333c769f9b7496b84cf46b096fde17339fa9ad5094ea053
mars@OSEE:~/software/apache-cassandra-2.2.7$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                       NAMES
639b2fdee143        cassandra:2.2       "/docker-entrypoint.s"   3 seconds ago       Up 2 seconds        7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9042->9042/tcp   kong-database
```

## Kong Installation and Configuration

```bash
$ wget https://github.com/Mashape/kong/releases/download/0.8.3/kong-0.8.3.trusty_all.deb

$ sudo dpkg -i kong-0.8.3.trusty_all.deb 
Selecting previously unselected package kong.
(Reading database ... 786606 files and directories currently installed.)
Preparing to unpack .../kong-0.8.3.trusty_all.deb ...
Unpacking kong (0.8.3) ...
Setting up kong (0.8.3) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...

$ kong status
[INFO] Using configuration: /etc/kong/kong.yml
[ERR] Kong is not running
```

**then configure /etc/kong.kong.yml**:

```yaml
######
## Specify which database to use. Only "cassandra" and "postgres" are currently available.
database: cassandra
######
## Cassandra configuration (keyspace, authentication, client-to-node encryption)
cassandra:
 ######
 ## Contact points to your Cassandra cluster.
  contact_points:
    - "127.0.0.1:9042"
  ######
  ## Name of the keyspace used by Kong. Will be created if it does not exist.
  keyspace: kong

  #####
  ## Connection and reading timeout (in ms).
  timeout: 5000

  replication_strategy: SimpleStrategy
```

> Note. Kong working dir: /usr/local/kong/
		Kong library dir: /usr/local/share/lua/5.1/kong 

more configuration, refer to [kong configuration](https://getkong.org/docs/0.8.x/configuration/)

Initial startup, it will initialize the data :

```bash
$ kong start
[INFO] kong 0.8.3
[INFO] Using configuration: /etc/kong/kong.yml
[INFO] Setting working directory to /usr/local/kong
[INFO] database...........cassandra contact_points=127.0.0.1:9042 data_centers= ssl=verify=false enabled=false port=9042 consistency=ONE replication_strategy=SimpleStrategy timeout=5000 replication_factor=1 keyspace=kong
[INFO] Migrating core (cassandra)
[INFO] core migrated up to: 2015-01-12-175310_skeleton
[INFO] core migrated up to: 2015-01-12-175310_init_schema
[INFO] core migrated up to: 2015-11-23-817313_nodes
[INFO] core migrated up to: 2016-02-25-160900_remove_null_consumer_id
[INFO] core migrated up to: 2016-02-29-121813_remove_ttls
[INFO] Migrating key-auth (cassandra)
[INFO] key-auth migrated up to: 2015-07-31-172400_init_keyauth
[INFO] Migrating galileo (cassandra)
[INFO] galileo migrated up to: 2016-04-15_galileo-import-mashape-analytics
[INFO] Migrating acl (cassandra)
[INFO] acl migrated up to: 2015-08-25-841841_init_acl
[INFO] Migrating response-ratelimiting (cassandra)
[INFO] response-ratelimiting migrated up to: 2015-08-21_init_response-rate-limiting
[INFO] Migrating ip-restriction (cassandra)
[INFO] ip-restriction migrated up to: 2016-05-24-remove-cache
[INFO] Migrating rate-limiting (cassandra)
[INFO] rate-limiting migrated up to: 2015-08-03-132400_init_ratelimiting
[INFO] Migrating oauth2 (cassandra)
[INFO] oauth2 migrated up to: 2015-08-03-132400_init_oauth2
[INFO] oauth2 migrated up to: 2015-08-24-215800_cascade_delete_index
[INFO] oauth2 migrated up to: 2016-02-29-435612_remove_ttl
[INFO] oauth2 migrated up to: 2016-04-14-283949_serialize_redirect_uri
[INFO] Migrating request-transformer (cassandra)
[INFO] request-transformer migrated up to: 2016-03-10-160000_req_trans_schema_changes
[INFO] Migrating jwt (cassandra)
[INFO] jwt migrated up to: 2015-06-09-jwt-auth
[INFO] jwt migrated up to: 2016-03-07-jwt-alg
[INFO] Migrating basic-auth (cassandra)
[INFO] basic-auth migrated up to: 2015-08-03-132400_init_basicauth
[INFO] Migrating response-transformer (cassandra)
[INFO] response-transformer migrated up to: 2016-03-10-160000_resp_trans_schema_changes
[INFO] Migrating hmac-auth (cassandra)
[INFO] hmac-auth migrated up to: 2015-09-16-132400_init_hmacauth
[INFO] dnsmasq............address=127.0.0.1:8053 dnsmasq=true port=8053
[INFO] serf ..............-profile=wan -rpc-addr=127.0.0.1:7373 -event-handler=member-join,member-leave,member-failed,member-update,member-reap,user:kong=/usr/local/kong/serf_event.sh -bind=0.0.0.0:7946 -node=OSEE_0.0.0.0:7946_38f1b96208954023b99bb5406ef5ec5b -log-level=err
[INFO] Trying to auto-join Kong nodes, please wait..
[INFO] No other Kong nodes were found in the cluster
[INFO] Auto-generating the default SSL certificate and key...
[WARN] ulimit is currently set to "1024". For better performance set it to at least "4096" using "ulimit -n"
[INFO] nginx .............admin_api_listen=0.0.0.0:8001 proxy_listen=0.0.0.0:8000 proxy_listen_ssl=0.0.0.0:8443
[OK] Started
```

### Test your installation

```bash
$ curl localhost:8001/apis

{"data":[],"total":0}
$ curl localhost:8001/plugins/enabled | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   347    0   347    0     0   269k      0 --:--:-- --:--:-- --:--:--  338k
{
  "enabled_plugins": [
    "ssl",
    "jwt",
    "acl",
    "correlation-id",
    "cors",
    "oauth2",
    "tcp-log",
    "udp-log",
    "file-log",
    "http-log",
    "key-auth",
    "hmac-auth",
    "basic-auth",
    "ip-restriction",
    "galileo",
    "request-transformer",
    "response-transformer",
    "request-size-limiting",
    "rate-limiting",
    "response-ratelimiting",
    "syslog",
    "loggly",
    "datadog",
    "runscope",
    "ldap-auth",
    "statsd"
  ]
}
```

## References

* [Kong Official Doc](https://getkong.org/docs/)
* [Cassandra Doc](http://cassandra.apache.org/doc/latest/)
* [Cassandra Docker Images](https://hub.docker.com/_/cassandra/)
