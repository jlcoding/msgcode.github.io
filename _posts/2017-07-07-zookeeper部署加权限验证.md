---
layout: post
title: Zookeeper部署
subtitle: 简单又实用
date: 2017-07-07 13:32:20 +0300
description: 之前尝试搭建Zk+Dubbo时的记录,现在迁移过来
tags: Zookeeper
post-card-type: image
card-image: assets/images/zookeeper.jpg
---

##前言

神马下载那些东西就不说啦..自己来啦 本教程是基于ZooKeeper 3.4.10进行的, 部分内容可能会有版本差异, 敬请家长留意. 这里只讲zookeeper(下面简称为:zk) ACL的digest scheme 和 ip scheme权限验证方式. 因为比较常用.



##实际操作

第一步: 解压, 这个不详细说了.

第二步: zk有一个默认配置, 我们copy 一份出来修改自己的配置

```
cp conf/zoo_sample.cfg zoo.cfg
```



第三步: zk里面的记录实际就像一个XML, 或者说像一个文件系统, 会有一个根目录 / 然后在根目录中创建节点, 来保存不同的数据.

```
create /zhishiquan "zhishiquan" #创建节点和别名
```



第四步

