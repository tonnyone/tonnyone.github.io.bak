---
title: kafka 学习系列 02-kafka生产者
category: kafka 
tags: [java, kafka]
date: 2018-05-20 22:00:00
---

## kafka 生产者流程

下图展示了kafka发送消息的整体流程

1. 创建一个ProductRecord对象(对象包含主题名称,分区,key,value).
1. 把key和value 序列化成字节数组.
1. 如果ProductRecord对象没有指定了分区,那么分区器会通过key选择一个分区,如果指定了分区的话直接返回
1. 被序列化后的这条记录会被发往一个记录批次里,这个批次里面的所有消息都是发往相同的主题和相同的分区的
1. 会有专门的线程把这些记录批次发到响应的broker上
1. 服务器收到消息会返回一个响应,如果写如成功就会返回一个RecordMetaData对象(包含了主题和分区信息以及分区的偏移量)
1. 如果不成功,则会返回一个错误,某些错误在生者收到后会重新尝试发送消息,几次后如果还是失败,就返回失败信息

<!--more-->

![](/uploads/kafka/kafka_produce.png)

### java api 解析

```java
Properties props = new Properties();
props.put("bootstrap.servers","192.168.200.219:19092");
props.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
KafkaProducer producer = new KafkaProducer<String,String>(props);

String topic = "test";
ProducerRecord<String,String> producerRecord = new ProducerRecord<String, String>(topic,"1","nihao");
producer.send(producerRecord);
```

#### send方法是生产者发送生产消息的关键方法,对于发送消息的处理有三种方式

1. 发送不管无论结果是否成功
2. 发送消息后阻塞直到消息发送成功
3. 发送消息,异步获取成功结果

```java
//1. 直接发送,无论是否成功
producer.send(producerRecord);
//2. 获取future,阻塞获取结果
Future<RecordMetadata> future = producer.send(producerRecord);
RecordMetadata recordMetadata = null;
try {
  recordMetadata = future.get();
} catch (InterruptedException e) {
  e.printStackTrace();
} catch (ExecutionException e) {
  e.printStackTrace();
}
System.out.println("发送成功"+recordMetadata.toString());
//3. 成功后执行回调
producer.send(producerRecord, new Callback() {
  public void onCompletion(RecordMetadata recordMetadata, Exception e) {
      System.out.println("发送成功CallBack:"+recordMetadata.toString());
  }
});
```

### 生产者关键配置

#### `bootstrap.servers` 

broker地址清单，生产者会从给定的groker里面查找到其他的broker的信息。建议配置多个，一旦其中一个宕机，生产者仍然能够连到集群上。

#### `key.serializer`

必须 设置为 `org.apache.kafka.common.serialization.Serializer`接口类，用于将消息key序列化为字节数组

#### `value.serializer` 

必须 设置为 `org.apache.kafka.common.serialization.Serializer`接口类，用于将消息value序列化为字节数组

#### `acks` 

指定了集群中多少个分区副本收到消息,生产者才认为消息是写入成功的.

1. acks=0 生产者不会等待任何分区副本的确认，如果配置`retries`,则配置无效,如果写入错误生产者是获取不到的,返回的offset永远都是`-1`.
2. acks=1 只要分区副本的`leader`被写入消息后,生产者就会认为消息写入成功.如果消息无法到达`leader`(比如leader崩溃后还没有选举出来),生产者会收到错误的响应重发,但是如果写入`leader`的消息没来得及同步到其他副本,挂掉了，此时还是会发生消息丢失.
3. acks=all 只有全部参与复制的节点收到都收到消息后才认为消息写入是成功的,就算有副本发生崩溃整个集群仍然可用.不过此选项写入消息的延迟会更高。

#### `buffer.memory`

用来设置生产者消息内存缓冲区的大小.如果消息发送的太快，就会导致缓冲区的空间不足.这个时候`send`方法要么被阻塞，要么抛出异常.`max.blocks.ms`设置了可阻塞的时长.

#### `compression.type`

压缩类型,指定了发送给broker的消息压缩算法,压缩可以降低网络传输和存储的开销,但是增加了CPU的计算压力

- none
- gzip
- anappy
- lz4

#### `retries`

重试次数: 生产者发送从服务器收到的错误消息可能是临时性的错误(比如分区找不到Leader),这种情况,retrise参数决定了生产者可以重试的次数,如果达到这个次数生产者会放弃重试，返回错误,默认情况重试会等待100ms,可以通过`retry.badkoff.ms`来修改.不过有些错误不是临时错误,比如(消息太大的错误),业务处理一般只处理不可重试的错误,或者重试次数超出上限的情况.