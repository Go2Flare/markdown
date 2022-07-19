kafka面试题











# kafka面试题

## 1.通用题目



### 1. Kafka 是什么？主要应用场景有哪些？



**Kafka 是一个分布式流式处理平台。** 



流平台具有三个关键功能： 



消息队列：发布和订阅消息流，这个功能类似于消息队列，这也是 Kafka也被归类为消息队列的原因。

容错的持久方式存储记录消息流：Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险。 

流式处理平台：在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。 



Kafka 主要有两大应用场景： 



消息队列：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。 

数据处理：构建实时的流数据处理程序来转换或处理数据流



### 2. 和其他消息队列相比，Kafka 的优势在哪里？



极致的性能 ：基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。 

生态系统兼容性无可匹敌 ：Kafka 与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域。 

LinkedIn 最早开发Kafka 用于处理海量的日志。



### 3. Kafka 的优点和缺点？



**优点：**

可扩展。Kafka集群可以透明的扩展，增加新的服务器进集群。
高性能。Kafka性能远超过传统的ActiveMQ、RabbitMQ等，Kafka支持Batch操作。
容错性。Kafka每个Partition数据会复制到几台服务器，当某个Broker失效时，Zookeeper将通知生产者和消费者从而使用其他的Broker。

持久性。消息可被持久化到磁盘，保证不会丢失。


**缺点：**

重复消息。Kafka保证每条消息至少送达一次，虽然几率很小，但一条消息可能被送达多次。
消息乱序。Kafka某一个固定的Partition内部的消息是保证有序的，如果一个Topic有多个Partition，partition之间的消息送达不保证有序。
复杂性。Kafka需要Zookeeper的支持，Topic一般需要人工创建，部署和维护比一般MQ成本更高。



### 4. 什么是主题、分区、broker、服务器、日志段？



**Topic(主题)：**发布订阅的对象是主题（Topic）。生产者将消息发送到特定的主题，消费者通过订 

阅特定的 Topic(主题) 来消费消息。



**Broker(代理)：**可以看作是一个独立的 Kafka 实例。多个 KafkaBroker 组成一个 Kafka Cluster。 



**server(服务器)：**kafka 的服务器端由被称为 Broker 的服务进程构成，即一个 Kafka 集群由多个 Broker 组成，Broker 负责接收和处理客户端发送过来的请求，以及对消息进行持久化。虽然多个 Broker 进程能够运行在同一台机器上，但更常见的做法是将不同的 Broker 分散运行在不同的机器上，这样如果集群中某一台机器宕机，即使在它上面运行的所有 Broker 进程都挂掉了，其他机器上的 Broker 也依然能够对外提供服务。



**Partition(分区)：**Partition 属于 Topic 的一部分。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。同一个partition的leader副本和follwer副本也可以分布在不同的broker上。



**Log Segment****(日志段)：**在 Kafka 底层，一个日志又近一步细分成多个日志段，消息被追加写到当前最新的日志段中，当写满了一个日志段后，Kafka 会自动切分出一个新的日志段，并将老的日志段封存起来。Kafka 在后台还有定时任务会定期地检查老的日志段是否能够被删除，从而实现回收磁盘空间的目的。



### 5. kafka的三层消息架构是什么？



第一层是主题层，每个主题可以配置 M 个分区，而每个分区又可以配置 N 个副本。

第二层是分区层，每个分区的 N 个副本中只能有一个充当领导者角色，对外提供服务；其他 N-1 个副本是追随者副本，只是提供数据冗余之用。

第三层是消息层，分区中包含若干条消息，每条消息的位移从 0 开始，依次递增。

最后，客户端程序只能与分区的领导者副本进行交互。



### 6. zookeeper在kafka中的作用



**Broker 注册 ：**在 Zookeeper 上会有一个专门用来进行 Broker 服务器列表记录的节点。每个 Broker 在启动时，都会到 Zookeeper 上进行注册，即到 /brokers/ids 下创建属于自己的节点。每个 Broker 就会将自己的 IP 地址和端口等信息记录到该节点中去。



