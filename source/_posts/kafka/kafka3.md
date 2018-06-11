---
title: kafka 学习系列 03-kafka消费者
category: kafka 
tags: [java, kafka]
date: 2018-06-04 10:10:00
---

## 消费者群组

为了横向伸缩,同属于一个组内的消费者只消费一个主题的部分消息，多个消费者可以对消息进行分流.Topic分区中消息只能由消费者组中的唯一个消费者处理(正常情况分区数量应该大于等于消费者数量,否则会有消费者会闲置)

![](/uploads/kafka/kafka_consumer.png)

<!--more-->

### 消费者群组以及分区再均衡

群组的消费者共同读取主题中的分区，当一个新的消费者加入群组或者当一个消费者被关闭或崩溃时，它就离开群组.分区的所有权从一个消费者转移到另一个消费者，这样的行为称为`分区再均衡`.

1. 消费者通过向指派为`群组协调器`的broker(不同的群主有不同的群组协调器)发送心跳.来维持他们和群主的从属关系以及它们对分区的所有权关系
2. 只要消费者正常的时间间隔发送心跳，就被`协调器`认为是活跃的。
3. 消费者会在轮询消息或者提交偏移量的时候发送心跳
4. 如果消费者停止发送心跳的时间过长会话就会过期, `群组协调器`会认为其已经死亡,就会触发一次再均衡。
5. 如果一个消费者发生死亡，群主协调器会等待几秒钟确定死亡了，才会触发再均衡。在这几秒钟的时间里，死亡的消费者不会消费分区中的消息。在清理消费者时，消费者会通知`群组协调器`它要离开群组，立即触发一次分区再均衡。

分配分区的过程:

![](/uploads/kafka/kafka_consumer2.png)

1. 当消费者要加入群组时，会向`群组协调器`发送一个JoinGroup 请求,第一个加入群组的消费者将成为`群主`
2. 群主从协调器那里获得群组成员列表(列表中包含了最近发送心跳的消费者，它被认为时活跃的)，并负责给每个消费者分配分区。
3. 群主使用一个实现了PartitionAssignor接口的类来决定哪些分区要被分配给哪个消费者。
4. 分配结束后群主把分配情况发送给`群组协调器`,协调器再把这些信息发送给所有消费者。每个消费者只能看到自己的分配信息，群主知道群组里面所有消费者的分配情况。
5. 这个过程每次分区再均衡的时候发生一次

## kafka 消费者API

```java
Properties props = new Properties();
props.put("bootstrap.servers","192.168.200.219:19092");
props.put("group.id","testgroup2");
props.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String,String> consumer = new KafkaConsumer<String,String>(props);
// 1. 消费者订阅主题，此处一次订阅多个主题，或者可以使用正则表达式
consumer.subscribe(Arrays.asList("test"));
try{
  while (true){
    // 2. 消费者必须持续向kafka轮询，否则就会被认为死亡。
    ConsumerRecords<String, String> cr = consumer.poll(100);
    for (ConsumerRecord<String, String> record : cr) {
        System.out.printf("record:%s \n",record.toString());
    }
  }
}finally {
  // 3.在退出程序之前使用close()方法关闭消费者。
  consumer.close();
}
```

1. 消费者订阅主题,可以同时订阅多个主题，或者使用正则表达式
2. `poll()`方法轮询kafka获取数据(必须持续的对kafka进行轮询否则消费者会被认为已经死亡,轮询不止获取数据这么简单，在第一次调用poll方法的时候，它会负责查找GroupCoordinator,然后加入群组，接受分配的分区，如果发生了在均衡整个过程也是在轮询期间进行的。当然心跳也是从轮询中发送出去的，所以我们应该确保轮询期间所做的任何工作尽快完成)。`poll`方法包含一个参数控制阻塞的超时时间。该参数如果设置为0，poll方法立即返回。否则它会在指定的毫秒数内一直等待broker返回数据。`poll`会返回一个记录列表.每条记录包含了主题记录的信息(分区信息，偏移量，键值对).无论是否有数据，`poll`方法都会返回，超时时间取决于应用程序对响应速度的要求，
3. 在退出程序之前使用close()关闭消费者。网络连接和socket随即会被关闭。并立即触发一次再均衡,而不是等待群组协调器发现它不在发送心跳并认定已死亡，因为那样需要更长时间，导致整个群组再一段时间内无法读取消息。

### 提交和偏移量

kafka 是如何记录消费者的偏移量的呢？ 在早期的版本里面消费者的偏移量是记录在`zookeeper`里面的，从0.9版本开始，偏移量记录在一个名为`__consumer_offset`的topic里面.如果消费者发生崩溃或者有新的消费者加入群组，就会触发在均衡。完成再均衡后，每个消费者可能分配到新的分区。消费者读取此分区最后一次的提交偏移量,继续处理

