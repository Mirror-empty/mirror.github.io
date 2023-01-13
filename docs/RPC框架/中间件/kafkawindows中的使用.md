# kafkawindows中的使用

1. [官网下载安装包](http://kafka.apache.org/downloads.html)
2. 在kafka_2.13-2.8.0\bin\windows 目录下启动以下命令
3. 启动ZooKeeper ``.\zookeeper-server-start.bat ..\..\config\zookeeper.properties``
4. 启动kafka  ``.\kafka-server-start.bat ..\..\config\server.properties ``
5. 创建主题 ``.\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka-test-topic``
6. 查看主题 ``.\kafka-topics.bat --list --zookeeper localhost:2181 ``
7. 启动生成者 ``.\kafka-console-producer.bat --broker-list localhost:9092 --topic kafka-test-topic `` 主题名
8. 启动消费者 ``.\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic kafka-test-topic --from-beginning ``

### SpingBoot整合Kafka

**1.POM**

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>

```

**@KafakListener注解**

属性：

- id：消费者的id，当GroupId没有被配置的时候，默认id为GroupId
- containerFactory：上面提到了@KafkaListener区分单数据还是多数据消费只需要配置一下注解的containerFactory属性就可以了，这里面配置的是监听容器工厂，也就是ConcurrentKafkaListenerContainerFactory，配置BeanName
- topics：需要监听的Topic，可监听多个
- topicPartitions：可配置更加详细的监听信息，必须监听某个Topic中的指定分区，或者从offset为200的偏移量开始监听
- errorHandler：监听异常处理器，配置BeanName
- groupId：消费组ID
- idIsGroup：id是否为GroupId
- clientIdPrefix：消费者Id前缀
- beanRef：真实监听容器的BeanName，需要在 BeanName前加 "__"

参数：

- data ： 对于data值的类型其实并没有限定，根据KafkaTemplate所定义的类型来决定。data为List集合的则是用作批量消费。
- ConsumerRecord：具体消费数据类，包含Headers信息、分区信息、时间戳等
- Acknowledgment：用作Ack机制的接口
- Consumer：消费者类，使用该类我们可以手动提交偏移量、控制消费速率等功能