**Topic 注册 ：** 在 Kafka 中，同一个 Topic 的消息会被分成多个分区并将其分布在多个 Broker 上，这些分区信息及与 Broker 的对应关系也都是由 Zookeeper 在维护。



**负载均衡 ：**上面也说过了 Kafka 通过给特定 Topic 指定多个Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力。 对于同一个 Topic 的不同 Partition，Kafka会尽力将这些 Partition 分布到不同的 Broker 服务器上。当生产者产生消息后也会尽量投递到不同 Broker 的 Partition 里面。当 Consumer 消费的时候，Zookeeper 可以根据当前的 Partition 数量以及 Consumer 数量来实现动态负载均衡。



### 7. Kafka 如何保证消息的消费顺序？



1. 1 个 Topic 只对应一个 Partition。 会影响TPS。
2. 发送消息的时候指定 key/Partition。



### 8. Kafka 判断一个节点是否还活着有那两个条件？



1. 节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接。
2. 如果节点是个 follower,他必须能及时的同步 leader 的写操作，延时不能太久。



1. **Kafka 高效文件存储设计特点是什么**	



1. Kafka 把 topic 中一个 parition 大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。 通过索引信息可以快速定位 message 和确定 response 的最大大小。 
2. 通过 index 元数据全部映射到 memory，可以避免 segment file 的 IO 磁盘操作。 
3. 通过索引文件稀疏存储，可以大幅降低 index 文件元数据占用空间大小。



### 10. partition 的数据如何保存到硬盘？



topic 中的多个 partition 以文件夹的形式保存到 broker，每个分区序号从0 递增，且消息有序。 Partition 文件下有多个 segment（xxx.index，xxx.log）segment 文件里的大小和配置文件大小一致可以根据要求修改，默认为 1g。 如果大小大于 1g 时，会滚动一个新的 segment 并且以上一个 segment 最后一条消息的偏移量命名。

## 2.生产者



### 1. 什么是生产者



向主题发布消息的客户端应用程序称为生产者（Producer），生产者程序通常持续不断地向一个或多个主题发送消息。



### 2. 分区的策略有哪些？



**轮询策略**



也称 Round-robin 策略，即顺序分配。比如一个主题下有 3 个分区，那么第一条消息被发送到分区 0，第二条被发送到分区 1，第三条被发送到分区 2，以此类推。当生产第 4 条消息时又会重新开始，即将其分配到分区 0。那么你的生产者程序会按照轮询的方式在主题的所有分区间均匀地“码放”消息。



轮询策略有非常优秀的负载均衡表现，它总是能保证消息最大限度地被平均分配到所有分区上，故默认情况下它是最合理的分区策略，也是我们最常用的分区策略之一。



**随机策略**



本质上看随机策略也是力求将数据均匀地打散到各个分区，但从实际表现来看，它要逊于轮询策略，所以如果追求数据的均匀分布，还是使用轮询策略比较好。有可能造成一个broker很热。



**按消息键保序策略**



这种策略一般是按照业务进行划分的，保证同一类消息发送到同一分区下，维护了消息的有序性。



### 3. 压缩算法是什么？



**压缩：具体来说就是用 CPU 时间去换磁盘空间或网络 I/O 传输量**，希望以较小的 CPU 开销带来更少的磁盘占用或更少的网络 I/O 传输。



生产者程序中**配置 compression.type 参数**即表示启用指定类型的压缩算法。



**有两种例外情况就可能让 Broker 重新压缩消息：**



情况一：Broker 端指定了和 Producer 端不同的压缩算法。



情况二：Broker 端发生了消息格式转换。



### 4. 如何确保消息不会丢失？



1. 不要使用 producer.send(msg)，而要使用 producer.send(msg, callback)。记住，一定要使用带有回调通知的 send 方法。



1. 设置 acks = all。acks 是 Producer 的一个参数，代表了你对“已提交”消息的定义。如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。



1. 设置 retries 为一个较大的值。这里的 retries 同样是 Producer 的参数，对应前面提到的 Producer 自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。



1. 设置 unclean.leader.election.enable = false。这是 Broker 端的参数，它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。



1. 设置 replication.factor >= 3。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。



1. 设置 min.insync.replicas > 1。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。



1. 确保 replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。



