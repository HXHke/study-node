#kafka学习笔记

什么是MQ

Message Queue（MQ），消息队列中间件。很多人都说：MQ 通过将消息的发送和接收分离来实现应用程序的异步和解偶，这个给人的直觉是——MQ 是异步的，用来解耦的，但是这个只是 MQ 的效果而不是目的。MQ 真正的目的是为了通讯，屏蔽底层复杂的通讯协议，定义了一套应用层的、更加简单的通讯协议。一个分布式系统中两个模块之间通讯要么是HTTP，要么是自己开发的（rpc） TCP，但是这两种协议其实都是原始的协议。HTTP 协议很难实现两端通讯——模块 A 可以调用 B，B 也可以主动调用 A，如果要做到这个两端都要背上WebServer，而且还不支持⻓连接（HTTP 2.0 的库根本找不到）。TCP 就更加原始了，粘包、心跳、私有的协议，想一想头皮就发麻。MQ 所要做的就是在这些协议之上构建一个简单的“协议”——生产者/消费者模型。MQ 带给我的“协议”不是具体的通讯协议，而是更高层次通讯模型。它定义了两个对象——发送数据的叫生产者；接收数据的叫消费者， 提供一个SDK 让我们可以定义自己的生产者和消费者实现消息通讯而无视底层通讯协议

面试题:为什么消息中间件不直接使用http协议呢?

1:因为http请求报文头和响应报文头是比较复业的，包含了cookie，数据的加密解密，状态码，响应码等附加的功能，但是对于一个消息而言，我们并不需要这么复杂，也没有这个必要性，它其实就是负责数据传递，存储，分发就行，一定要迫求的是高性能。尽量简洁，快速。

2:-大部分情况下http人部分都是短镒接，在实际的交互过程中，一个请求到响应很有可能会中断，中断以后就不会就行持久化，就会造成请求的丢失。这样就不利于消息中间件的业务场景，因为消息中间件可能是一个长期的获取消息的过程，出现问题和故障要对数据或消息就行持久化等，目的是为了保证消息和数据的高可靠和稳健的运行。

使用同步通信方式

![](C:\Users\13726\Desktop\studynode\学习图片\kafka1.jpg)

使用异步通信方式

![](C:\Users\13726\Desktop\studynode\学习图片\kafka2.jpg)

针对于同步的通信方式来说，异步的方式可以让上游快速成功，极大提高了系统的吞吐量。而且在分布式系统中，通过下游多个服务的分布式事务的保障，也能保障业务执行之后的最终一致性

消息队列解决的具体问题是什么--通信问题

###消息队列的流派

1. 有broker的MQ

	* 重topic：kafka、rocketMQ、activeMQ
	  整个broker，依据topic来进行消息的中转。在重topic的消息队列里必须需要topic的存在
	* 轻topic：rabbitMQ
	  topic只是一种中转模式

2. 无broker的MQ

	* 无 Broker 的 MQ 的代表是 ZeroMQ。该作者非常睿智，他非常敏锐的意识到——MQ 是更高级的 Socket，它是解决通讯问题的。所以 ZeroMQ 被设计成了一个“库”而不是一个中间件，这种实现也可以达到——没有 Broker 的目的	
	* 节点之间通讯的消息都是发送到彼此的队列中，每个节点都既是生产者又是消费者。ZeroMQ做的事情就是封装出一套类似于 Socket 的 API 可以完成发送数据，读取数据
	* ZeroMQ 其实就是一个跨语言的、重量级的 Actor 模型邮箱库。你可以把自己的程序想象成一个 Actor，ZeroMQ 就是提供邮箱功能的库；ZeroMQ 可以实现同一台机器的 RPC 通讯也可以实现不同机器的 TCP、UDP 通讯，如果你需要一个强大的、灵活、野蛮的通讯能力，别犹豫 ZeroMQ


###Kafka介绍

