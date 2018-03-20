---
layout: post
title: Redis Value对象缓存用Hash方穿透(代码片码)
subtitle: 简单实用减轻数据库压力
date: 2017-07-06 13:32:20 +0300
description: 简单实用减轻数据库压力
tags: MySQL
post-card-type: image
card-image: assets/images/mysql.jpeg
---

##前言

很多缓存是由数据库中刷出来的, 查到缓存没有就去查数据库然后存到缓存里面, 但是对于一些经常为空的对象应该采取防穿透处理, 避免大量查询数据库增大io压力.



## 代码片段

ps:请不要太在意缩进问题, 这个是编辑器的问题

```java
/**
 * 查询最后一个练习id
 * @param lessonId 课时id
 * @param userId 用户id
 */
@Override
public Long lastMomentId(Long lessonId, Long userId) {
Long challengeMomentId = 0L;
String key = RedisKeyUtil.build(RedisConstants.ChallengeMoment.LAST_MOMENT_ID, lessonId, userId);

if(hashRedisTemplate.exists(key)) { //判断key是否存在, 存在取出, 不存在就从数据库reload
String json = hashRedisTemplate.hget(key, RedisConstants.HashEntity.VALUE);
if(StringUtils.isNotBlank(json)) {
	challengeMomentId = Long.parseLong(json);
}
}else {
ChallengeMoment challengeMoment = challengeMomentRepository.findFirstByMap(Mapper.sql("lesson.last.moment"), new MapBuilder()
	.append("createBy", userId)
	.append("lessonId", lessonId)
	.build()); //数据库查询操作
if(null != challengeMoment) {
	challengeMomentId = challengeMoment.getId();
	hashRedisTemplate.hset(key, RedisConstants.HashEntity.VALUE, 			  String.valueOf(challengeMomentId));
	hashRedisTemplate.hset(key, RedisConstants.HashEntity.FLAG, 				RedisConstants.HashEntity.FLAG_EXIST); //防穿透 插入一个flag为存在的值进去
}else {
	hashRedisTemplate.hset(key, RedisConstants.HashEntity.FLAG, 		RedisConstants.HashEntity.FLAG_NOT_EXIST); //插入一个flag为不存在的值进去
}
	hashRedisTemplate.expire(key, CommonConstants.Time.THREE_DAY); //设置key过期时间
}
return challengeMomentId;
}

@Override
public void addLastMomentId(Long lessonId, Long userId, Long momentId) {
    String key = RedisKeyUtil.build(RedisConstants.ChallengeMoment.LAST_MOMENT_ID, lessonId, userId);
    hashRedisTemplate.hset(key, RedisConstants.HashEntity.FLAG, RedisConstants.HashEntity.FLAG_EXIST);
    hashRedisTemplate.hset(key, RedisConstants.HashEntity.VALUE, String.valueOf(momentId));  //防穿透 插入一个flag为存在的值进去
    hashRedisTemplate.expire(key, CommonConstants.Time.THREE_DAY); //设置key过期时间
}
```



## 原理

hash中的flag字段来保持key一直存在..使用exist key判断是否需要查库.. 然后用flag的值来判断value是否存在..或者直接判断value是否存在. 请注意了解原理, 不用太在意常理问题. 这是从项目中抽取的代码片段