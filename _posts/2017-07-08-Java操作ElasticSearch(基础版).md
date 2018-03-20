---
layout: post
title: Java操作ElasticSearch(基础版)
subtitle: 简单实现搜索
date: 2017-07-06 13:32:20 +0300
description: 产品经理叫你做个搜索怎么办?
tags: Elastic
post-card-type: image
card-image: assets/images/elastic.jpg
---

##前言

看本文前需要先对ElasticSearch有一定了解(比如什么是index 什么是type)

​     本人推荐直接使用ElasticSearch Java API来操作..本人最开始是用springboot spring-data-elasticsearch-starter 来作为orm的..操作是方便了很多.支持类spring-data-jpa的接口化查询..但是弊端太多.

​     1.更新非常慢.

​     2.文档非常少.

​     3.封装太多,不利于学习原生ElasticSearch

​     4.对于Springboot maven modules 项目来说..当更新ElasticSearch要更换parent的springboot 版本..

​     用spring-data-elasticsearch的话少了第四点.



## 开始搞事

1.加入依赖..(本例是基于Springboot + Maven..)

​      

```xml
  <dependency>
  	<groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>5.5.0</version>
 </dependency>
```

```xml
 <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>transport</artifactId>
    <version>5.5.0</version>
 </dependency>

   <!-- 没有使用X-Pack可以忽略下面依赖 -->
 <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>x-pack-transport</artifactId>
    <version>5.5.0</version>
 </dependency>
```

**版本需要与你的ElasticSearch版本相同**

ElasticsearchConfiguration.java

```java
package cn.youyinian.server;

import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.InetSocketTransportAddress;
import org.elasticsearch.xpack.client.PreBuiltXPackTransportClient;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

import java.net.InetAddress;
import java.net.UnknownHostException;

@Configuration
public class ElasticsearchConfiguration implements FactoryBean<TransportClient>, InitializingBean, DisposableBean {
    private static final Logger logger = LoggerFactory.getLogger(ElasticsearchConfiguration.class);
    //由于项目从2.2.4配置的升级到 5.5.0版本 原配置文件不想动还是指定原来配置参数
 @Value("${spring.data.elasticsearch.cluster-nodes}")
    private String clusterNodes ;

    @Value("${spring.data.elasticsearch.cluster-name}")
    private String clusterName;

    private TransportClient client;

    @Override
 public void destroy() throws Exception {
        try {
            logger.info("Closing elasticSearch client");
            if (client != null) {
                client.close();
            }
        } catch (final Exception e) {
            logger.error("Error closing ElasticSearch client: ", e);
        }
    }

    @Override
 public TransportClient getObject() throws Exception {
        return client;
    }

    @Override
 public Class<TransportClient> getObjectType() {
        return TransportClient.class;
    }

    @Override
 public boolean isSingleton() {
        return false;
    }

    @Override
 public void afterPropertiesSet() throws Exception {
        buildClient();
    }

    protected void buildClient()  {
        try {
            PreBuiltXPackTransportClient preBuiltXPackTransportClient = new PreBuiltXPackTransportClient(settings());
            if (!"".equals(clusterNodes)){
                for (String nodes:clusterNodes.split(",")) {
                    String InetSocket [] = nodes.split(":");
                    String  Address = InetSocket[0];
                    Integer  port = Integer.valueOf(InetSocket[1]);
                    preBuiltXPackTransportClient.addTransportAddress(new
 InetSocketTransportAddress(InetAddress.getByName(Address),port ));
                }
                client = preBuiltXPackTransportClient;
            }
        } catch (UnknownHostException e) {
            logger.error(e.getMessage());
        }
    }
    /**
 * 初始化默认的client
 */
 private Settings settings(){
        Settings settings = Settings.builder()
                .put("cluster.name",clusterName)
                .put("xpack.security.transport.ssl.enabled", false)
                .put("xpack.security.user", "elastic:changeme") //xpack user配置
                .put("client.transport.sniff", true).build();
        return settings;
    }
}

```

操作ElasticSearch

### [官方Java API 文档](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-docs-index.html)

```java
本例是使用json字符串来插入

先注入client对象

public final TransportClient client;

public TopicAuthSearchRemoteController(TransportClient client) {
    this.client = client;
}


//加入索引

IndexResponse response = client.prepareIndex("index_topic", "topic_auth")
        .setSource(topicAuthJson, XContentType.JSON)
        .get();


//搜索 - 根据实体搜索

SearchResponse response = client.prepareSearch("index_topic")
        .setTypes("topic_auth")
        .setQuery(QueryBuilders.termQuery("id", topicAuthId))
        .get();


//删除索引

DeleteResponse deleteResponse = client.prepareDelete("index_topic", "topic_auth", id).get(); //重点:这里的id并不是实体的id..而是ElasticSearch每条记录都有的 _id 可以通过String id = response.getHits().getAt(0).getId(); 或者用API中得Delete by Queries

```



## 注意事项

你可以在过程中发现没有保存但是操作也没有成功. 因为ElasticSearch Java API TransportClient是异步的. 每一个请求都是一条线程. 并没有在主线程捕捉到Error. 所以应该在方法外面加个Try Catch... 如:

```java
try {
    if(StringUtils.isEmpty(topicAuthJson)) {
        logger.info(topicAuthJson);
        IndexResponse response = client.prepareIndex("index_topic", "topic_auth")
                .setSource(topicAuthJson, XContentType.JSON)
                .get();
    }else {
        logger.error("object add to index cannot be null");
    }
}catch (Exception e) {
    logger.error("error", e);
}
```

