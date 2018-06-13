---
title: kafka 学习系列 04-spring boot 集成 spring-kafka
category: kafka 
tags: [java, kafka]
date: 2018-06-12 10:10:00
---

## spring和kafka的集成的三种方式

- spring-kafka
- spring-integration for kafka
- spring cloud stream kafka

以上每一种都在上一种的基础上进行封装集成。spring integretion 2.0以前版本并不基于spring for kafka

|spring for kafka |spring integration for kafka | kafka-clients
|:-|:-|--|
|2.2.*|3.1.*|1.1.*|
|2.1.*|3.0.*|1.0.\*,1.1.\*|
|2.0.*|3.0.*|0.11.0.\*,1.0.\*|
|1.3.*|2.3.*|0.11.0.\*,1.0.\*|
|1.2.*|2.2.*|0.10.2.*|
|1.1.*|2.1.*|0.10.0.\*,0.10.1.*|
|1.0.*|2.0.*|0.9.*.\*|
|N/A *|1.3.*|0.8.2.2|

说明:大于0.10.2.0版本的kafka客户端，都可以与更旧版本kafka服务端通信，所有大于`0.10.x.x`推荐按使用spring-kafka 版本 1.3.x 或以上。

## 参考

- [spring-kafka](https://spring.io/projects/spring-kafka#learn)
