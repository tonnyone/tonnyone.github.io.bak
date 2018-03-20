# kafka 学习

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