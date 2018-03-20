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

#####第一步: 

​	解压, 这个不详细说了.

#####第二步: 

​	zk有一个默认配置, 我们copy 一份出来修改自己的配置

```shell
cp conf/zoo_sample.cfg zoo.cfg 
```



#####第三步:

 	zk里面的记录实际就像一个XML, 或者说像一个文件系统, 会有一个根目录 / 然后在根目录中创建节点, 来保存不同的数据.

```shell
create /zhishiquan "zhishiquan" #创建节点和别名
```



#####第四步: 

​	ip scheme(通过限制ip 127.0.0.1 或限制ip断 127.0.0.0/16) 来达到权限控制效果

 1. 新建节点.(不再阐述)

 2. ```shell
    setAcl /zhishiquan ip:127.0.0.1:crwda #限制只有ip127.0.0.2才可以访问 (crwda 代表权限下面解释)
    ls /zhishiquan #本机是可以正常访问的,可以尝试用另一台机器进行访问来判断权限是否加成功
    ```



​	digest scheme(实际就是添加一个用户, 这种方式定制化比较强点)

1. 新建节点(不再阐述)
2. 生成密文  (username:BASE64(SHA1(password)))

```shell
java -cp ./zookeeper-3.4.10.jar:./lib/log4j-1.2.16.jar:./lib/slf4j-log4j12-1.6.1.jar:./lib/slf4j-api-1.6.1.jar org.apache.zookeeper.server.auth.DigestAuthenticationProvider test:123456  #通过这个类计算出密文
```

3.设置用户名密码

```shell
setAcl /zhishiquan digest:test:生成出来的密文:crwda #test是用户名 生成出来的密文=刚刚生成出来的密文
```

4.测试下

```shell
ls /zhishiquan #现在是正常的,重新登录再试试
```

5.登录

```shell
addauth digest test:123456 #test是用户名 123456是密码(不是密文哦)
ls /zhishiquan
```



## 总结

这里介绍了两种zk的权限管理方式, 通常应该第二种比较常用~希望可以帮到大家. 大神不要喷我哦!