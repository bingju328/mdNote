** 1,启动Kafka**

Kafka 使用 ZooKeeper 如果你还没有ZooKeeper服务器，你需要先启动一个ZooKeeper服务器。 您可以通过与kafka打包在一起的便捷脚本来快速简单地创建一个单节点ZooKeeper实例。如果你有使用docker的经验，你可以使用docker-compose快速搭建一个zk集群。

```
>bin/zookeeper-server-start.sh config/zookeeper.properties
现在启动Kafka服务器：
>bin/kafka-server-start.sh config/server.properties
```
后台启动：
```
>bin/kafka-server-start.sh config/server.properties 1>/dev/null  2>&1  &
```
其中1>/dev/null  2>&1 是将命令产生的输入和错误都输入到空设备，也就是不输出的意思。
/dev/null代表空设备。

** 2, 创建一个topic **

创建一个名为“test”的topic，它有一个分区和一个副本：
```
>bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
运行list（列表）命令来查看这个topic：
```
>bin/kafka-topics.sh --list --zookeeper localhost:2181 test
```
除了手工创建topic外，你也可以配置你的broker，当发布一个不存在的topic时自动创建topic。

Setp 4：发送消息
Kafka自带一个命令行客户端，它从文件或标准输入中获取输入，并将其作为message（消息）发送到Kafka集群。默认情况下，每行将作为单独的message发送。
运行 producer，然后在控制台输入一些消息以发送到服务器。

```
>bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
hello world
Hello study.163.com

给多个发
bin/kafka-console-producer.sh --broker-list node3:9092,node4:9092,node5
```

Setp 5：启动消费者
Kafka还有一个命令行使用者，它会将消息转储到标准输出。
```
>bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
hello world
hello study.163.com

多个接收
bin/kafka-console-consumer.sh --bootstrap-server node3:9092,node4:9092,node5:9092 --from-beginning --topic test
```
如果在不同的终端中运行上述命令，能够在生产者终端中键入消息并看到它们出现在消费者终端中。

### Kafka集群部署方式

设置多 broker 集群
到目前，我们只是单一的运行一个broker，对于Kafka，一个broker仅仅只是一个集群的大小，接下来我们来设多个broker。
首先为每个broker创建一个配置文件:
```
>cp config/server.properties config/server-1.properties
>cp config/server.properties config/server-2.properties
```
现在编辑这些新建的文件，设置以下属性：
```
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dir=/tmp/kafka-logs-1

config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dir=/tmp/kafka-logs-2
```
如果是多个不同服务器部署还需要配制如下
```
advertised.listeners=PLAINTEXT://node512:9092
```
broker.id属性是集群中每个节点的名称，这一名称是唯一且永久的。
我们已经建立Zookeeper和一个单节点了，现在我们只需要启动两个新的节点：
```
>bin/kafka-server-start.sh config/server-1.properties &
...
>bin/kafka-server-start.sh config/server-2.properties &
...
```
现在创建一个副本为3的新topic：
```
>bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
运行命令“describe topics” 查看集群中的topic信息
```
>bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```
以下是对输出信息的解释：第一行给出了所有分区的摘要，下面的每行都给出了一个分区的信息。因为我们只有一个分区，所以只有一行。
	“leader”是负责给定分区所有读写操作的节点。每个节点都是随机选择的部分分区的领导者。
	“replicas”是复制分区日志的节点列表，不管这些节点是leader还是仅仅活着。
	“isr”是一组“同步”replicas，是replicas列表的子集，它活着并被指到leader。
请注意，在示例中，节点1是该主题中唯一分区的领导者。

我们运行这个命令，看看一开始我们创建的那个test节点：
```
>bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: testPartition: 0    Leader: 0   Replicas: 0 Isr: 0
```
这并不奇怪，刚才创建的主题没有Replicas，并且在服务器“0”上，我们创建它的时候，集群中只有一个服务器，所以是“0”。
发布一些信息在新的topic上：
```
>bin/kafka-console-producer.sh --broker-list localhost:9092 --topicmy-replicated-topic
...
my testmessage 1
my testmessage 2
```
消费这些消息：
```
>bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my testmessage 1
my testmessage 2
```
测试集群的容错，kill掉leader，Broker1作为当前的leader，也就是kill掉Broker1。
```
>psaux | grepserver-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
>kill-9 7564
```
备份节点之一成为新的leader，而broker1已经不在同步备份集合里了。
```
>bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```
即使最初接受写入的领导者已经失败，这些消息仍可供消费：
```
>bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
```
