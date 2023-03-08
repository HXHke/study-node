#redis

redis是基于内存操作，属于单线程

默认16个数据库，使用select 命令切换

	flushdb：清空当前数据库
	flushall：清空所有数据库

---

	exist key //返回1为存在key，为0则不存在
	move key dbname //转移数据库
	expire key time //设置过期时间，单位秒
	ttl key 查看过期时间

###string操作
	set key value
	get key 
	append key "value" 	末尾追加字符串，如果当前key不存在，则相当于新建一个string
	strlen key  获取字符串长度
	incr key  value加一
	decr key  value减一
	incrby key num  value加num
	decrby key num  value减num
	getrange key 0 3 截取字符串从0到3
	setrange key 2 xx 从字符串第二下标位置开始修改为xx
	setex （set with expire） 设置过期时间
	setnx （set if not exist）不存在则设置
	mset k1 v1 k2 v2 k3 v3
	mget k1 k2 k3

###list操作
	lpush k v1 lpush k v2 lpush k v3 将一个或多个值插入到队列头部
	lrange k 0 -1 获取list所有值
	lrange k 0 1 ("v3" "v2") <栈>
	rpush k v4 将一个或多个值插入到队列尾部
	lpop k 移除列表的头部
	rpop k 移除列表的尾部
	lindex k 1 获取列表索引为1的值
	llen k 获取队列的长度
	lrem k num v 移除列表中num个v值
	ltrim k 1 2 通过下标截取指定长度的列表 

###set操作
	sadd name "hk"  添加元素
	smembers name 获取集合所有元素
	sismember name key 判断是否存在元素key
	spop name 随机移除一个元素
	srandmember name num 随机返回num个随机元素
	srem name v 移除集合中的v
	smove name1 name2 v 移动元素v从name1到name2
	scard name 返回集合元素的个数
	sinter name1 name2 获取两个元素的交集
	sinterstore name_interset name1 name2 将两个集合的交集保存在name_interset中
	sunion name1 name2 获取两个元素的并集
	sunionstore name name1 name2 将两个集合的并集保存在name中
	sdiff name1 name2 返回name1中与name2的差集
	sdiff name name1 name2 将name1中与name2的差集保存至name中

###hash操作
map集合 k,v键值对

	hset myhash field value 将myhash哈希表里存入一个键值对
	hget myhash field	从myhash哈希表里获取值
	hmset myhash field1 v1 field2 v2 多行操作
	hmset myhash field1 field2
	hgetall myhash 获取哈希表里所有键值对
	hdel myhash field 删除hash指定的key
	hlen myhash 获取hash中键值对数
	hexists myhash field 判断hash中指定字段是否存在
	hkeys myhash 获取所有的k
	hvals myhash	 获取所有的v
	hincrby myhash field 1 增1
	hsetnx myhash field v 不存在则添加

###zset操作

	zadd key score v 为集合元素添加score值
	zrange key min max 返回min到max范围内的集合元素
	zrevrange key max min 从大到小返回集合元素
	zrangebyscore key min max [withscores]根据score从min到max返回集合元素
	zrem key v 移除元素
	zcard key 获取有序集合的元素个数
	zcount key start stop 获取指定区间的元素
	
###geospatial地理位置

	geoadd city 166.40 39,90 (纬度经度) 北京  添加地理位置
	geopos city beijing 获取城市的纬度和经度
	geodist city beijing shanghai m(单位)返回两地的距离 单位(m,km,mi,ft)
	georadius city 110 30 1000 km 以给定的经纬度为中心，返回周围1000km的city
	georadiusbymember city beijing 1000 km 以指定成员为中心，返回周围1000km的city
	geohash city beijing 返回成员经纬度转为hash字符串(如果两个字符串越相近，则地理位置越近)
geo底层的实现原理就是zset，我们可以使用zset去操作geo(zrange,zrem)

###hyperloglog
A{1,3,5,7,8,7}
B{1,3,5,7,8}
基数 = 5,可以接受误差，0.81%的容错率
网页的uv（一个人访问一个网站多次，但是还是算作一个人！）
优点：占用的内存时固定的，2^64个不同的元素技术，只需要12kb的内存

	pfadd key a b c d e f g...添加元素
	pfcount key 统计总数
	pfmerge key3 key1 key2 合并（重复的不计，求并）