1. 确保消息消费完成再提交。Consumer 端有个参数 enable.auto.commit，最好把它设置成 false，并采用手动提交位移的方式。就像前面说的，这对于单 Consumer 多线程处理的场景而言是至关重要的。



### 5. 如何保证消息的幂等性？



**可以设置idempotence属性为True。**



首先，它只能保证单分区上的幂等性，即一个幂等性 Producer 能够保证某个主题的一个分区上不出现重复消息，它无法实现多个分区的幂等性。



其次，它只能实现单会话上的幂等性，不能实现跨会话的幂等性。这里的会话，你可以理解为 Producer 进程的一次运行。当你重启了 Producer 进程之后，这种幂等性保证就丧失了。



### 6. 如何使用事务？



Kafka 自 0.11 版本开始也提供了对事务的支持，目前主要是在 read committed 隔离级别上做事情。它能保证多条消息原子性地写入到目标分区，同时也能保证 Consumer 只能看到事务成功提交的消息。



设置事务型 Producer 的方法也很简单，满足两个要求即可：

- 和幂等性 Producer 一样，开启 enable.idempotence = true。
- 设置 Producer 端参数 transctional. id。最好为其设置一个有意义的名字。



此外，你还需要在 Producer 代码中做一些调整，如这段代码所示：

```sql
producer.initTransactions();
try {
            producer.beginTransaction();
            producer.send(record1);
            producer.send(record2);
            producer.commitTransaction();
} catch (KafkaException e) {
            producer.abortTransaction();
}
```



和普通 Producer 代码相比，事务型 Producer 的显著特点是调用了一些事务 API，如 initTransaction、beginTransaction、commitTransaction 和 abortTransaction，它们分别对应事务的初始化、事务开始、事务提交以及事务终止。



这段代码能够保证 Record1 和 Record2 被当作一个事务统一提交到 Kafka，要么它们全部提交成功，要么全部写入失败。实际上即使写入失败，Kafka 也会把它们写入到底层的日志中，也就是说 Consumer 还是会看到这些消息。因此在 **Consumer 端**，读取事务型 Producer 发送的消息也是需要一些变更的。修改起来也很简单，设置 isolation.level 参数的值即可。当前这个参数有两个取值：



1. read_uncommitted：这是默认值，表明 Consumer 能够读取到 Kafka 写入的任何消息，不论事务型 Producer 提交事务还是终止事务，其写入的消息都可以读取。很显然，如果你用了事务型 Producer，那么对应的 Consumer 就不要使用这个值。



1. read_committed：表明 Consumer 只会读取事务型 Producer 成功提交事务写入的消息。当然了，它也能看到非事务型 Producer 写入的所有消息。

## 3.消费者



### 1. 什么是消费者



订阅这些主题消息的客户端应用程序就被称为消费者（Consumer）。和生产者类似，消费者也能够同时订阅多个主题的消息。



### 2. 什么是消费者组



**Consumer Group 是 Kafka 提供的可扩展且具有容错性的消费者机制。**



既然是一个组，那么组内必然可以有多个消费者或消费者实例（Consumer Instance），它们共享一个公共的 ID，这个 ID 被称为 Group ID。组内的所有消费者协调在一起来消费订阅主题（Subscribed Topics）的所有分区（Partition）。当然，每个分区只能由同一个消费者组内的一个 Consumer 实例来消费。



**三个特性：**



Consumer Group 下可以有一个或多个 Consumer 实例。这里的实例可以是一个单独的进程，也可以是同一进程下的线程。在实际场景中，使用进程更为常见一些。



Group ID 是一个字符串，在一个 Kafka 集群中，它标识唯一的一个 Consumer Group。



Consumer Group 下所有实例订阅的主题的单个分区，只能分配给组内的某个 Consumer 实例消费。这个分区当然也可以被其他的 Group 消费。



Kafka 仅仅使用 Consumer Group 这一种机制，却同时实现了传统消息引擎系统的两大模型：



如果所有实例都属于同一个 Group，那么它实现的就是消息队列模型；



如果所有实例分别属于不同的 Group，那么它实现的就是发布 / 订阅模型。



