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


### 从特定的偏移量开始处理记录

api 提供了`seekToBeginning(Collection<TopicPartition>, tp)` 和 `seekToEnd(Collection<TopicPartition> tp)` 这两个方法。

### 如何退出

### 消费者关键配置一览

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

重试次数: 生产者发送从服务器收到的错误消息可能是临时性的错误(比如分区找不到Leader),这种情况,retrise参数决定了生产者可以重试的次数,如果达到这个次数生产者会放弃重试，返回错误,默认情况重试会等待100ms,可以通过`retry.badkoff.ms`来修改.不过有些错误不是临时错误,比如(消息太大的错误),业务处理一般只处理不可重试的错误,或者重试次数超出上限的情况.如果设置了

#### `batch.size`

当多个消息被发送到同一个分区的时候，生产者会把它放入到一个批次当中提交，该参数指定了一个批次可使用的内存大小，按照字节数计算，当批次被填满是就会被发送出去，不过生产者生产者并不一定会等批次被填满才会去发送,详见`linger.ms`参数， 所以就算把批次设置的很大也不会造成延迟，只会增加更多多的内存而已，但是如果设置的很小，生产者就会频繁的发送消息，会增加额外的开销

#### `linger.ms`

该参数指定了生产者在发送批次之前等待更多消息加入批次的时间，生产者会在批次被填满或者等待时间达到该参数的值的时候消息被发送出去。把`linger.ms`设置为比0大的值，让生产者发送到批次的时候等待一会儿，虽然这样会增加延迟，但是会提高kafka的吞吐量.

#### `client.id`

该参数如果被设置，服务器会用来标识消息的来源

#### `max.block.ms`

该参数指定在调用send方法或使用partitionFor方法获取元数据的时候，生产者的阻塞时间。当生产者发送的缓存区已满，或者没有可用的元数据时，方法就会阻塞，阻塞时间达到`max.block.ms`时，生产者会抛出异常

#### `max.request.size`

生产者发送单个消息的最大值,也可以指单个请求中所有消息的大小。另外，broker对可接收消息最大值也有自己的限制`message.max.bytes`指一个批次发送到broker的最大值，这个值应该是`max.request.size`的整数倍

#### `max.in.flight.requests.per.connection`

该参数指定了生产者在服务器收到响应之前可以发送多少个消息。它的值越大就会占用越多的内存提高吞吐量。把它设置为1 可以保证消息是按照顺序写入服务器的，但是如果retries不为0的话，消息的先后顺序会发生错位,假设第一条消息失败了，第二条先发送成功了，然后第一条又重试成功了，这个时候发送的服务器的顺序就出现了错位。

## 参考

- [kafka 官网](https://cwiki.apache.org/confluence/display/KAFKA/Index)
- [kafka 权威指南](https://book.douban.com/subject/27665114/)
- [聊聊spring集成kafka的集中方式](https://segmentfault.com/a/1190000011454052#articleHeader4)