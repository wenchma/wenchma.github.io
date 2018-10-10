---
layout: post
title: "Ceph Deployment and Automization"
categories: tech
tags: ceph
date: 2016-10-17 15:45:50
---


## Ceph Overview

Ceph 是一个PB级的统一存储集群系统, 支持的存储包括: 

* Object Storage: 有原生的API，而且也兼容Swift和S3的API
* Block Storage: 支持可配置、snapshot, clone
* File System: Posix接口，支持snaphot

Ceph 的底层核心是RADOS(Reliable, autonomous, distributed object storage),
有两个守护进程: Monitor (维护整个Ceph集群的全局状态); OSD (bject Storage Device，提供存储资源).

Ceph 还有着高性能, 高可靠性, 高可扩展性的特点，被许多厂商青睐, 云计算领域被广泛采用.

## Ceph pg 常见状态

pg ( placement group ) 是数据存储的重要单位，在使用 ceph 的时候, pg 会经常发生状态的变化,  
* 当创建pool的时候, 将会创建相应的 pg, 那么可以看到 pg creating 状态
* 当部分 pg 创建成功后, 将会发现 pg 会进入 peering 状态
* 当所有 pg peering 完成后,  将可见到状态变成 active+clean

### creating

PG 正在被创建, 通常当存储池正在卑创建或增加一个存储池的 PG 数量时, PG 会呈现这个状态

### Down

PG 处于失效状态, PG 应该处于离线状态

### repair

PG 正在被检查, 被发现的任何不一致都将尽可能被修复.

### peering

* 当 ceph peering pg, ceph 将会把 pg 副本协定导入 osd, 当 ceph 完成 peering, 意味着 osd 同意当前 PG 状态, 并允许写入
* PG 处于 peering 过程中, peering 由主 osd 发起的使存放 PG 副本的所有 OSD 就 PG 的所有对象和元素数据的状态达成一致的过程,  peering 过程
  完成后, 主 OSD 就可以接受客户端写请求.

### Active

* 当 ceph 完成 peering 过程, pg 将会变成 active, active 状态意味着 pg 中的数据变得可用, 主 pg 将可执行读写操作
* PG 是活动的, 意味着 PG 中的数据可以被读写, 对该 PG 的操作请求都讲会被处理. 

### Clean

* 当 pg 显示 clean 状态, 主 osd 与副本 osd 成功同步并且没有异步复制, ceph 在 pg 中所有对象具有正确的副本数量
* PG 中的所有对象都已经卑复制了规定的副本数量.

### Replay

某 OSD 崩溃后, PG 正在等待客户端重新发起操作

### Degraded

* 当客户端写对象到主 osd, 主 OSD 会把数据写复制到对应复制 OSD, 在主 OSD 把对象写入存储后, PG 会显示为 degraded 状态, 直到主 osd 从复制 
  OSD 中接收到创建副本对象完成信息

* PG 处于 active+degraded 原因是因为 OSD 是处于活跃, 但并没有完成所有的对象副本写入, 假如 OSD DOWN, CEPH 标记每个 PG 分配到这个相关 OSD
  的状态为 degraded, 当 OSD 重新上线, OSD 将会重新恢复, 

* 假如 OSD DOWN 并且 degraded 状态持续, CEPH 会标记 DOWN OSD, 并会对集群迁移相关 OSD 的数据, 对应时间由 mon osd down out interval 参
  数决定

* PG 可以被北极为 degraded, 因为 ceph 在对应 PG 中无法找到一个或者多个相关的对象, 你不可以读写 unfound 对象, 你仍然可以访问标记为 
  degraded PG 的其他数据

* PG 中部分对象的副本数量未达到规定的数量

### Inconsistent

PG副本出现不一致, 对象大小不正确或者恢复借宿后某个副本出现对象丢失现象

### recoverying

* ceph 设备故障容忍在一定范围的软件与硬件问题, 当 OSD 变 DOWN, 那么包含该 OSD 的 PG 副本都会有问题, 当 OSD 恢复, OSD 对应的 PG 将会更新
  并反映出当前状态, 在一段时间周期后, OSD 将会恢复 recoverying 状态

* recovery 并非永远都有效, 因为硬件故障可能会导致多个 OSD 故障, 例如, 网络交换机故障, 可以导致集群中的多个主机及主机包含的 OSD 故障
  当网络恢复之后, 每个 OSD 都必须执行恢复

