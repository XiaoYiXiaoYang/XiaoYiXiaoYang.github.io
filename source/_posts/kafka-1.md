---
title: Kafka介绍
date: 2020-08-24 22:33:03
tags: [Kafka]
categories: [学习笔记]
---

一个有名的多分区多副本且基于ZooKeeper协调的分布式消息系统，高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性被使用。

<!--more-->

- 消息系统：Kafka和传统的消息系统都具备系统解耦性、冗余存储、流量削峰、缓冲、异步通信、扩展性、可恢复性等功能。Kafka还提供大多数消息系统难以实现的消息顺序性保障和回溯消费功能
- 存储系统：Kafka把消息持久化到磁盘，相比于其他基于内存存储的系统而言，有效地降低了数据丢失的风险。也正是得益于Kafka的消息持久化功能和多副本机制，我们可以吧Kafka作为长期对的数据存储系统来使用。
- 流式处理平台：Kafka不仅为每个流行的流式处理框架提供了可靠的数据来源，还提供了一个完整的流式处理类库



# 第一章初识Kafka



## 基本概念



- Producer：生产者，也就是发送消息的一方，生产者负责创建消息，然后将其投递到Kafka中
- Consumer：消费者，也就是接收消息的一番。消费者连接到Kafka上并接收消息，进而进行相应的业务逻辑处理
- Broker：服务代理结点：对于Kafka而言，Broker可以简单的看做一个独立的Kafka服务结点或Kafka服务实例。
- 主题：Kafka中的消息以主题为单位进行归类，生产者负责将消息发送到特定的主题，而消费者负责订阅主题并进行消费
- 分区：一个分区只属于单个主题，同一主题下不同分区包含的消息是不同的，分区在存储层面可以看做一个可追加的日志文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量。
- Offset偏移量：offset是消息在分区中的唯一标识，Kafka通过它来保证消息在分区内的顺序性，不过offset并不跨越分区，Kafka保证分区有序而不是主题有序
- 副本因子：副本个数
- AR：分区内所有副本
- ISR:所有与leader副本保持一定程度同步的副本
- OSR:与leader副本同步滞后过多的副本
- HW:高水位：标识一个特定的offset，消费者只能拉取到这个offset之前的消息
- LEO: Log End Offset 标识当前日志文件中下一条待写入消息的offset



## 生产与消费



```
./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-demo --replication-factor 3 --partitions 4


./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic topic-emo

./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic demo

./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic topic-demo
```



# 第二章 生产者



步骤：

1. 配置生产者客户端参数及创建相应的生产者实例
2. 构建待发送的消息
3. 发送消息
4. 关闭生产者实例

```
public static Properties initConfig() {
	Properties props = new Properties();
	props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
}

public static void main(String[] args) {
	Properties props = initConfig();
	KafkaProducer<String, String> producer = new KafkaProducer<>(props);
	ProducerRecord<String, String> record = new ProducerRecord<>(topic, "Hello, Kafka");
	
	try {
		producer.send(record);
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```



消息对象ProducerRecord 并不是单纯意义上的消息，包含了多个属性

```java
public class ProducerRecord<K, V> {
	private final String topic;
	private final Integer paitition;
	private final Headers headers;
	private final K key;
	private final V value;
	private final Long timestamp;
}
```



- KafkaProducer是线程安全的，可以在多个线程中共享单个KafkaProducer实例，也可以将KafkaProducer实例进行池化来供其他线程调用

- 构建ProducerRecord 对象，topic属性和value属性是必填，其他选填



**发送消息**



发后即忘：send方法不指定Callback，性能最高，可靠性最差

同步：send方法利用返回的Future对象，阻塞等待Kafka响应

异步：send方法，指定Callback回调函数



可重试异常和不可重试异常

对于可重试异常，如果配置了retries参数，那么只要在规定的重试次数内自行恢复，就不会抛出异常

对于不可重试的异常，则直接抛出异常，不进行重试



对于同一个分区而言，如果消息record1先与record2发送，那么KafkaProducer就可以保证对应的callback1先与callback2调用



**序列化器**

生产者需要使用序列化器将对象转换成字节数组，才能通过网络发送给Kafka，在对端消费者使用反序列化器把Kafka转换成相应的对象



序列化器实现了org.apache.kafka.common.serialization.Serializer接口

一般要实现

```
public void configure(Map<String, ?>configs,boolean isKey)
public byte[]serialize(String topic, T data)
public void close()
```



可以使用Avro、JSON、Thrift、Protobuf、Protostuff等通用工具来实现



**分区器**



```
public int partition(String topic, Object key,byte[] keyBytes, Object Value, byte[] valueBytes,Cluster cluster);
public void close();
```



- 如果ProducerRecord中指定了Partition字段，则不需要分区器，partition字段就是要发往的分区号

- 如果没有指定分区器，就需要分区器根据key字段来计算partition值。Kafka的默认分区器实现了 xx.Partitioner接口，接口中有partition方法和close方法

  默认分区器会判断key不为null，则对key进行哈希，最终根据得到的哈希值来计算分区号，拥有相同key的消息会被写入同一个分区。如果key为null，那么消息会以轮询的方式发往主题内的某一个可用分区

  

  自定义分区器也只需实现上述接口即可



