关于Kafka的一个形象的解释：
```
https://www.zhihu.com/question/53331259/answer/242678597
老板有个好消息要告诉大家，有两个办法:
1.到每个座位上挨个儿告诉每个人。什么？张三去上厕所了？那张三就只能错过好消息了！ 2.老板把消息写到黑板报上，谁想知道就来看一下，什么？张三请假了？没关系，我一周之后才擦掉，总会看见的！什么张三请假两周？那就算了，我反正只保留一周，不然其他好消息没地方写了redis用第一种办法，kafka用第二种办法，知道什么区别了吧
```

看了这个教程，讲的很好：  
[【大数据】Kafka全套教程 从源码到面试真题by海波](https://www.youtube.com/watch?v=JUq1N8NClcg)  
下面的各集的笔记：

01 Kafka分布式原理 集群
```
Java的分为栈（线程和run的控制），堆（内存存放）和方法区（类的模板信息）。
Java的演化RMI(RPC) -> EJB -> Spring。
```

02 Kafka分布式原理 系统
```
集群的演化，
1）负载均衡控制 
最初可以使用于负载均衡器，但是负载均衡服务器本身会达到性能上限。
2）注册中心
返回所有机器连接，只返回连接，用户来轮询服务器，将负载放在客户端，相关软件有zookeeper。
3）集群下的性能关系
多个服务器会影响机器性能，网络性能，要解耦分离为多个微服务。但是为了解决微服务之间的通信和单点性能问题，要加上一个消息总线来管理和分配信息。（kafka来了）
```

03 Kafka原理介绍
```
1）Apache Flume：高可靠海量日志采集传输框架
log数据一般通过socket传输给另外一个系统，可能需要flume来帮助传输，保证抵达，高可靠。source-channel-sink模式。
数据保存在内存，容易丢，不容易增加消费者，数据无法长时间保留。

数据的冗余保存策略，在不同的node进行备份，一个宕了下一个还能用。

重温hash的原理：
hashcode -> hash -> ? & 15 -> index 所以hash容量初始16，扩容必须是2倍数
分区的备份均摊算法就可以用这个算法。

额外阅读
对搭建日志系统而言，也没什么新花样，以下这些依样画葫芦就可以了：
1. 日志采集（Logstash/Flume，SDK wrapper）
2. 实时消费 (Storm, Flink, Spark Streaming)
3. 离线存储 （HDFS or Object Storage or NoSQL) +  离线分析（Presto，Hive，Hadoop）
4. 有一些场景可能需要搜索，可以加上ES或ELK
比如log->flume->kafka->hdfs(solr)
```

04 Kafka介绍&安装
```
消息分发的方式：
1.点对点模式，就是传统生产者消费者模式。
2.发布/订阅模式，一对多，数据生产后，所有订阅者都可以取。
最大的区别就是发布订阅模式要保存数据，不会因为一个客户取了就删除。

kafka是发布订阅模式，
扩展性：可以随意扩展，
可恢复性：有多个队列宕一个也可以，然后之前的队列恢复了继续提供服务
顺序保证：比较重要的特性，每个队列的顺序是一致的，不过不同分区同时获取不保证顺序（因为网速不一致）

kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个实例（server）称为broker。
无论kafka集群，还是consumer都依赖于zookeeper集群保存一些meta信息。

kafka的架构；
1）Producer：消息生产者，就是向kafka broker发消息的客户端。
2）Consumer：消息消费者，向kafka broker取消息的客户端
3）Topic：可以理解为一个队列。
4）Consumer Group（CG）：这个kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个partition只会把消息发给该CG中的一个consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要实现所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。为了增加消费能力，多个Consumer组成Consumer group，每个Consumer读取一些Partition，然后组合。CG里面的Consumer一般和partition的数量一样。
5）Broker：一台kafka服务器就是一个broker。一个集群由多个broker组成，一个broker可以容纳多个topic。
6）Partition:对了实现扩展性，一个非常大的topic可以分不到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给Consumer，不保证一个topic的整理（多个partition间）的顺序。此外这些partition还会备份到其他broker去。
所以某个业务的topic如果需要顺序，必须只能放到一个broker。
7）offset：kafka的存储文件都是按照offset。kafka来命名，用offset做名字的好处是方便查找。
8）Leader Follower: Consumer只会从Leader的partition读，follower的partition只用来作备份。
9）zookeeper注册消息：保存所有集群机器信息，以及消费者当前消费信息。

实例安装
可以选择多少个分区，多少个备份
```

05 Kafka命令行操作
```
操作和试验kafka
比如生成first0，first1的文件，会根据设置存放到不同的集群机器中去

一个kafka的分区文件类似：
00000000000000000000.index ->索引，快速定位内容位置
00000000000000000000.log ->这个是实际文件
00000000000000000000.timeing

一些kafka配置的解释：
broker.id=0 这个broker(server)的编号
delete.topic.enable=true 是否删除物理topic
log.dirs=/opt/module/kafka/logs 存储的位置
nums.partitions=2 多少个分区
zookeeper.connect=hadoop102:2181,hadoop103:2181 连接zookeeper集群，可以多个

部分指令示例：
#启动
[hadoop102]$ bin/kafka-server-start.sh config/server.properties &
[hadoop103]$ bin/kafka-server-start.sh config/server.properties &
[hadoop104]$ bin/kafka-server-start.sh config/server.properties &
#关闭
[hadoop102]$ bin/kafka-server-stop.sh stop
[hadoop103]$ bin/kafka-server-stop.sh stop
[hadoop104]$ bin/kafka-server-stop.sh stop
#查看topic
[hadoop102]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --list
#创建topic #partitions分区个数 #replication-factor副本数（包括leader）
[hadoop102]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 \
--create --topic first --partitions 1 --replication-factor 3
#删除topic
[hadoop102]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 \
--delete --topic first
#发送消息
[hadoop102]$ bin/kafka-console-producer.sh \
--broker-list hadoop102:9092 --topic first
>hello world
>hahahaha
#消费消息 #from-begining从头读取，默认是从上次接着读
[hadoop102]$ bin/kafka-console-consumer.sh \
--zookeeper hadoop102:2181 --from-begining --topic first
#查看摸个topic的详情
[hadoop102]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 \
--describe --topic first
```

06 Kafka写入流程1
```
系统读写的原理
kafka的高性能原理
1）高吞吐量
2）顺写日志
3）0拷贝技术：从系统的缓存（pageCache）直接拷贝
4）分段日志：将大文件（比如大于1G）拆分称为多个文件，可以从偏移量后后面的分段开始读，快速定位从而加快读取
5）预读（Read ahead）：就是缓存了相关数据，比如读取2，顺便把相临的1，3也读取了
6）后写（Write Behind）：kafka直接写入操作系统Cache，由操作系统决定什么时间写入File。而Java程序是写入本身内存buffer，然后写入File。Kafka和Java程序是应用程序，而kafka让系统cache来写，是OS层，同一层级内的写是很快的。
```

07 Kafka写入流程2
```
int i=0;
i =i++;
System.out.println(i); --就是0
因为有临时变量，其实有临时变量，过程是
_a = i++;
_a = 0; i = 1;
i = _a = 0;
idea编译器反编译的技巧可以看过程。

Kafka是高读高写，Producer只和leader交互，然后replica（followers）的分区定时从leader拷贝数据。

kafka生成数据三种应答机制（ACK）：
1）取值为0：生产者发送完数据，不关心数据是否到达kafka，然后，直接发送下一条数据，这样的效率非常高，告示数据丢失的可能性非常大。
2）取值为1：生产者发送数据，需要等待Leader的应答，如果应答完成，才能发送下一条数据，不关系follower是否接受成功，这种场合，性能会慢一些，但是数据比较安全。但是leader保存数据成功后，突然down掉，follower没来得及获取数据，那么数据就会丢失。
3）取值为-1（all）：生产者发送数据，需要等待所有副本（leader+follower）的应答，这种方式数据最安全，但是性能非常差。
Kafka默认使用的是1的模式
```

08 Kafka工作流程 数据写入&数据存储
```
kafka生产数据的过程：
produer有多个P，创建后放入Deque，然后通过sender发送到kafka集群，kafka集群通过zookeeper管理

kafka数据保存：
文件是顺序存储的，所以00000000000000000000.log存放的就是11111222223333之类的数据，然后在所以00000000000000000000.index里面过程类似KV的格式，比如1 0, 2 59来控制偏移量

在follower（replica）还没来得及从leader取数据，leader就挂了怎么办？
HW: High WaterMark 木桶理论中最矮的那段
LEO: Log End offset 是leader的那段
用户只能看到HW的，follower会从leader取，只更新LEO部分，如果这个时候leader挂掉了，follower换上，HW还是3，所以对于用户来说没有区别。只有全部follower更新后，leader的LEO就更新的，所以就可以更新HW了。
```

09 Kafka工作流程 数据消费&消费者组
```
kafka消费数据：
消费组CG在消费的时候需要连接zookeeper，因为断点续传的信息保存在zookeeper中。在新版本0.9版本后就不需要了，信息会保存在集群里。例子：
#生产
[kafka]$ bin/kafka-console-producer.sh --broker-list linux1:9092 --topic first
#消费
[consumer]$ bin/kafka-console-consumer.sh --bootstrap-server linux1:9092 --topic first 
#这样就会创建一堆consumer_offset文件用来记录消费的偏移量，就不需要zookeeper了

创建topic的说明：
比如3台机器，如果创建4个partition没有问题，但是创建4个replica会报错，因为如果不是1台机器1个replica没有任何意义。

一个消费者可以消费多个kafka分区，但是一个分区同一时间只能被一个消费者消费。如果消费者数量多于kafka分区，多余的消费者没有用。但是如果增加分区数据，kafka会自动平衡（由leader分配）

存储策略：
kafka会保留所有消息，可以根据1）基于时间删除,2）基于大小删除，注意kafka读取数据复杂度是O(1)，减少数据不提升性能。

其中消费也有高级API和低级API，高级的都自动处理，低级需要自己管理偏移量和读取分区。消费者采用pull（拉）的方式来从分区读取，这样的好处是可以控制速率，来决定拉多少数据。

演示消费者组：
#生产
[kafka]$ bin/kafka-console-producer.sh --broker-list linux1:9092 --topic first
#单独的消费者组，每个命令启动一个消费者
[consumer]$ bin/kafka-console-consumer.sh --zookeeper linux1:2181 --topic first --from-beginning
#需要修改properties，控制消费者组
group.id=test-consuemr-group
#用消费者组的方式，两台consumer都运行下列指令
[consumer]$ bin/kafka-console-consumer.sh --zookeeper linux1:2181 --topic first --from-beginning --consumer.config config/consumer.properties
#用这种消费者组的方式，两台消费者收到的是分组后的信息
```

10 Kafka JavaAPI生产数据
```
Java用maven增加kafka依赖就能连接

public class TestProducer {
	public static void main(String[] args){
		//创建配置对象
		Properties prop = new Properties();
		//kafka集群
		prop.setProperty("bootstrap.servers", "linux:9092");
		//K,V序列化对象
		prop.setProperty("key.serializaer", "org.apache.kafka.common.serialization.StringSerializer");
		prop.setProperty("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		prop.setProperty("acks", "1");
		//创建生产者
		Producer<String, String> producer = new KafkaProcuer<String, String>(props);
		//准备数据
		String topic = "first";
		String value = "Hello Kafka";
		ProducerRecord record = new ProducerRecord(topic, value);
		//生产（发送）数据
		producer.send(record);
		//关闭生产者
		producer.close();
	}
}
```

11 Kafka Java生产者源码
```
kafka的是异步，这里提到了Java的future特性以及一个多线程写法

public static void main(String[] args) throws Exception {
	FutureTask<Integer> ft1 = new FutureTask<Integer>(new MyCallable10());
	FutureTask<Integer> ft2 = new FutureTask<Integer>(new MyCallable100());
	FutureTask<Integer> ft3 = new FutureTask<Integer>(new MyCallable1000());
	
	Thread t1 = new Thread(ft1);
	Thread t2 = new Thread(ft2);
	Thread t3 = new Thread(ft3);
	
	t1.start();
	t2.start();
	t3.start();
	
	System.out.println("main sm = " + (ft1.get() + ft2.get() + ft3.get());
}

get会阻塞当前的主线程，必须等待get结束

public class TestProducer {
	public static void main(String[] args) throws Exception{
		Properties prop = new Properties();
		prop.setProperty("bootstrap.servers", "linux:9092");
		prop.setProperty("key.serializaer", "org.apache.kafka.common.serialization.StringSerializer");
		prop.setProperty("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		prop.setProperty("acks", "1");	//应答机制
		//增加分区类，自己定义的分区类
		Prop.setProperty("partitioner.class", "com.xxx.bigdata.kafka.producer.MyPartitioner");
		Producer<String, String> producer = new KafkaProcuer<String, String>(props);
		String topic = "first";
		String value = "Hello Kafka";
		ProducerRecord record = new ProducerRecord(topic, value);
		//指定分区号
		//ProducerRecord record = new ProducerRecord(topic, 1, null, value);
		
		//同步发送数据(需要获取结果)
		producer.send(record).get();
		//异步发送
		producer.send(record);
		//异步，增加回调函数，可以看到存放在哪里了
		producer.send(record, new Callback()){
			//回调方法
			public void onCompletion(RecordMeadata metadata, Exception exception){
				//发送哪一个分区
				System.out.println(metadata.partition());
				//发送数据偏移量
				System.out.println(metadata.offset());
			}
		}
		//关闭生产者
		producer.close();
	}
}

/*分区算法类
 *就是写业务逻辑了，比如2020-10都存放分区1，2020-11都存放分区2，这样保证业务有序
 */
public class MayPartitioner implements Partitioner {
	public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster){
		//2018-10
		//crc32
		//hash & %
		String val = (String)value;
		int result = values.hashCode() ^ Integer.MAX_VALUE;
		return result % 4; 
	}
	public void close(){
	}
	public void configure(Map<String, ?> configs){
	}
}

看了些源码，帮助不大
``` 

12 Kafka方法重载&重写
```
介绍了些源码，帮助不大

讲个了个题
public static void main(String[] args) throws Exception {
	AA aa = new BB();
	System.out.println(aa.getResult()); //是20
}
class AA {
	public int i = 10;
	//注意这里的i只看在哪申明的，所以如果调用的AA这个class的，用的就是AA的变量10
	public int getResult(){
		return i + 10;
	}
}
class BB extends AA {
	public int i = 20;
	//public int getResult(){
	//	return i + 20;
	//}
}
另外一种情况
public static void main(String[] args) throws Exception {
	AA aa = new BB();
	System.out.println(aa.getResult()); //是30
}
class AA {
	public int i = 10;
	//这种情况重载调用的是BB的getI()，返回20，再+10，就返回了30
	public int getResult(){
		return getI() + 10;
	}
	public int getI(){
		return i;
	}	
}
class BB extends AA {
	public int i = 20;
	pubic int getI(){
		return i; //这个默认是this.i，所以返回就是20
	}
}
```

13 Kafka JavaAPI消费者
```
publi class TestConsumer {
	public static void main(String[] args){
		Properties prop = new Properties();
		prop.setProperty("bootstrap.servers", "linux:9092");
		prop.setProperty("key.deserializaer", "org.apache.kafka.common.Deserialization.StringSerializer");
		prop.setProperty("value.deserializer", "org.apache.kafka.common.Deserialization.StringSerializer");
		//指定consumer group
		prop.setProperty("group.id", "test");
		prop.setProperty("enable.auto.commit", "true");
		prop.setProperty("auto.commit.interval.ms", "1000");
	
		//创建消费者对象
		KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(prop);
		
		//订阅主题
		consumer.subscribe(Arrays.asList("first"));
		
		while ( true ) {
			//拉取数据
			ConsumerRecords<String, String> recors = consumer.poll(500); //500ms
			
			for(ConsumerRecord<String, String> record: records){
				//打印数据
				System.out.println(record);
			}
		}
	}
}
还有更底层的低级API来进行consume
public class LoweApiConsumer {
	public static void main(String[] args){
		BrokerEndPoint leader = null;
		//创建简单消费者
		String host = "linux1";
		int port = 9092;
		//获取分区的leader
		SimpleConsumer metaConsumer = new SimpleConsumer(host, port, 500, 10 * 1024, "metadata");
		//获取元数据消息
		TopicMetadataRequest request = new TopicMetadataRequest(Arrays.asList("first");
		TopicMetadataResponse response = metaConsumer.send(request);
		response.topicsMetadata();
		
		//这里的break用label的方式可以跳出多级循环
		leaderLabel:
		for(TopicMetadata topicMetadata : response.topicsMetadata()){
			if("first".equals(topicMetadata.topic())){
				//关心的主题元数据信息
				for(PartitionMetadata partitionMetadata : topicMetadata.partitionMetadata(){
					int partid = partitionMetadata.partitionId();
					if(partid == 1){
						//关心的分区原数据信息
						leader = partitionMetadata.leader();
						break leaderLabel;
					}
				}
			}
		}
		//host,port应该是指定分区的leader
		SimpleConsumer consumer = new SimpleConsumer(host, port, 500, 10*1024, "accessLeader");
		//抓取数据
		FetchRequest req = new FetchRequestBuilder().addFetch("first",1,5,10*1024).build();
		FetchResponse resp = consumer.fetch();
		
		ByteBufferMessagesSet  messageSet = resp.messageSet("first", 1);
		for(MessageAndOffset messageAndOffset : messageSet){
			ByteBuffer buffer = messageAndOffset.message().payload();
			byte[] bs = new byte[buffer.limit()];
			buffer.get(bs);
			String value = new String(bs, "UTF-8");
			System.out.println(value);
		}
	}
}
```

14 Kafka Java工作流程回顾 & 拦截器
```
提高消费能力：
1.加大kafka分区，这样消费者组也能增加，可以同时消费。
2.可以加大consumer.poll(100);的数量，就是一次poll更多数据，不过数据传输和网速也有关。
3.自己控制offset，通常offset由kafka自己控制，为了性能如果consumer端控制也可以提高kafka集群性能

拦截器(interceptor)
kafka0.10版本后引入，用于client端的定制化控制逻辑。对于producer而言，interceptor使得用户在消费发送前以及producer回调逻辑前有机会对消息做一些定制化需求，比如修改消息等。和web的filter有点像。

需求：实现一个简单的双interceptor组成的拦截链。第一个interceptor会在消息发送前将时间戳信息加到信息value的最前部；第二个interceptor会在消息发送后更新成功发送消息数或失败发送消息数。
//拦截器1
public class TimeInterceptor implements ProducerInterceptor<String, String> {
	public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record){
		String oldValue = record.value();
		String newValue = System.currentTimemillis() + "_" + oldValue;\
		return new ProducerRecord<String, String>(record.topic(), newValue);
	}
	public void onAcknowledgement(RecordMetadata metadata, Exception exception){
	}
	public void close(){
	}
	public void configure(Map<String, ?> configs){
	}
}
//拦截器2
public class CounterInterceptor implements ProducerInterceptor<String, String> {
	private int errCounter = 0;
	private int successCounter = 0;
	public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record){
		//此处返回原始数据，不能为空
		return record;
	}
	public void onAcknowledgement(RecordMetadata metadata, Exception exception){
		//统计成功和失败的次数
		if(exception == null){
			successCounter++;
		} else {
			errorCounter++;
		}
	}
	//销毁时候最终统计
	public void close(){
		System.out.println("success count = " + successCounter);
		System.out.println("error count = " + errorCounter);
	}
	public void configure(Map<String, ?> configs){
	}
}
//主程序
public class TestProducer {
	public static void main(String[] args) throws Exception{
		Properties prop = new Properties();
		prop.setProperty("bootstrap.servers", "linux:9092");
		prop.setProperty("key.serializaer", "org.apache.kafka.common.serialization.StringSerializer");
		prop.setProperty("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		prop.setProperty("acks", "1");	//应答机制		
		//增加分区类，自己定义的分区类
		Prop.setProperty("partitioner.class", "com.xxx.bigdata.kafka.producer.MyPartitioner");
		
		//配置拦截器
		List<String> interceptors = new ArrayList<String>();
		interceptors.add("com.xxx.bigdata.kafka.producer.interceptor.TimeInterceptor");
		interceptors.add("comt.xxx.bigdata.kafka.producer.interceptor.CounterInterceptor");
		prop.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);
		
		Producer<String, String> producer = new KafkaProcuer<String, String>(props);
		String topic = "first";
		String value = "Hello Kafka";
		ProducerRecord record = new ProducerRecord(topic, value);
		//指定分区号
		//ProducerRecord record = new ProducerRecord(topic, 1, null, value);
		
		//同步发送数据(需要获取结果)
		producer.send(record).get();
		//异步发送
		producer.send(record);
		//异步，增加回调函数，可以看到存放在哪里了
		for(int i= 1; i<=5; i++){
			producer.send(record, new Callback()){
				//回调方法
				public void onCompletion(RecordMeadata metadata, Exception exception){
					//发送哪一个分区
					System.out.println(metadata.partition());
					//发送数据偏移量
					System.out.println(metadata.offset());
					//会打印出5条成功的拦截器信息
				}
			}
		}
		//关闭生产者
		producer.close();
	}
}
```

15 Kafka流程处理
```
Kafka stream的流框架，不过有了Apache spark和Apark storm，所以kafka stream的用的不多。

需求：实时处理单词带有>>>前缀的内容，例如输出是"xxx>>>ximenqing",最终处理成"ximenqing"

kafka的处理就是将错误的原始数据放在一个topic，处理完又放到一个topic，所以用的不多。

Java String的底层知识：
Java类上有final说明这个类不能有子类，final一个数组，比如char[]，表示内存地址开辟后无法改变，但是其中的内容还是能变的。
比如String就是无法变的，但是通过Java的反射机制，是有办法绕过
public static void main(String[] args) throws Exception {
	String s = "abc";
	//s = s.trim() 通常要用这种方式
	Class clazz = s.getClass();
	Field f = clazz.getDeclaredField("value");
	f.setAccessible(true); //加了这个就能访问私有了，这在通常情况下是不允许的
	char[] cs = (char[])f.get(s); //char[] 是string的私有final属性
	cs[1] = '%';
	System.out.println(s); //显示了 "a%c"
}

```

16 Kafka和flume集成
```
线上数据 -> flume -> kafka -> flume（根据情景增删流程）-> HDFS

其他跳过

一些Broker配置参数的介绍：
比如控制message保存的时间；副本数据，建议2；等等

Producer也有单独配置：
比如压缩；自动提交（关闭预防宕机后的重复消费，因为offset会在开始存放在zookeeper）；

Consumer的设置：
比如增加poll来增加消费性能。
```

17 Kafka面试题
```
1.为什么kafka可以实现高吞吐？单节点kafka的吞吐量也比其他消息队列大，为什么?
集群所以高吞吐，单节点使用了一系列技术比如零拷贝（zero-copy），顺写日志，预读后写，分段日志，批处理，压缩来提高吞吐。

2.kafka的偏移量offset存放在哪儿，为什么？
早期存放在zookeeper，但是zookeeper主要用来做协调，如果集群大持续的存放offset，导致zookeeper压力过大不合适。所以0.9之后会推荐存放到kafka cluster里面去。当然也可以自定义逻辑，自己用另外系统操作offset存放。

3.Kafka里面用什么方式消费数据，拉的方式还是推的方式？
poll（是因为客户端速率问题），如果server发有可能客户端无法承受这么多数据，导致压力过大。

4.如何保证数据不会丢失或者重复消费的情况？做过哪些预防措施，怎么解决以上问题？
Producer同步发送数据；使用ACK=-1（all）牺牲性能保证传输成功；自己维护offset避免重复消费（低级API）

5.Kafka元数据存放在哪？
zookeeper（/controller, /cluster, /consumer, /broker）

6.为什么使用kafka，可不可以用flume直接将数据放在hdfs上
可以，但是flume只是传输框架，不能存储，所以会丢失数据。另外flume消费者处理比较麻烦。

7.kafka用的哪个版本？
2.11_0.11.0.2，2.11是scala版本，0.11是kafka版本

8.kafka如何保证不同的订阅源都收到相同的一份内容？
是通过HW，LEO，水桶理论，来保证客户端看到的数据是一样的

9.Kafka中leader的选举机制？
通过/controller来做的，通过zookeeper来管理的，哪个broker先再zookeeper上创建了/controller就是leader了，其他的broker会创建watch来监控/controller是否还在，不在就再次竞选。
还有ISR的概念

10.kafka的消费速度
增加分区和消费者，增加拉取数据的大小，增大批处理的大小

11.kafka分区
包括Leader，Follower和Replication，
分区导致的情况:用paritioner算法自定义，有序性用hash算法保证放在指定分区

12.kafka原理，isr中什么情况下brokerid会消失
ISR是正在同步的副本，
在副本宕掉，网络阻塞时候就消失，或者lag导致数据缺失过多（就是落后太久没有追上leader的日志）

13.为什么用kafka？kafka是如何存数据？
速度快，冗余，异步等等。
分段数据，offset，topics，partitions，index，log，replication等等概念

14.flume和kafka有序吗？
flume有序，kafka同一个分区有序，但是不保证不同分区的有序

15.kafka控制台向topic生产数据的命令及控制台消费topic数据的命令
等等

16.kafka消费用的高级API，低级API
KafkaConsumer.poll
SimpleConsumer.fetch, send

17.怎么手动维护offset
关闭自动提交，调用Consumer.commitSync();，将offset保存在redis，mysql之类的。
```


