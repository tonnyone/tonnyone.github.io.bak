---
title: kafka 学习系列 04--kafka的事务处理机制
category: kafka 
tags: [java, kafka]
date: 2018-06-12 10:10:00
---

## 简介

kafka提供了`at-least-once` 至少一次的语义保证。多余的消息可能由于生产者重试, 消费者失败重启等原因造成的。kakf-client 在`0.11.0.0`版本加入了对事务的支持提供了`exactly-once`的语义.

## 参考

- [transations-apache-kafka](https://www.confluent.io/blog/transactions-apache-kafka/)
- [Transactional+Messaging+in+kafka](https://cwiki.apache.org/confluence/display/KAFKA/Transactional+Messaging+in+Kafka)
- [KafKa-exactly-onece](https://hevodata.com/blog/kafka-exactly-once/)