**生产者拦截器**

消息发送前做一些过滤，修改等等

需要自定义实现ProducerInterceptor接口



KafkaProducer会在消息被应答之前或消息发送失败时调用拦截器的onAcknowledgement方法，优于用户设定的Callback之前执行。

```java
public ProducerRecord<K, V> onSend(ProducerRecord<K, V> record);
public void onAcknowledgement(RecordMetadata metadata, Exception exception);
public void close();
```



可以指定一个拦截链，KafkaProducer按照interceptor.classes参数配置的拦截器的顺序来一一执行（各个拦截器按逗号隔开）



**整体架构**



![生产者客户端整体架构](producer-structure.jpg)



生产者客户端有两个线程，主线程和Sender线程。主线程生产消息经过拦截器、序列化器、分区器缓存到消息累加器中，Sender线程从RecordAccumulator中获取消息并发往Kafka中



buffer.memory： 指定RecordAccumulator缓存的大小

max.block.ms：指定生产者发送太快，缓冲区满了，阻塞的最大时间

RecordAccumulator缓存的大小由buffer.memory配置；如果生产者发送消息的速度超过发送到服务器的速度，则会导致生产者空间不足，这时候producer的send方法调用要么被阻塞，要么抛出异常，这个取决于参数max.block.ms的设置。





RecordAccumulator为每个分区维护一个双端队列，队列内容为ProducerBatch，ProducerBatch为一个至多个ProducerRecord；可以使得生产者创建的消息组成一个批次，更为紧凑。



消息在网络上传输是以字节传输的，发送之前要创建内存区域。kafka生产者中，通过java.io.ByteBuffer实现消息内存创建和释放。RecordAccumulator内部还有一个BufferPool，实现ByteBuffer的复用。BufferPool只针对特定大小的ByteBuffer进行管理，这个大小由batch.size指定。

batch.size 指定ByteBuffer的大小



ProducerBatch的大小和batch.size相关。当一条ProducerRecord消息到了RecordAccumulator，会先寻找与分区对应的双端队列(如果没有则新建)，再从尾部获取一个ProducerBatch，查看该ProducerBatch中是否还可以写入这个ProducerRecord，可以写入则写入，不可以写入则新建ProducerBatch。

新建ProducerBatch时，判断这条ProducerRecord消息大小是否超过batch.size没超过，则就以batch.size的大小新建ProducerBatch，这段内存还可以由ByterBuffer复用；如果超过了则以**评估的大小？**新建ProducerBatch，这段内存不会被复用。



Sender从RecordAccumulator获取缓存的消息后，进一步将原本的

```
<分区，Deque<ProducerBatch>>
```

转换为

```
<Node, List<ProducerBatch>>
```

Node表示kafka集群的结点。生产者向具体的broker结点发消息。

Sender还会进一步封装为

```
<Node, Request>
```

才发往各个Node



请求在从Sender发往kafka之前会保存到InFlightRequests中，保存形式为

```
Map<NodeId, Deque<Request>>
```

主要作用是缓存了已经发出去，但是还没有收到响应的请求。

这里限制了每个连接最多缓存的请求数，由max.in.flight.requests.per.connecttion



**元数据的更新**



上一节提到的inFlightRequests可以获得leastLoadedNode，即所有Node中负载最小的。

Node中未确认的请求越多，则认为负载越大。



选择leastLoadedNode发送请求可以使它能尽快发出，避免网络拥塞等异常的影响。

leastLoadedNode还可以用于**元数据请求**、**消费者组播协议的交互**



如果发送一个很简单的消息

```
ProducerRecord<string, string> record = new ProducerRecord<>(topic, "hello");
```

这里只有主题和消息

KafkaProducer需要将消息追加到指定主题的某个分区的对应leader副本之前。需要知道分区数目，计算出目标分区，需要知道目标分区的leader副本所在broker结点的地址、端口信息。这些需要的信息都属于**元数据信息**。



bootstrap.servers参数只需要配置部分broker结点的地址，客户端可以发现其他broker结点的地址，这一过程属于元数据更新。



客户端没有元数据信息时，会先选出leastLoadedNode，然后向这个Node发送MetadataRequest请求来获取具体的元数据信息。这个更新操作由Sender线程发起，在创建完MetadataRequest后同样会存入inFlightRequests。元数据虽然由Sender线程负责更新，但是主线程也需要读取这些信息，这里数据同步通过synchronized 和 final关键字保障。



**重要的生产者参数**

acks：指定分区中必须要有多少副本收到这条消息，之后生产者才会认为这条消息是成功写入的。

默认acks = 1

acks = 0 生产者发送消息之后不需要等待任何服务端响应



max.request.size 客户端能发送消息的最大值



retries 和 retry.backoff.ms

retries是生产者重试的次数

retry.backoff.ms 两次重试之间的间隔默认100ms



compression.type 默认值为none，指定消息压缩



connections.max.idle.ms 指定在多久之后关闭闲置的连接