###bitmap
统计用户信息。活跃不活跃，登陆未登录。打卡
bitmaps位图，数据结构，都是操作二进制位来进行记录，就只有0和1两个状态

	127.0.0.1:6379> setbit sign 0 1
	(integer) 0
	127.0.0.1:6379> setbit sign 1 0
	(integer) 0
	127.0.0.1:6379> setbit sign 2 0
	(integer) 0
	127.0.0.1:6379> setbit sign 3 1
	(integer) 0
	127.0.0.1:6379> setbit sign 4 1
	(integer) 0
	127.0.0.1:6379> setbit sign 5 0
	(integer) 0
	127.0.0.1:6379> setbit sign 6 0
	(integer) 0
	127.0.0.1:6379> getbit sign 3
	(integer) 1
	127.0.0.1:6379> getbit sign 6
	(integer) 0
	127.0.0.1:6379> bitcount sign
	(integer) 3

---
###redis事务
redis事务本质：一组命令的集合。一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行！一次性、顺序性、排他性，执行一系列的命令。

==redis命令没有隔离级别的概念==

所有的命令在事务中并没有直接被执行，只有发起执行命令的时候才会执行exec

reids单条命令是保证原子性的，但是事务不保证原子性

redis的事务：

* 开启事务（multi）
* 命令入队（...）
* 执行事务（exec）

		127.0.0.1:6379> multi
		OK
		127.0.0.1:6379(TX)> set k1 v1
		QUEUED
		127.0.0.1:6379(TX)> set k2 v2
		QUEUED
		127.0.0.1:6379(TX)> get k2
		QUEUED
		127.0.0.1:6379(TX)> set k3 v3
		QUEUED
		127.0.0.1:6379(TX)> exec
		1) OK
		2) OK
		3) "v2"
		4) OK

放弃事务

	127.0.0.1:6379> multi
	OK
	127.0.0.1:6379(TX)> set k1 v1
	QUEUED
	127.0.0.1:6379(TX)> discard
	OK

编译型异常（代码有问题，命令有错），事务中所有命令都不会被执行

运行时异常，如果事务队列中存在语法性错误时，那么执行命令的时候，其他命令是可以正常执行的，错误命令抛出异常

###监控
悲观锁：

* 什么时候都会出问题，无论做什么都会加锁

乐观锁：

* 认为什么时候都不会出问题，所以不会上锁，更新数据的时候去判断一下。在此期间是否有人修改过这个数据
* 获取version
* 更新的时候比较version

redis监视测试

	127.0.0.1:6379> set money 100
	OK
	127.0.0.1:6379> set out 0
	OK
	127.0.0.1:6379> watch money //监视money对象
	OK
	127.0.0.1:6379> multi 
	OK
	127.0.0.1:6379(TX)> decrby money 20
	QUEUED
	127.0.0.1:6379(TX)> incrby out 20
	QUEUED
	127.0.0.1:6379(TX)> exec
	1) (integer) 80
	2) (integer) 20
	//事务正常结束，数据期间没有发生任何改动，这个时候就正常执行成功

	
线程一：

	127.0.0.1:6379> set money 100
	OK
	127.0.0.1:6379> set out 0
	OK
	127.0.0.1:6379> watch money
	OK
	127.0.0.1:6379> multi
	OK
	127.0.0.1:6379(TX)> decrby money 20
	QUEUED
	127.0.0.1:6379(TX)> incrby out 20
	QUEUED
	127.0.0.1:6379(TX)> exec //执行之前另外一个线程修改了值
	(nil)

线程二：

	127.0.0.1:6379> get money
	"100"
	127.0.0.1:6379> set money 1000
	OK
	//在线程一事务中插入，事务执行失败

  
如果发现事务执行失败，就先解锁
获取最新的值，再次监视，select version
比对监视的值是否发生了变化

###redis持久化
* pdb

在指定的时间间隔内将内存中的数据集快照写入次磁盘，也就是行话讲的snapshot快照，他恢复时是将快照文件直接读到内存里。

redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何io操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那rdb方式要比aof方式更加的高效。rdb的缺点是最后一次持久化后的数据可能丢失。我们默认的就是rdb，一般情况下不需要修改这个配置。