* CEPH 提供一定数量的设定在新服务请求与恢复 PG 中数据对象时的资源平衡,  
* osd recovery delay start 设定允许 osd 重启, re-peer 并在启动 恢复之前处理一些回应请求,  
* osd recovery threads 设定了恢复过程中线程限制 (默认 1 ) 
* osd recovery thread timeout 设定线程超时, 因为可能出现多个 osd 故障, 重启后在 re-peer 过程中可能出现污染
* osd recovery max active 设定限制对一个 osd 从故障后, 恢复请求并发数量
* osd recovery max chunk 限制恢复时的数据 chunk 大小, 预防网络堵塞

* PG 正在迁移或者同步对象及其副本, 一个 OSD 停止服务(DOWN), 其内容将会落后与 PG 内的其他副本, 这时 PG 将会进入 recoverying 状态, 该 OSD 
  上的对象将从其他副本同步过来

### BACK FILLING

* 当新 OSD 加入集群, CRUSH 将会为集群新添加的 OSD 重新分配 PG, 强制新的 OSD 接受重新分配的 PG 并把一定数量的负载转移到新 OSD 中
  back filling OSD 会在后台处理, 当 backfilling 完成, 新的 OSD 完成后, 将开始对请求进行服务

* 在 backfill 操作期间, 你可以看到多种状态, 
  backfill_wait 表示 backfill 操作挂起, 但 backfill 操作还没有开始 ( PG 正在等待开始回填操作 )
  backfill 表示 backfill 操作正在执行
  backfill_too_full 表示在请求 backfill 操作, 由于存储能力问题, 但不可以完成, 

* ceph 提供设定管理装载重新分配 PG 关联到新的 OSD
* osd_max_backfills 设定最大数量并发 backfills 到一个 OSD, 默认 10
* osd backfill full ratio  当 osd 达到负载, 允许 OSD 拒绝 backfill 请求, 默认 85%, 
* 假如 OSD 拒绝 backfill 请求,  osd backfill retry interval 将会生效, 默认 10 秒后重试
* osd backfill scan min ,  osd backfill scan max 管理检测时间间隔

* 一个新 OSD 加入集群后, CRUSH 会把集群先有的一部分 PG 分配给他, 该过程称为回填, 回填进程完成后, 新 OSD 准备好了就可以对外提供服务

### REMAPPED

* 当 pg 改变, 数据从旧的 osd 迁移到新的 osd, 新的主 osd 应该请求将会花费一段时间, 在这段时间内, 将会继续向旧主 osd 请求服务, 直到
  PG 迁移完成, 当数据迁移完成,  mapping 将会使用新的 OSD 响应主 OSD 服务

* 当 PG 的 action set 变化后, 数据将会从旧 acting set 迁移到新 action set, 新主 OSD 需要过一段时间后才能提供服务, 因此它会让老的主 OSD    继续提供服务, 知道 PG 迁移完成, 数据迁移完成后, PG map 将会使用新 acting set 中的主 OSD

### Stale

* 当 ceph 使用 heartbeat 确认主机与进程是否运行,  ceph osd daemon 可能由于网络临时故障, 获得一个卡住状态 (stuck state) 没有得到心跳回应;
* 默认, osd daemon 会每 0.5 秒报告 PG, up 状态, 启动与故障分析, 
* 假如 PG 中主 OSD 因为故障没有回应 monitor 或者其他 OSD 报告 主 osd down, 那么 monitor 将会标记 PG stale, 
* 当你重启集群, 通常会看到 stale 状态, 直到 peering 处理完成, 
* 在集群运行一段时候, 看到 stale 状态, 表示主 osd PG DOWN 或者没有主 osd 没有报告 PG 信息到 monitor 中

* PG 状态很长时间没有被 ceph-osd 更新过, 标识存储在该 GP 中的节点显示为 DOWN,  PG 处于 unknown 状态, 因为 OSD 没有报告 monitor 由 mon 
  osd report timeout 定义超时时间

* PG 处于未知状态, monitors 在 PG map 改变后还没有收到过 PG 的更新, 启用一个集群后, 常常会看到主 peering 过程结束前 PG 处于该状态

### Scrubbing

PG 在做不一至性校验

### inactive

PG 很长时间没有显示为 acitve 状态,  (不可执行读写请求), PG 不可以执行读写, 因为等待 OSD 更新数据到最新的备份状态

### unclean

PG 很长时间都不是 clean 状态 (不可以完成之前恢复的操作),  PG 包含对象没有完成相应的复制副本数量, 通常都要执行恢复操作

## Ceph Troubleshooting

### Ceph存储空间查看
```
# ceph df
GLOBAL:
    SIZE     AVAIL      RAW USED     %RAW USED 
    100T     73363G       29059G         28.37 
POOLS:
    NAME                               ID     USED       %USED     MAX AVAIL     OBJECTS 
    rbd                                0           8         0        14736G           1 
    volumes                            13      7339G     21.50        14736G     1893336 
    vms                                16      1600G      4.69        20211G      413169 
```