linger.ms 指定生产者发送ProducerBatch之间等待更多ProducerRecord加入的时间



receive.buffer.bytes 设置Socket接收消息缓冲区大小  默认32kb



send.buffer.bytes Socket发送消息缓冲区大小



request.timeout.ms 配置Producer等待请求响应的最长时间，默认30000ms





# 第三章 消费者



**消费者与消费者组**

每个消费者都有一个对应的消费者组。当消息发布到主题后，只会被投递给订阅它的每个消费组中的一个消费者。

每个消费组消费全部分区的消息。

消费者与消费组这种模型又可以让整体的消费能力具备横向伸缩性，我们可以增加消费者的个数来提高整体的消费能力。对于分区数固定的情况，一直增加消费者，到消费者个数超过分区数，就会有消费者分配不到分区。



点对点模式：基于队列，消息生产者发送消息到队列，消费者从消息队列中接收消息。

发布订阅模式：定义了如何想一个内容节点发布和订阅消息，这个内容节点叫做主题，主题可以认为是消息传递的中介，消息发布者将消息发布到某个主题，而消息订阅者从主题中订阅消息。主题使得消息的订阅者和发布者互相保持独立，不需要进行接触即可保证消息的传递，发布订阅模式在消息的一对多广播时采用。

kafka同时支持两种消息投递模式。

- 如果所有的消费者隶属于一个消费者组，那么所有的消息都会被均衡的投递给每一个消费者，即每条消息只会被一个消费者处理，这相当于点对点。
- 如果所有的消费者隶属于不同的消费组，那么所有的消息都会被广播给所有的消费者，即每条消息都会被所有的消费者处理，相当于发布订阅模式应用。



消费组是一个逻辑概念，每个消费者在消费前需要指定所属消费组的名称，由group.id指定。消费者是实际的应用实例，可以是一个线程，也可以是一个进程，同一个消费组的消费者既可以部署在同一机器上，也可以部署在不同机器上。



**客户端开发**

1. 配置消费者客户端参数以及创建相应的消费者实例
2. 订阅主题
3. 拉取消息并消费
4. 提交消费位移
5. 关闭消费者实例

```
public class KafkaConsumerAnalysis {
	public static final String brokerList = "";
	...
	
	public static Properties initConfig() {
		Properties props = new Properties();
		props.put("bootstrap.servers", brokerList);
	}
	
	public static void main() {
		Properties props = initConfig();
		KafkaConsumer<String, String> consumer = new KafkaConsmer<>(props);
		consumer.subscribe(Arrays.asList(topic));
		
		try {
			while(isRunning.get()) {
				ConsumerRecrds<String, String> records = consumer.poll(Duration.ofMillis(1000));
				
			} catch (Exception e) {
				log.error("");
			} finally {
				consumer.close();
			}
		}
	}
}
```



**必要的参数配置**

bootstrap.servers 集群broker地址

group.id 消费者组名称

key.deserializer  反序列化

value.deserializer



参数众多，直接使用org.apache.kafka.clients.consumer.ConsumerConfig

每个参数在ConsumerConfig类中都有对应的名称

如ConsumerConfig.GROUP_ID_CONFIG



**订阅主题与分区**

一个消费者可以订阅一个或多个主题，subscribe的几个重载方法

```
public void subscribe(Collection<String> topics, ConsumerRebalanceListener listener);
public void subscribe(Collection<String> topics);
public void subscribe(Pattern pattern, ConsumerRebalanceListener listener);
public void subscribe(Pattern pattern);
```



1.集合方式，订阅了什么就消费什么主题的消息，如果前后订阅了不同的主题，那么消费者以最后一次的为准。

2.正则表达式，如果采用正则表达式的方式，在之后如果有人创建了新的主题，且主题名字与正则表达式匹配，那么这个消费者就可以消费到新添加的主题中的消息。



参数类型是ConsumerRebalanceListener，设置的是在均衡监听器

消费者还能直接订阅某些主题的特定分区

```
public void assign(Collection<TopicPartition> partitions);
```



TopicPartition类表示分区

```
public final class TopicPartition implements Serializable {
	private final int partition; //分区
	private final String topic; //主题
	...
}
```



如果事先不知道主题中有多少分区，则使用partitionsFor()方法查询指定主题的元数据信息

```
public List<PartitionInfo> partitionsFor(String topic)

public class PartitionInfo {
	private final String topic;
	private final int paitition;
	private final Node leader;
	private final Node[] replicas;  //AR
	private final Node[] inSyncReplicas; //ISR
	private final Node offlineReplicas;  //OSR
}
```



取消订阅

```
consumer.unsubscribe()
```

如果没有订阅任何主题或分区，那么继续执行消费程序会报异常IllegalStateException



订阅状态

集合订阅  AUTO_TOPICS

正则表达式订阅 AUTO_PATTERN

指定分区订阅 USER_ASSIGNED



通过subscribe()方法订阅主题具有消费者自动在均衡的功能，在多个消费者的情况下，可以根据分区分配策略来自动分配各个消费者与分区的关系。