Kafka是最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的 （replica），基于zookeeper协调的分布式消息系统，它的最大的特性就是可以实时的处理 大量数据以满足各种需求场景：比如基于hadoop的批处理系统、低延迟的实时系统、 Storm/Spark流式处理引擎，web/nginx日志、访问日志，消息服务等等，用scala语言编 写，Linkedin于 2010 年贡献给了Apache基金会并成为顶级开源 项目。

1. 使用场景

日志收集：一个公司可以用Kafka收集各种服务的log，通过kafka以统一接口服务的方式 开放给各种consumer，例如hadoop、Hbase、Solr等。 消息系统：解耦和生产者和消费者、缓存消息等。 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网⻚、 搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过 订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖 掘。 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产 各种操作的集中反馈，比如报警和报告。

2. 基本概念

kafka是一个分布式的，分区的消息(官方称之为commit log)服务。它提供一个消息系统应该 具备的功能，但是确有着独特的设计。可以这样来说，Kafka借鉴了JMS规范的思想，但是确 并 没有完全遵循JMS规范。

基础的消息(Message)相关术语：

| 名称 | 解释 |
|
| Broker	 |消息中间件处理节点一个Kafka节点就是一个broker，一个或者多个Broker可以组成一个Kafka集群 |
| Topic | 	Kafka根据topic对消息进行归类，发布到Kafka集群的每条消息都需要指定一个topic |
| Producer	| 消息⽣产者，向Broker发送消息的客户端
| Consumer | 	消息消费者，从Broker读取消息的客户端
|ConsumerGroup | 	每个Consumer属于⼀个特定的Consumer Group，⼀条消息可以被多个不同的Consumer Group消费，但是⼀个Consumer Group中只能有⼀个Consumer能够消费该消息
|Partition|	物理上的概念，⼀个topic可以分为多个partition，每个partition内部消息是有序的|

因此，从一个较高的层面上来看，producer通过网络发送消息到Kafka集群，然后consumer 来进行消费，如下图：