![](https://img-blog.csdnimg.cn/92195d2bb92c4c20bc447e8cf83416aa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aOr5aSa56Kn6I6J,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/img_convert/c30b1fb95dcb47c0639ab05aaf62f2c6.png)

rdb保存的文件时dump.rdb

触发rdb的条件

1. save的规则满足的情况下，会自动触发rdb规则
2. 执行flushall命令，也会触发我们的rdb规则
3. 退出redis，也会产生rdb文件

如何恢复rdb文件

1. 只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查dump.rdb恢复其中的数据
2. 查看需要存在的位置 （config get dir）

优点：
1. 适合大规模的数据恢复
2. 对数据的完整性要求不高

缺点：
1. 需要一定的时间间隔进程操作！如果redis宕机了，这个最后一次修改数据就没有了
2. fork进程的时候，会占用一定的内存空间

* aof

将我们所有的命令都记录下来，history，恢复的时候就把这个文件全部再执行一遍

以日志的形式来记录每个写操作，将redis执行过的每个指令记录下来（读操作不记录），值许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

默认是不开启的，我们需要手动进行配置，我们只需要将appendonly改为yes就开启了

重写规则：如果aof文件大于64m，就会fork一个新的进程来将我们的文件进行重写

优点：
1. 每一次修改都同步，文件的完整会更加好
2. 每秒同步一次，可能会丢失一秒的数据
3. 从不同步效率最高

缺点：
1. 相对于数据文件来说，aof远远大于rdb，修改的速度也比rdb慢
2. aof的运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化

###redis发布订阅
redis发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息
redis客户端可以订阅任意数量的频道
![](http://https://img-blog.csdnimg.cn/a067ccf7acf4460ba61a8100e3ca7646.png)

订阅端：

	127.0.0.1:6379> SUBSCRIBE hk
	Reading messages... (press Ctrl-C to quit)
	1) "subscribe"
	2) "hk"
	3) (integer) 1

发送端：

	127.0.0.1:6379> PUBLISH hk "hello,hk"
	(integer) 1

订阅端：

	1) "message"
	2) "hk"
	3) "hello,hk"

Redis的发布与订阅功能可以让客户端通过广播方式，将 消息（message）同时发送给可能存在的多个客户端，并且发送消息的客户端不需要知道接收消息的客户端的具体信息。换句话说，发布消息的客户端与接收消息的客户端两者之间没有直接联系。

在Redis中，客户端可以通过订阅特定的 频道（channel）来接收发送至该频道的消息，我们把这些订阅频道的客户端称为订阅者（subscriber）。一个频道可以有任意多个订阅者，而一个订阅者也可以同时订阅任意多个频道。除此之外，客户端还可以通过向频道发送消息的方式，将消息发送给频道的所有订阅者，我们把这些发送消息的客户端称为发送者（publisher）

* 发布订阅的实现场景

	1. 实时沟通消息系统 
	2. 微信公众号（点击关注，后台发送一篇博客，订阅的用户就可以监听到）
	3. 电商中，用户下单成功之后向指定频道发送消息，下游业务订阅支付结果这个频道处理自己相关业务逻辑
	4. 粉丝关注功能、文章推送
	5. 
	…

还有一些比较复杂的场景，可以使用消息中间件来做，kafka RabbitMQ ActiveMQ RocketMQ …等

###redis主从复制

主从复制，读写分离。80%的情况下都是在进行读操作，减缓服务器压力，架构中经常使用

* 环境配置

只配置从库，不配置主库,我们一般情况下只用配置从机就好了

	127.0.0.1:6379> info replication //查看当前库的信息
	# Replication
	role:master //角色
	connected_slaves:0 //没有从机
	master_failover_state:no-failover
	master_replid:d2fab7e8e7ecddf2198dd65c17db075acacdf162
	master_replid2:0000000000000000000000000000000000000000
	master_repl_offset:0
	second_repl_offset:-1
	repl_backlog_active:0
	repl_backlog_size:1048576
	repl_backlog_first_byte_offset:0
	repl_backlog_histlen:0

slaveof 127.0.0.1 6379 //slaveof host 6370 找主机

真实的主从配置应该在配置文件中配置，这样的话是永久的，使用命令的是暂时的

配置文件

	replicaof <masterip> <masterport>
	masterauth <master-password>

主机可以写，从机不能写只能读。主机中的所有的信息和数据都会自动被从机保存。主机断开连接，从机依旧连接到主机的，但是没有写操作，这个时候，主机回来了，从机依旧可以获取到主机写的信息