1. 如果之前的消费者消费了消息但是来不及提交崩溃了，再均衡的时候，未提交偏移量的这部分消费会被重复处理。(这就是kakfa本身不能保证消息只被处理一次的原因)。
2. 消费者也可以提交大于它当前正处理消息的偏移量，跳过某些消息。

#### 自动提交偏移量

如果消费者的`enable.auto.commit` 的配置项被设置为 `true` ,那么消费者会默认每隔`5s`就会自动把从poll接收到的最大偏移量提交上去。提交时间由`auto.commit.interval.ms`控制，单位为秒.
自动提交偏移量也是发生在轮询中，每次轮询，消费者会检测是否到了该提交偏移量了的时候，如果是，那么提交上次poll返回的偏移量。如果在最近一次提交后3s发生了`再均衡`,那么待消费者重新分配完分区后这3s的消费就会被重复处理。

#### 手动提交偏移量

消费者把`auto.commit.offset`设置为`false`,可以手动控制偏移量

##### 同步提交

`commitSync()`方法会将上次poll方法返回的偏移量提交。只要没有发生不可恢复的错误，commitSync()方法会一直尝试提交直到成功。如果提交失败，我们可以把错误日志记录下来。如果在提交过程中发生再均衡，最近的一批消息在再均衡后还会被重复处理.

