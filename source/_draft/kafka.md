# kafka 学习

## 下载软件

[kafka官网下载](http://kafka.apache.org/downloads.html)

## 简介

### 是什么

- 发布和订阅消息（流），在这方面，它类似于一个消息队列或企业消息系统。
- 以容错的方式存储消息（流）。
- 在消息流发生时处理它们。

### 用于什么地方

- 构建实时的流数据管道，可靠地获取系统和应用程序之间的数据(主要介绍此部分的内容)。
- 构建实时流的应用程序，对数据流进行转换或反应。

### 基本概念

1. Topic
1. Producer
1. Consumer
1. Broker
1. Log

#### 日志

1. Kafka 无论消息是否被消费都保存所有的消息,除非消息过期.
1. 消费者持有的数据就是消息的偏移量,偏移量由消费者控制,所以消费者可以重新读取每一条消息.

#### 分布式

Log的分区被分布到集群中的多个服务器上。每个服务器处理它分到的分区。 根据配置每个分区还可以复制到其它服务器作为备份容错。
每个分区有一个leader，零或多个follower。Leader处理此分区的所有的读写请求，而follower被动的复制数据。如果leader宕机，其它的一个follower会被推举为新的leader。 一台服务器可能同时是一个分区的leader，另一个分区的follower。

#### 生产者

生产者往某个Topic上发布消息。生产者也负责选择发布到Topic上的哪一个分区。最简单的方式从分区列表中轮流选择。也可以根据某种算法依照权重选择分区。开发者负责如何选择分区的算法。

#### 消费者

消费者用一个消费者组名标记自己。 一个发布在Topic上消息被分发给此消费者组中的一个消费者。 假如所有的消费者都在一个组中，那么这就变成了**queue模型**。假如所有的消费者都在不同的组中，那么就完全变成了**发布-订阅**模型

- 像传统的消息系统一样，Kafka保证消息的顺序不变
- Topic分区中消息只能由消费者组中的唯一一个消费者处理(正常情况分区数量应该大于等于消费者数量,否则会有消费者接收不到消息)
- kafka 保证Topic的一个分区顺序处理，不能保证跨分区的消息先后处理顺序。 所以，如果你想要顺序的处理Topic的所有消息，那就只提供一个分区。

## 常用命令

```bash
# 查看已经创建的topic
bin/kafka-topics.sh --list --zookeeper localhost:2181

# 查看topic详细信息
bin/kafka-topics.sh --list --zookeeper localhost:2181 --describe --topic test

# 生产消息
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

# 消费消息
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```

## 参考文章

- [kafka 推荐书籍](https://www.zhihu.com/question/56172498/answer/148006508)
- [kafka 命令](http://orchome.com/454)
- [kafka 教程](http://orchome.com/18)