如果是使用命令行配置的主从，这个时候从机重启了就会变成主机。只要变成从机，就会立刻从主机中获取值

如果主机断开了连接，我们可以使用slaveof no one 让自己变成主机

###哨兵模式

![](https://img-blog.csdnimg.cn/img_convert/88d68b7e93e4fee57abc8574d62f3db1.png)

由两部分组成，哨兵节点和数据节点：

哨兵节点：监哨兵系统由一个或者多个哨兵节点组成，哨兵节点是特殊的Redis节点，不存储数据
数据节点：主节点master和从节点slave都是数据节点
关于哨兵的工作原理，主要是理解定时任务、主观下线和客观下线的几个概念

1）定时任务： 每个哨兵节点维护了3个定时任务，分别是向主从发送info命
令获取最新的主从结构、通过发布订阅获取其它哨兵节点信息、通过向其它节点发送ping命令进行心跳检测

2）主观下线： 在心跳检测中，如果其它节点超过一定时间未回复，哨兵节点会将其进行主观下线，即一个哨兵节点主观的判断下线

3）客观下线： 哨兵节点对主节点进行主观下线之后，会通过sentinel is-master-down-by-addr命令向其它哨兵节点询问该主节点的状态，如果判断该主节点主观下线的哨兵数量达到一定的数值，则对该主节点进行客观下线

配置哨兵

1. 配置文件sentinel.conf

		sentinel monitor redis79 127.0.0.1 6379 1

	后面的1代表主机挂了，slave投票让谁来接替主机，票数最多的就会成为主机

2. 启动哨兵

		redis-sentinel sentinel.conf

如果主机宕机了，这个时候就会从从机中随机选择一个来做主机（投票算法）
如果主机此时回来了，只能归并到新的主机下，当做从机。

优点：

1. 哨兵集群，基于主从复制模式，所有的主从配置优点，他都有
2. 主从可以切换，故障可以转移，系统的可用性就会更好
3. 哨兵模式就是主从模式的升级，手动到自动，更加健壮

缺点：

1. redis不好在线扩容，集群容量一旦到达上限，在线扩容就十分麻烦
2. 实现哨兵模式的配置其实是很麻烦的，里面有很多的选择

哨兵模式的全部配置

	#sentinel端口
	port 26379
	#工作路径，注意路径不要和主重复
	dir "/usr/local/redis-6379"
	# 守护进程模式
	daemonize yes
	#关闭保护模式
	protected-mode no
	# 指明日志文件名
	logfile "./sentinel.log"
	#哨兵监控的master，主从配置一样，这里只用输入redis主节点的ip/port和法定人数。
	sentinel monitor mymaster 192.168.125.128 6379 1
	# master或slave多长时间（默认30秒）不能使用后标记为s_down状态。
	sentinel down-after-milliseconds mymaster 5000
	#若sentinel在该配置值内未能完成failover操作（即故障时master/slave自动切换），则认为本次failover失败。
	sentinel failover-timeout mymaster 18000
	#设置master和slaves验证密码
	sentinel auth-pass mymaster 123456
	sentinel parallel-syncs mymaster 1//指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步

###redis缓存穿透和雪崩

* 缓存穿透

概念：用户想要查询一个数据，发现redis内存数据库中没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透

解决方案：

	1. 布隆过滤器
	2. 缓存空对象

* 缓存击穿

概念：这里需要注意和缓存穿透的区别，缓存击穿是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库。

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写数据，回导致数据库瞬间压力过大

解决方案：

	1. 设置热点数据永不过期
	2. 加互斥锁（在缓存到数据库之间加一个互斥锁）

* 缓存雪崩

概念：是指在某一个时间段，缓存集中过期失效。redis宕机，然后对数据的访问全部落在数据库上。对于数据库而言就会产生周期性的压力波峰。或者缓存服务器某个节点宕机或断电，对数据库服务器的压力是不可预知的，很可能瞬间就把数据库压垮

解决方案：

	1. redis高可用：多增设几台redis，一台挂掉之后其他的还可以继续工作，其实就是搭建集群
	2. 限流降级：在缓存失效后，通过加锁或者队列来控制数据库写缓存的线程数量。比如某个key只允许一个线程查询数据和写缓存，其他线程等待
	3. 数据预热：在正式部署之前，先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中，在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