```java
while (true){
    //2. 消费者必须持续向kafka轮询，否则就会被认为死亡。
    ConsumerRecords<String, String> cr = consumer.poll(100);
    for (ConsumerRecord<String, String> record : cr) {
        System.out.printf("record:%s \n",record.toString());
    }
    // 4 .同步提交
    try {
        consumer.commitSync();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

##### 异步提交

`commitAsync` 方法不会重试，这是异步提交不好的一个地方。不重试的原因是，可能在此次提交失败后，另一个更大的偏移量已经提交.`commitAsync`支持回调，在broker做出响应的时候会执行回调。回调一般用于记录提交错误或者生成指标。

```java
while (true){
    //2. 消费者必须持续向kafka轮询，否则就会被认为死亡。
    ConsumerRecords<String, String> cr = consumer.poll(100);
    for (ConsumerRecord<String, String> record : cr) {
        System.out.printf("record:%s \n",record.toString());
    }
    // 4 .异步提交
    try {
        consumer.commitAsync(new OffsetCommitCallback() {
            public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                if(exception!=null) {
                    System.out.printf("提交失败\n");
                }
            }
        });
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

##### 同步加异步提交

一般情况下，针对偶尔出现的提交失败，不进行重试不会有太大的问题，因为如果提交失败是由临时问题导致的，那么后续的提交总是会成功的。因此消费者关闭前，一般会组合使用同步和异步提交，这样确保最后一次提交能够成功。如下:

```java
try{
    while (true){
        //2. 消费者必须持续向kafka轮询，否则就会被认为死亡。
        ConsumerRecords<String, String> cr = consumer.poll(100);
        for (ConsumerRecord<String, String> record : cr) {
            System.out.printf("record:%s \n",record.toString());
        }
        consumer.commitAsync();
    }
}catch (Exception e){
    System.out.println(e.getMessage());
}finally {
    try{
        consumer.commitSync();
    }finally {
        //3.在退出程序之前使用close()方法关闭消费者。
        consumer.close();
    }
}
```

#### 提交特定的偏移量

之前提交的方式都是按照`poll()`方法同步，按照如果想根据按照消息数量来提交。下面的方法每隔`1000`条消息提交偏移量。

```java
 // 1. 用于跟踪各个分区的偏移量
Map<TopicPartition,OffsetAndMetadata> currentOffsets = new HashMap();
int count=0;
try{
    while (true){
        //2. 消费者必须持续向kafka轮询，否则就会被认为死亡
        ConsumerRecords<String, String> cr = consumer.poll(100);
        for (ConsumerRecord<String, String> record : cr) {
            System.out.printf("record:%s \n",record.toString());
            currentOffsets.put(new TopicPartition(record.topic(),record.partition()),new OffsetAndMetadata(record.offset()+1,"no metadata"));
            // 3. 如果消费数量到1000 就提交偏移量
            if(count % 1000==0){
                consumer.commitAsync();
            }
            count++;
        }
    }
}catch (Exception e){
    e.printStackTrace();
}finally {
    //4.在退出程序之前使用close()方法关闭消费者。
    consumer.close();
}
```

### 再均衡监听器

1. 实现ConsumerRebalanceListener接口
2. 如果发生再均衡，先要把即将失去的分区读取的偏移量全部都体提交。
3. 消费者订阅消息的时候把`ConsumerRebalanceListener`传给`subscribe()`方法

```java
private static Map<TopicPartition,OffsetAndMetadata> currentOffsets = new HashMap();
private static KafkaConsumer<String,String> consumer = new KafkaConsumer<String,String>(props);

// 1. 实现ConsumerRebalanceListener 接口,实现消费者再均衡监听器
private static class HandleRebalance implements org.apache.kafka.clients.consumer.ConsumerRebalanceListener{
    //2. 方法会在再均衡开始之前和消费者停止读取消息之后被调用,在这里提交已经读取的偏移量，下一个接管分区的消费者就会继续从这里开始读取
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        System.out.println("Lost paritions in rebalance,commit current offset"+currentOffsets);
        consumer.commitSync(currentOffsets);
    }

    // 3. 方法会在在均衡之后，就是重新分配分区之后，新的消费者开始读取消息之前被调用
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {}
}

// 4. 注册再均衡监听器
consumer.subscribe(Arrays.asList("test"),new HandleRebalance());
try{
    while (true){
        //5. 消费者必须持续向kafka轮询，否则就会被认为死亡
        ConsumerRecords<String, String> cr = consumer.poll(100);
        for (ConsumerRecord<String, String> record : cr) {
            System.out.printf("record:%s \n",record.toString());
            currentOffsets.put(new TopicPartition(record.topic(),record.partition()),new OffsetAndMetadata(record.offset()+1,"no metadata"));
        }
        consumer.commitAsync(currentOffsets,null);
    }
}catch (WakeupException e){
}catch (Exception e){
    e.printStackTrace();
}finally {
    try{
        //6.在退出程序之前使用close()方法关闭消费者。
        consumer.commitSync(currentOffsets);
    }finally {
        consumer.close();
        System.out.println("consumer has been Closed");
    }
}
```

### 从特定的偏移量开始处理记录

api 提供了`seekToBeginning(Collection<TopicPartition>, tp)` 和 `seekToEnd(Collection<TopicPartition> tp)`,`seek()`等方法，可以直接跳转到指定的位置进行读取记录。有人尝试把偏移量保存到数据库里面，让处理数据和提交偏移量再一个事务里面处理，这样保证了数据的完整和唯一性。

### 如何退出

如果消费者要退出循环.唯一一个可以从其他线程安全调用的方法`consumer.wakeup()`。调用`consumer.wakeup()`可以退出poll().并抛出WakeupException异常，如果效用consumer.wakeup()时，线程没有再等待poll方法，那么他会在下一轮循环跳出。在退出线程之前调用`consumer.close()`时必要的，它会提交任何没有提交的东西，并且向群组协调器发送消息，告知自己要离开群组，触发再均衡，而不需要等待会话超时。

### 消费者关键配置一览

#### `fetch.min.bytes`

该属性指定了消费者从服务器获取记录的最小字节数。当broker收到消费者数据请求时，如果可用的数据量小宇fetch.min.bytes指定大小，那么它会等到有足够的可用数据时才把它返回给消费者.

#### `fetch.max.wait.ms`

此配置用于指定broker的等待时间，默认是`500ms`,如果没有足够的数据流入kafka,消费者获取的最小数据量达不到满足，到500ms,就会返回给消费者.

#### `max.partition.fetch.bytes`

此配置指定了服务器从每个分区返回给消费者的最大字节数，默认值为`1M`,即`poll()`方法从每个分区返回的记录最多不超过`max.partition.fethc.bytes`指定的字节数。

#### `session.timeout.ms`

该属性指定了消费者被认为死亡之前可以与服务器断开的连接时间,默认为`3s`,也就是说如果没有在`session.timeout.ms`指定的时间内发送心跳给群组协调器，就会被认为死亡触发再均衡。该属性与`heartbeat.interval.ms`密切相关。`heartbeat.interval.ms`指定了`poll()`方法向协调器发送心跳的频率。所以`heartbeat.interval.ms`一定要比`session.timeout.ms`小，一般为`session.timout.ms`的三分之一。

#### `auto.offset.reset`

该属性指定了消费者再读取一个没有偏移量的分区或者偏移量无效的情况下，该作何处理,默认值为`lastest`,从最新的记录开始读取,另一个值`earliest`,意思说，在偏移量无效的情况下，从起始位置开始读取

#### `enable.auto.commit`

指定了消费者是否自动提交偏移量,默认为`true`,可以设置为`false`,进行手动提交

#### `partition.assignmeng.strategy`

分区分配策略,默认kafka有两种`Range`和`RoundRobin`,当然可以自定义

#### `client.id`

broker用它来表示从客户端发送过来的消息，通常用在日志，指标监控中。

## 参考

- [kafka 官网](https://cwiki.apache.org/confluence/display/KAFKA/Index)
- [kafka 权威指南](https://book.douban.com/subject/27665114/)