![](https://bright-boy.gitee.io/technical-notes/kafka/images/QQ%E6%88%AA%E5%9B%BE20220110112502.png)

服务端(brokers)和客户端(producer、consumer)之间通信通过 TCP协议 来完成。

###kafka使用

1. 安装 

	准备好jdk、zookeeper、和 kafka

2. 启动服务

	./kafka-server-start.sh -daemon../config/server.properties

	server.properties核心配置详解

	|Property |	Default |	Description
	|
	|broker.id	|0|	每个broker都可以⽤⼀个唯⼀的⾮负整数id进⾏标识；这个id可以作为broker的“名字”，你可以选择任意你喜欢的数字作为id，只要id是唯⼀的即可。
	|log.dirs	|/tmp/kafka-logs|	kafka存放数据的路径。这个路径并不是唯⼀的，可以是多个，路径之间只需要使⽤逗号分隔即可；每当创建新partition时，都会选择在包含最少partitions的路径下进⾏。
	|listeners	|PLAINTEXT://192.168.65.60:9092|	server接受客户端连接的端⼝，ip配置kafka本机ip即可
	|zookeeper.connect	|localhost:2181|	zooKeeper连接字符串的格式为：hostname:port，此处hostname和port分别是ZooKeeper集群中某个节点的host和port；zookeeper如果是集群，连接⽅式为hostname1:port1, hostname2:port2,hostname3:port3
	|log.retention.hours	|168|	每个⽇志⽂件删除之前保存的时间。默认数据保存时间对所有topic都⼀样。
	|num.partitions	|1|	创建topic的默认分区数
	|default.replication.factor	|1|	⾃动创建topic的默认副本数量，建议设置为⼤于等于2
	|min.insync.replicas|	1|	当producer设置acks为-1时，min.insync.replicas指定replicas的最⼩数⽬（必须确认每⼀个repica的写数据都是成功的），如果这个数⽬没有达到，producer发送消息会产⽣异常
	|delete.topic.enable	|false	|是否允许删除主题

3. 创建主题topic

	* topic是什么概念？topic可以实现消息的分类，不同消费者订阅不同的topic。

	![](https://bright-boy.gitee.io/technical-notes/kafka/images/QQ%E6%88%AA%E5%9B%BE20220110122844.png)

	执行以下命令创建名为“test”的topic，这个topic只有一个partition，并且备份因子也设置为1

		./kafka-topics.sh --create --topic test --bootstrap-server 192.168.18.133:9092


4. 发送消息

	kafka自带了一个producer命令客户端，可以从本地文件中读取内容，或者我们也可以以命令行中直接输入内容，并将这些内容以消息的形式发送到kafka集群中。在默认情况下，每一个行会被当做成一个独立的消息。使用kafka的发送消息的客户端，指定发送到的kafka服务器地址和topic

		./kafka-console-producer.sh --broker-list 172.16.253.38:9092 --topic test

5. 消费消息

	对于consumer，kafka同样也携带了一个命令行客户端，会将获取到内容在命令中进行输 出， 默认是消费最新的消息 。使用kafka的消费者消息的客户端，从指定kafka服务器的指定 topic中消费消息

	方式一：从最后一条消息的偏移量+1开始消费

		./kafka-console-consumer.sh --bootstrap-server 172.16.253.38:9092 --topic test

	方式二：从头开始消费

		./kafka-console-consumer.sh --bootstrap-server 172.16.253.38:9092 --from-beginning --topic test

注意点：

* 消息会被存储
* 消息是顺序存储
* 消息是由偏移量的
* 消费时可以指明偏移量进行消费

###kafak的关键细节

1. 消息的顺序存储
	
	消息的发送方会把消息发送到broker中，broker会存储消息，消息是按照发送的顺序进行存储。因此消费者在消费消息时可以指明主题中消息的偏移量。默认情况下，是从最后一个消息的下一个偏移量开始消费。

2. 单播消息的实现

	单播消息：一个消费组里 只会有一个消费者能消费到某一个topic中的消息。于是可以创建多个消费者，这些消费者在同一个消费组中。

		./kafka-console-consumer.sh --bootstrap-server 10.31.167.10:9092 --consumer-property group.id=testGroup --topic test

3. 多播消息的实现

	kafka实现多播，只需要让不同的消费者处于不同的消费组即可

		./kafka-console-consumer.sh --bootstrap-server 10.31.167.10:9092 --consumer-property group.id=testGroup1 --topic test

		./kafka-console-consumer.sh --bootstrap-server 10.31.167.10:9092 --consumer-property group.id=testGroup2 --topic test

4. 查看消费组及信息

		//查看当前主题下有哪些消费组
		./kafka-consumer-groups.sh --bootstrap-server 10.31.167.10:9092 --list
		//查看消费组中的具体信息：比如当前偏移量、最后一条消息的偏移量、堆积的消息数量
		./kafka-consumer-groups.sh --bootstrap-server 172.16.253.38:9092 --describe --group testGroup
	* Currennt-offset: 当前消费组的已消费偏移量
	* Log-end-offset: 主题对应分区消息的结束偏移量(HW)
	* Lag: 当前消费组未消费的消息数

###主题、分区的概念

1. 主题topic

	主题topic可以理解成是一个类别的名称

2. partition分区

	![](https://bright-boy.gitee.io/technical-notes/kafka/images/QQ%E6%88%AA%E5%9B%BE20220110125413.png)

	一个主题中的消息量是非常大的，因此可以通过分区的设置，来分布式存储这些消息。比如一个topic创建了 3 个分区。那么topic中的消息就会分别存放在这三个分区中。	

为一个主题创建多个分区

	./kafka-topics.sh --create --zookeeper localhost:2181 --partitions 2 --topic test1

可以通过这样的命令查看topic的分区信息

	./kafka-topics.sh --describe --zookeeper localhost:2181 --topic test1

分区的作用：

* 可以分布式存储
* 可以并行写

定期将自己消费分区的offset提交给kafka内部topic：__consumer_offsets，提交过去的 时候，key是consumerGroupId+topic+分区号，value就是当前offset的值，kafka会定 期清理topic里的消息，最后就保留最新的那条数据 因为__consumer_offsets可能会接收高并发的请求，kafka默认给其分配 50 个分区(可以 通过offsets.topic.num.partitions设置)，这样可以通过加机器的方式抗大并发。 通过如下公式可以选出consumer消费的offset要提交到__consumer_offsets的哪个分区 公式：hash(consumerGroupId) % __consumer_offsets主题的分区数

###kafka集群及副本概念

1. 搭建kafka集群，三个broker

	准备三个server.properties文件，每个文件的broker.id，listeners和log.dir需要修改

	启动三台kafka服务，搭建完可以在zk中的/brokers/ids查看是否启动成功

2. 副本的概念

	副本是对分区的备份。在集群中，不同的副本会被部署在不同的broker上。下面例子：创建 1个主题， 2 个分区、 3 个副本。
	
		./kafka-topics.sh --create --zookeeper 172.16.253.35:2181 --replication-factor 3 --partitions 2 --topic my-replicated-topic
	
	![](https://bright-boy.gitee.io/technical-notes/kafka/images/QQ%E6%88%AA%E5%9B%BE20220110133849.png)

	通过查看主题信息，其中的关键数据：
	
	* replicas：当前副本存在的broker节点
	*leader：副本里的概念
		* 每个partition都有一个broker作为leader。
		* 消息发送方要把消息发给哪个broker？就看副本的leader是在哪个broker上面。副本里的leader专⻔用来接收消息。
		* 接收到消息，其他follower通过poll的方式来同步数据。
	* follower：leader处理所有针对这个partition的读写请求，而follower被动复制leader，不提供读写（主要是为了保证多副本数据与消费的一致性），如果leader所在的broker挂掉，那么就会进行新leader的选举，至于怎么选，在之后的controller的概念中介绍。

	isr： 可以同步的broker节点和已同步的broker节点，存放在isr集合中。

3. broker、主题、分区、副本

	* kafka集群中由多个broker组成
	* 一个broker中存放一个topic的不同partition——副本

	![](https://bright-boy.gitee.io/technical-notes/kafka/images/QQ%E6%88%AA%E5%9B%BE20220110134554.png)

4. kafka集群消息的发送

		./kafka-console-producer.sh --broker-list 172.16.253.38:9092,172.16.253.38:9093,172.16.253.38:9094 --topic my-replicated-topic

5. kafka集群消息的消费

	./kafka-console-consumer.sh --bootstrap-server 172.16.253.38:9092,172.16.253.38:9093,172.16.253.38:9094 --from-beginning --topic my-replicated-topic

6. 关于分区消费组消费者的细节

	![](https://bright-boy.gitee.io/technical-notes/kafka/images/QQ%E6%88%AA%E5%9B%BE20220110134734.png)

	图中Kafka集群有两个broker，每个broker中有多个partition。一个partition只能被一个消费组里的某一个消费者消费，从而保证消费顺序。Kafka只在partition的范围内保证消息消费的局部顺序性，不能在同一个topic中的多个partition中保证总的消费顺序性。一个消费者可以消费多个partition。

	注：消费组中消费者的数量不能比一个topic中的partition数量多，否则多出来的消费者消费不到消息。

###kafka集群controller、rebalance和hw

1. controller

	Kafka集群中的broker在zk中创建临时序号节点，序号最小的节点（最先创建的节点）将作为集群的controller，负责管理整个集群中的所有分区和副本的状态：

	* 当某个分区的leader副本出现故障时，由控制器负责为该分区选举新的leader副本。
	* 当检测到某个分区的ISR集合发生变化时，由控制器负责通知所有broker更新其元数据信息。
	* 当使用kafka-topics.sh脚本为某个topic增加分区数量时，同样还是由控制器负责让新分区被其他节点感知到。

2. rebalance机制

	前提是：消费者没有指明分区消费。当消费组里消费者和分区的关系发生变化，那么就会触发rebalance机制。

	这个机制会重新调整消费者消费哪个分区。
	
	在触发rebalance机制之前，消费者消费哪个分区有三种策略：
	
	* range：通过公示来计算某个消费者消费哪个分区
	* 轮询：大家轮着消费
	* sticky：在触发了rebalance后，在消费者消费的原分区不变的基础上进行调整。

3. hw和leo

	HW俗称高水位，HighWatermark的缩写，取一个partition对应的ISR中最小的LEO(log-end-offset)作为HW，consumer最多只能消费到HW所在的位置。另外每个replica都有HW,leader和follower各自负责更新自己的HW的状态。对于leader新写入的消息，consumer不能立刻消费，leader会等待该消息被所有ISR中的replicas同步后更新HW，此时消息才能被consumer消费。这样就保证了如果leader所在的broker失效，该消息仍然可以从新选举的leader中获取。

###kafka线上问题优化

1. 如何防止消息丢失

	* 发送方： ack是 1 或者-1/all 可以防止消息丢失，如果要做到99.9999%，ack设成all，把min.insync.replicas配置成分区备份数
	* 消费方：把自动提交改为手动提交。

2. 如何防止消息的重复消费

	一条消息被消费者消费多次。如果为了消息的不重复消费，而把生产端的重试机制关闭、消费端的手动提交改成自动提交，这样反而会出现消息丢失，那么可以直接在防治消息丢失的手段上再加上消费消息时的幂等性保证，就能解决消息的重复消费问题。

	幂等性如何保证：

	* mysql 插入业务id作为主键，主键是唯一的，所以一次只能插入一条
	* 使用redis或zk的分布式锁（主流的方案）

3. 如何做到顺序消费rocketmq

	* 发送方：在发送时将ack不能设置 0 ，关闭重试，使用同步发送，等到发送成功再发送下一条。确保消息是顺序发送的。
	* 接收方：消息是发送到一个分区中，只能有一个消费组的消费者来接收消息。因此，kafka的顺序消费会牺牲掉性能。

4. 解决消息积压问题

	消息积压会导致很多问题，比如磁盘被打满、生产端发消息导致kafka性能过慢，就容易出现服务雪崩，就需要有相应的手段：

	* 方案一：在一个消费者中启动多个线程，让多个线程同时消费。——提升一个消费者的消费能力（增加分区增加消费者）。
	* 方案二：如果方案一还不够的话，这个时候可以启动多个消费者，多个消费者部署在不同的服务器上。其实多个消费者部署在同一服务器上也可以提高消费能力——充分利用服务器的cpu资源。
	* 方案三：让一个消费者去把收到的消息往另外一个topic上发，另一个topic设置多个分区和多个消费者 ，进行具体的业务消费。

5. 延迟队列

	延迟队列的应用场景：在订单创建成功后如果超过 30 分钟没有付款，则需要取消订单，此时可用延时队列来实现

	* 创建多个topic，每个topic表示延时的间隔

		* topic_5s: 延时5s执行的队列
		* topic_1m: 延时 1 分钟执行的队列
		* topic_30m: 延时 30 分钟执行的队列
		
	* 消息发送者发送消息到相应的topic，并带上消息的发送时间

	* 消费者订阅相应的topic，消费时轮询消费整个topic中的消息

	* 如果消息的发送时间，和消费的当前时间超过预设的值，比如 30 分钟
	* 如果消息的发送时间，和消费的当前时间没有超过预设的值，则不消费当前的offset及之后的offset的所有消息都消费
	* 下次继续消费该offset处的消息，判断时间是否已满足预设值

###kafka-eagle监控平台

对kafka系统设计的一个监控平台