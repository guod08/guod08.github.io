---
layout: post
title: "Druid cluster setup"
date: 2015-03-02 14:53:25 +0800
comments: true
categories: engineering
---
本文介绍如何搭建Druid cluster，Druid的介绍与应用见[另一篇文章](http://dongguo.me/blog/2014/05/05/druid-introduction-and-practise/)

Druid的官网也有详细的[文档](http://druid.io/docs/latest/)，建议浏览一遍。本文对关键部分做一些梳理，总结一些比较坑的点。

##机器准备
Druid包含若干个services和nodes，我的配置如下（如果没有多个机器，当然可以将所有模块都起在一台机器上）

+ services/nodes on machine1: Mysql server, Zookepper server, coordinator node, overlord node (indexing service)  
+ services/nodes on machine2: Historical node, Realtime node
+ services/nodes on machine3: Broker node

3台机器都安装配置好java ([how](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get))



##安装配置依赖
####mysql配置
按照Druid的文档安装mysql并创建一个新的用户druid/diurd。理论上Druid在后续步骤会在database druid中创建3张表druid_config, druid_rules和druid_segments。如果最终你发现没有这3张表，可以手动创建。

####安装Zookeeper
安装启动，无坑

####Deep storage
如果是local模式（全部都在一台机器上），使用本地磁盘作为deep storage是最简单的，对于cluster，较简单的方式是大家（indexing services, historical node, realtime node）挂载一块公共的磁盘(比如nfs方式)，这样historical node就可以同步deep storage上的segments，realtime node也可以将segments同步到deep storage上来。

在实际应用中数据量通常比较大，常常会使用hdfs作为deep storage，为了能够将segments写入到hdfs中，


###配置启动Druid各个nodes
对于如下每个node/service，Druid都有一个配置文件runtime.properties（较新的版本将一些公共的配置提取了出来），每个node/service都配置下druid.zk.service.host为zookeeper的地址。

+ coordinator node: 无坑，在machine1上启动
+ historical node: Druid默认Deep storage数据路径为/tmp/druid/localStorage, 可通过配置druid.storage.storageDirectory=XXX来覆盖。

	```
druid.storage.type=local
druid.storage.storageDirectory=/mnt/data/druid/localStorage
druid.segmentCache.locations=[{"path": "/mnt/data/druid/indexCache", "maxSize"\: 10000000000}]
	```
	如果deep storage是hdfs，则修改druid.storage.type=hdfs，druid.storage.storageDirectory为hdfs上的路径

+ broker node: 无坑，在machine3上启动；
+ indexing service: 
	
	```
druid.indexer.task.hadoopWorkingPath=hdfs://elsaudnn001.prod.hulu.com/user/guodong/druid
druid.storage.type=hdfs
druid.storage.storageDirectory=hdfs://elsaudnn001.prod.hulu.com/user/guodong/druid
	```
	对应deep storage是hdfs
+ realtime node: 需要使用kafka，参考官网文档即可；


###数据导入
使用indexing services是Druid推荐的数据导入方式，数据的input和output都可以是本地/挂载磁盘或者hdfs。
如果要读写hdfs，需要保证druid引用的hadoop版本和你使用的版本一致。

另外Druid引用了2.5.0版本的protobuf，而2.1.0之前版本的hadoop使用的是更老的protobuf版本（如2.4.0a），如果你遇到protobuf版本冲突的问题，需要修改druid的pom.xml[重新打包](http://druid.io/docs/latest/Build-from-source.html)


##参考
1. [官方文档](http://druid.io/docs/latest/)
2. [google groups讨论区](https://groups.google.com/forum/#!forum/druid-development)