理想情况下，Consumer 实例的数量应该等于该 Group 订阅主题的分区总数。



### 3. 消费者位移是如何进行管理的



将 Consumer 的位移数据作为一条条普通的 Kafka 消息，提交到 __consumer_offsets 中。可以这么说，__consumer_offsets 的主要作用是保存 Kafka 消费者的位移信息。



位移主题的 Key 中应该保存 3 部分内容：**Group ID，主题名，分区号 。**



如果位移主题是 Kafka 自动创建的，**那么该主题的分区数是 50，副本数是 3**。

### 

### 4. 消费者位移提交的方式



**自动提交位移**



Consumer 端有个参数叫 enable.auto.commit，如果值是 true，则 Consumer 在后台默默地为你定期提交位移，提交间隔由一个专属的参数 auto.commit.interval.ms 来控制。自动提交位移有一个显著的优点，就是省事，你不用操心位移提交的事情，就能保证消息消费不会丢失。但这一点同时也是缺点。因为它太省事了，以至于丧失了很大的灵活性和可控性，你完全没法把控 Consumer 端的位移管理。



如果你选择的是自动提交位移，那么就可能存在一个问题：只要 Consumer 一直启动着，它就会无限期地向位移主题写入消息。



**手动提交位移**



事实上，很多与 Kafka 集成的大数据框架都是禁用自动提交位移的，如 Spark、Flink 等。这就引出了另一种位移提交方式：**手动提交位移**，即设置 enable.auto.commit = false。一旦设置了 false，作为 Consumer 应用开发的你就要承担起位移提交的责任。Kafka Consumer API 为你提供了位移提交的方法，如 consumer.commitSync 等。当调用这些方法时，Kafka 会向位移主题写入相应的消息。

### 

### 5. 如何删除位移主题的过期数据

### 

Kafka 使用**Compact 策略**来删除位移主题中的过期消息，避免该主题无限期膨胀。



![img](https://cdn.nlark.com/yuque/0/2022/jpeg/22228669/1644735481220-d5d737c2-5713-441e-8f9b-2f74e83964c7.jpeg)

图中位移为 0、2 和 3 的消息的 Key 都是 K1。Compact 之后，分区只需要保存位移为 3 的消息，因为它是最新发送的。



**Kafka 提供了专门的后台线程定期地巡检待 Compact 的主题，看看是否存在满足条件的可删除数据**。这个后台线程叫 Log Cleaner。很多实际生产环境中都出现过位移主题无限膨胀占用过多磁盘空间的问题，如果你的环境中也有这个问题，我建议你去检查一下 Log Cleaner 线程的状态，通常都是这个线程挂掉了导致的。



### 6. 消费者组消费分区有哪些策略



**Range(默认策略)**



Range是对每个Topic而言的（即一个Topic一个Topic分），首先对同一个Topic里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。然后用Partitions分区的个数除以消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽，那么前面几个消费者线程将会多消费一个分区。



**RoundRobin**



RoundRobinAssignor策略的原理是将消费组内所有消费者以及消费者所订阅的所有topic的partition按照字典序排序，然后通过轮询方式逐个将分区以此分配给每个消费者。



使用RoundRobin策略有两个前提条件必须满足：

1. 同一个消费者组里面的所有消费者的num.streams（消费者消费线程数）必须相等；
2. 每个消费者订阅的主题必须相同。



**StickyAssignor**



Kafka从0.11.x版本开始引入这种分配策略，它主要有两个目的：

1. 分区的分配要尽可能的均匀，分配给消费者者的主题分区数最多相差一个
2. 分区的分配尽可能的与上次分配的保持相同



### 7. rebalance是什么？



Rebalance 就是让一个 Consumer Group 下所有的 Consumer 实例就如何消费订阅主题的所有分区达成共识的过程。在 Rebalance 过程中，所有 Consumer 实例共同参与，在协调者组件的帮助下，完成订阅主题分区的分配。但是，在整个过程中，所有实例都不能消费任何消息，因此它对 Consumer 的 TPS 影响很大。

### 8. rebalance发生的时机



1. 组成员数量发生变化
2. 订阅主题数量发生变化
3. 订阅主题的分区数发生变化

### 9. rebalance的弊端是什么



1. Rebalance 过程对 Consumer Group 消费过程有极大的影响。Rebalance 过程也和这个类似，在 Rebalance 过程中，所有 Consumer 实例都会停止消费，等待 Rebalance 完成。



1. 目前 Rebalance 的设计是所有 Consumer 实例共同参与，全部重新分配所有分区。其实更高效的做法是尽量减少分配方案的变动。



1. Rebalance 实在是太慢了。

### 

### 10. 如何避免不必要的rebalance



**第一类非必要 Rebalance 是因为未能及时发送心跳，导致 Consumer 被“踢出”Group 而引发的**。



因此，你需要仔细地设置**session.timeout.ms 和 heartbeat.interval.ms**的值。我在这里给出一些推荐数值，你可以“无脑”地应用在你的生产环境中。



- 设置 session.timeout.ms = 6s。
- 设置 heartbeat.interval.ms = 2s。
- 要保证 Consumer 实例在被判定为“dead”之前，能够发送至少 3 轮的心跳请求，即 session.timeout.ms >= 3 * heartbeat.interval.ms。
- 

将 session.timeout.ms 设置成 6s 主要是为了让 Coordinator 能够更快地定位已经挂掉的 Consumer。毕竟，我们还是希望能尽快揪出那些“尸位素餐”的 Consumer，早日把它们踢出 Group。希望这份配置能够较好地帮助你规避第一类“不必要”的 Rebalance。



**第二类非必要 Rebalance 是 Consumer 消费时间过长导致的**。



此时，**max.poll.interval.ms**参数值的设置显得尤为关键。如果要避免非预期的 Rebalance，你最好将该参数值设置得大一点，比你的下游最大处理时间稍长一点。就拿 MongoDB 这个例子来说，如果写 MongoDB 的最长时间是 7 分钟，那么你可以将该参数设置为 8 分钟左右。



如果你按照上面的推荐数值恰当地设置了这几个参数，却发现还是出现了 Rebalance，那么我建议你去排查一下**Consumer 端的 GC 表现**，比如是否出现了频繁的 Full GC 导致的长时间停顿，从而引发了 Rebalance。

### 

### 11. 如何监控消费者组的消费进度



1. 使用 Kafka 自带的命令行工具 kafka-consumer-groups 脚本。



1. 使用 Kafka Java Consumer API 编程。



1. 使用 Kafka 自带的 JMX 监控指标。

## 4.内核



### 1. kafka的副本机制



所谓的副本机制（Replication），也可以称之为备份机制，通常是指分布式系统在多台网络互联的机器上保存有相同的数据拷贝。



**好处**



- **提供数据冗余**。即使系统部分组件失效，系统依然能够继续运转，因而增加了整体可用性以及数据持久性。
- **提供高伸缩性**。支持横向扩展，能够通过增加机器的方式来提升读性能，进而提高读操作吞吐量。
- **改善数据局部性**。允许将数据放入与用户地理位置相近的地方，从而降低系统延时。
- 

在实际生产环境中，每台 Broker 都可能保存有各个主题下不同分区的不同副本，因此，单个 Broker 上存有成百上千个副本的现象是非常正常的。

![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645081630889-683d2ee4-1865-426c-b39a-32dcef712535.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

### 

### 2. 追随者副本不对外提供服务的好处有哪些？



**方便实现“Read-your-writes”**



所谓 Read-your-writes，顾名思义就是，当你使用生产者 API 向 Kafka 成功写入消息后，马上使用消费者 API 去读取刚才生产的消息。



举个例子，比如你平时发微博时，你发完一条微博，肯定是希望能立即看到的，这就是典型的 Read-your-writes 场景。如果允许追随者副本对外提供服务，由于副本同步是异步的，因此有可能出现追随者副本还没有从领导者副本那里拉取到最新的消息，从而使得客户端看不到最新写入的消息。



**方便实现单调读（Monotonic Reads）**



什么是单调读呢？就是对于一个消费者用户而言，在多次消费消息时，它不会看到某条消息一会儿存在一会儿不存在。



如果允许追随者副本提供读服务，那么假设当前有 2 个追随者副本 F1 和 F2，它们异步地拉取领导者副本数据。倘若 F1 拉取了 Leader 的最新消息而 F2 还未及时拉取，那么，此时如果有一个消费者先从 F1 读取消息之后又从 F2 拉取消息，它可能会看到这样的现象：第一次消费时看到的最新消息在第二次消费时不见了，这就不是单调读一致性。但是，如果所有的读请求都是由 Leader 来处理，那么 Kafka 就很容易实现单调读一致性。

### 

### 3. 主从副本达到同步的标准是什么？



**Kafka 引入了 In-sync Replicas，也就是所谓的 ISR 副本集合。**ISR 中的副本都是与 Leader 同步的副本，相反，不在 ISR 中的追随者副本就被认为是与 Leader 不同步的。



ISR 不只是追随者副本集合，它必然包括 Leader 副本。甚至在某些情况下，ISR 只有 Leader 这一个副本。



![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645082989840-68796ed7-9984-41ee-85d6-b98046f74193.png)



是否在ISR集合中的标准就是 Broker 端参数 replica.lag.time.max.ms 参数值。



这个参数的含义是 Follower 副本能够落后 Leader 副本的最长时间间隔，当前默认值是 10 秒。这就是说，只要一个 Follower 副本落后 Leader 副本的时间不连续超过 10 秒，那么 Kafka 就认为该 Follower 副本与 Leader 是同步的，即使此时 Follower 副本中保存的消息明显少于 Leader 副本中的消息。



我们在前面说过，Follower 副本唯一的工作就是不断地从 Leader 副本拉取消息，然后写入到自己的提交日志中。如果这个同步过程的速度持续慢于 Leader 副本的消息写入速度，那么在 replica.lag.time.max.ms 时间后，此 Follower 副本就会被认为是与 Leader 副本不同步的，因此不能再放入 ISR 中。此时，Kafka 会自动收缩 ISR 集合，将该副本“踢出”ISR。



值得注意的是，倘若该副本后面慢慢地追上了 Leader 的进度，那么它是能够重新被加回 ISR 的。这也表明，ISR 是一个动态调整的集合，而非静态不变的。

### 

### 4. Unclean 领导者选举指的是什么？



**Kafka 把所有不在 ISR 中的存活副本都称为非同步副本。**



通常来说，非同步副本落后 Leader 太多，因此，如果选择这些副本作为新 Leader，就可能出现数据的丢失。毕竟，这些副本中保存的消息远远落后于老 Leader 中的消息。在 Kafka 中，选举这种副本的过程称为 Unclean 领导者选举。Broker 端参数 unclean.leader.election.enable 控制是否允许 Unclean 领导者选举。



**开启会提高可用性，但是可能会造成数据丢失。因此，不建议开启。**

### 

### 5. kafka的请求模型是什么？





Kafka 使用的是**Reactor 模式**。

![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645083986099-536b65bc-3d7b-4731-8c36-f65273d4190d.png)

**Reactor 模式的架构图**



![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645084139399-f097a07e-1771-4494-86b3-680e3b9ad376.png)

**kafka实现请求的流程**



Kafka 提供了 Broker 端参数 num.network.threads，用于调整该网络线程池的线程数。其**默认值是 3，表示每台 Broker 启动时会创建 3 个网络线程，专门处理客户端发送的请求**。(使用轮询的方式将请求发送到某一个网络线程中。)



**当网络线程拿到请求后，它不是自己处理，而是将请求放入到一个共享请求队列中。**



IO 线程池处中的线程才是执行请求逻辑的线程。Broker 端参数**num.io.threads**控制了这个线程池中的线程数。**目前该参数默认值是 8，表示每台 Broker 启动后自动创建 8 个 IO 线程处理请求**。你可以根据实际硬件条件设置此线程池的个数。



**Purgatory是 Kafka 中著名的“炼狱”组件。**是用来**缓存延时请求**（Delayed Request）的。**所谓延时请求，就是那些一时未满足条件不能立刻处理的请求**。



比如设置了 acks=all 的 PRODUCE 请求，一旦设置了 acks=all，那么该请求就必须等待 ISR 中所有副本都接收了消息后才能返回，此时处理该请求的 IO 线程就必须等待其他 Broker 的写入结果。当请求不能立刻处理时，它就会暂存在 Purgatory 中。稍后一旦满足了完成条件，IO 线程会继续处理该请求，并将 Response 放入对应网络线程的响应队列中。



**请求队列是所有网络线程共享的，而响应队列则是每个网络线程专属的**。

### 

### 6. 数据类请求与控制类请求分别是什么？



**数据类请求：**客户端发送的 PRODUCE 请求和 FETCH 请求等。



**控制类请求：**Kafka 社区把 PRODUCE 和 FETCH 这类请求称为数据类请求，把 LeaderAndIsr、StopReplica 这类请求称为控制类请求。



社区于 2.3 版本正式实现了数据类请求和控制类请求的分离。



社区完全拷贝了这张图中的一套组件，实现了两类请求的分离。也就是说，Kafka Broker 启动后，会在后台分别创建网络线程池和 IO 线程池，它们分别处理数据类请求和控制类请求。至于所用的 Socket 端口，自然是使用不同的端口了，你需要提供不同的**listeners 配置**，显式地指定哪套端口用于处理哪类请求。



### 7. 控制器是什么？

### 

**控制器组件（Controller），是 Apache Kafka 的核心组件。它的主要作用是在 Apache ZooKeeper 的帮助下管理和协调整个 Kafka 集群**。



集群中任意一台 Broker 都能充当控制器的角色，但是，在运行过程中，只能有一个 Broker 成为控制器，行使其管理和协调的职责。



实际上，Broker 在启动时，会尝试去 ZooKeeper 中创建 /controller 节点。Kafka 当前选举控制器的规则是：**第一个成功创建 /controller 节点的 Broker 会被指定为控制器**。



### 8. 控制器有哪些作用



**1.主题管理（创建、删除、增加分区）**



这里的主题管理，就是指控制器帮助我们完成对 Kafka 主题的创建、删除以及分区增加的操作。换句话说，当我们执行**kafka-topics 脚本**时，大部分的后台工作都是控制器来完成的。关于 kafka-topics 脚本，我会在专栏后面的内容中，详细介绍它的使用方法。



**2.分区重分配**



分区重分配主要是指，**kafka-reassign-partitions 脚本**（关于这个脚本，后面我也会介绍）提供的对已有主题分区进行细粒度的分配功能。这部分功能也是控制器实现的。



**3.Preferred 领导者选举**



Preferred 领导者选举主要是 Kafka 为了避免部分 Broker 负载过重而提供的一种换 Leader 的方案。在专栏后面说到工具的时候，我们再详谈 Preferred 领导者选举，这里你只需要了解这也是控制器的职责范围就可以了。



**4.集群成员管理（新增 Broker、Broker 主动关闭、Broker 宕机）**



这是控制器提供的第 4 类功能，包括自动检测新增 Broker、Broker 主动关闭及被动宕机。这种自动检测是依赖于前面提到的 Watch 功能和 ZooKeeper 临时节点组合实现的。



比如，控制器组件会利用**Watch 机制**检查 ZooKeeper 的 /brokers/ids 节点下的子节点数量变更。目前，当有新 Broker 启动后，它会在 /brokers 下创建专属的 znode 节点。一旦创建完毕，ZooKeeper 会通过 Watch 机制将消息通知推送给控制器，这样，控制器就能自动地感知到这个变化，进而开启后续的新增 Broker 作业。



侦测 Broker 存活性则是依赖于刚刚提到的另一个机制：**临时节点**。每个 Broker 启动后，会在 /brokers/ids 下创建一个临时 znode。当 Broker 宕机或主动关闭后，该 Broker 与 ZooKeeper 的会话结束，这个 znode 会被自动删除。同理，ZooKeeper 的 Watch 机制将这一变更推送给控制器，这样控制器就能知道有 Broker 关闭或宕机了，从而进行“善后”。



**5.数据服务**



控制器的最后一大类工作，就是向其他 Broker 提供数据服务。控制器上保存了最全的集群元数据信息，其他所有 Broker 会定期接收控制器发来的元数据更新请求，从而更新其内存中的缓存数据。



![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645086876347-afaa15de-2831-49ce-939d-ad0fa66d01a9.png)



值得注意的是，这些数据其实在 ZooKeeper 中也保存了一份。每当控制器初始化时，它都会从 ZooKeeper 上读取对应的元数据并填充到自己的缓存中。有了这些数据，控制器就能对外提供数据服务了。这里的对外主要是指对其他 Broker 而言，控制器通过向这些 Broker 发送请求的方式将这些数据同步到其他 Broker 上。

### 

### 9. 高水位的作用是什么



**高水位的作用主要有 2 个：**



1. 定义消息可见性，即用来标识分区下的哪些消息是可以被消费者消费的。



1. 帮助 Kafka 完成副本同步。



![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645159303949-f40348ad-e678-4ef8-a847-a3054c464acd.png)



**位移值等于高水位的消息也属于未提交消息。也就是说，高水位上的消息是不能被消费者消费的**。



图中还有一个日志末端位移的概念，即 Log End Offset，简写是 LEO。它表示副本写入下一条消息的位移值。



**同一个副本对象，其高水位值不会大于 LEO 值**。



**高水位和 LEO 是副本对象的两个重要属性**。Kafka 所有副本都有对应的高水位和 LEO 值，而不仅仅是 Leader 副本。只不过 Leader 副本比较特殊，Kafka 使用 Leader 副本的高水位来定义所在分区的高水位。换句话说，**分区的高水位就是其 Leader 副本的高水位**。

### 

### 10. kafka的副本同步机制是什么？





![img](https://cdn.nlark.com/yuque/0/2022/png/22228669/1645160179238-6925eb33-37ed-4906-a288-02f08d1e71f1.png)



在这张图中，我们可以看到，Broker 0 上保存了某分区的 Leader 副本和所有 Follower 副本的 LEO 值，而 Broker 1 上仅仅保存了该分区的某个 Follower 副本。Kafka 把 Broker 0 上保存的这些 Follower 副本又称为**远程副本**（Remote Replica）。Kafka 副本机制在运行过程中，会更新 Broker 1 上 Follower 副本的高水位和 LEO 值，同时也会更新 Broker 0 上 Leader 副本的高水位和 LEO 以及所有远程副本的 LEO，但它不会更新远程副本的高水位值，也就是我在图中标记为灰色的部分。



为什么要在 Broker 0 上保存这些远程副本呢？其实，它们的主要作用是，**帮助 Leader 副本确定其高水位，也就是分区高水位**。



![img](https://cdn.nlark.com/yuque/0/2022/jpeg/22228669/1645160204759-2cd2d467-4d24-4471-ae86-fb06bad14a91.jpeg)



**追随者副本与leader副本保持同步需要两个条件：**



1. 该远程 Follower 副本在 ISR 中。



1. 该远程 Follower 副本 LEO 值落后于 Leader 副本 LEO 值的时间，不超过 Broker 端参数 replica.lag.time.max.ms 的值。如果使用默认值的话，就是不超过 10 秒。



**Leader 副本**

处理生产者请求的逻辑如下：

1. 写入消息到本地磁盘。
2. 更新分区高水位值。
   i. 获取 Leader 副本所在 Broker 端保存的所有远程副本 LEO 值{LEO-1，LEO-2，……，LEO-n}。
   ii. 获取 Leader 副本高水位值：currentHW。
   iii. 更新 currentHW = min(currentHW, LEO-1，LEO-2，……，LEO-n)。



处理 Follower 副本拉取消息的逻辑如下：

1. 读取磁盘（或页缓存）中的消息数据。
2. 使用 Follower 副本发送请求中的位移值更新远程副本 LEO 值。
3. 更新分区高水位值（具体步骤与处理生产者请求的步骤相同）。



**Follower 副本**

从 Leader 拉取消息的处理逻辑如下：

1. 写入消息到本地磁盘。
2. 更新 LEO 值。
3. 更新高水位值。
   i. 获取 Leader 发送的高水位值：currentHW。
   ii. 获取步骤 2 中更新过的 LEO 值：currentLEO。
   iii. 更新高水位为 min(currentHW, currentLEO)。****