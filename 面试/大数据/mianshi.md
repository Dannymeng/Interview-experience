# mianshi


大数据面试    张浩

**策略**

1.  抛砖引玉

2.  反客为主

（自己主动说，不要等着被面试官问，说完一段内容，可以停顿一下，然后接着说）

用A4纸写把项目的部分写下来

**项目介绍部分：**

1.  架构设计（画图）

2.  组件选择（调研+压测）

3.  集群规模

4.  高可靠的实现

5.  压缩格式

6.  文件格式

7.  每秒、每分钟数据量   离线、实时的数据量

8.  哪块高可靠没有做（ flume memory | spark yarn 调优）

9.  集群调优（硬件、Linux、JVM、CDH、HDFS）

开发内容：

1.  Spark、Hive

2.  存储

3.  监控（运维）

补充：

1.  Hive、Spark调优

2.  Bug怎么去解决

3.  算法

4.  机器学习

5.  仓库

6.  YARN怎么调优

7.  CDH资源池

**Java部分**

1.  GC、JVM垃圾选择器参数

2.  Java值传递与对象传递的区别

3.  Java的多继承、多态


4.  Java的sleep和wait的区别

5.  HashMap和HashTable的区别

6.  Java的多线程有哪几种形式

7.  Java接口和抽象类的区别
- 抽象类是捕捉子类的一般特性，抽象类是被用来创建继承层级里子类的模板；
- 接口是抽象方法的集合。如果一个类实现了某个接口，那么它就继承了这个接口的抽象方法。这就像契约模式，如果实现了这个接口，那么就必须确保使用这些方法。

| **参数** | **抽象类** | **接口** |
| :-----:| :--------: | :-----: |
| 默认的方法实现 | 它可以有默认的方法实现 | 接口完全是抽象的。它根本不存在方法的实现 |
| 实现 | 子类使用**extends**关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 | 子类使用关键字**implements**来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器 | 抽象类可以有构造器 | 接口不能有构造器 |
| 与正常Java类的区别 | 除了你不能实例化抽象类之外，它和普通Java类没有任何区别 | 接口是完全不同的类型 |
| 访问修饰符 | 抽象方法可以有**public**、**protected**和**default**这些修饰符 | 接口方法默认修饰符是**public**。你不可以使用其它修饰符。 |
| main方法 | 抽象方法可以有main方法并且我们可以运行它 | 接口没有main方法，因此我们不能运行它。 |
| 多继承 | 抽象方法可以继承一个类和实现多个接口 | 接口只可以继承一个或多个其它接口 |
| 速度 | 它比接口速度要快 | 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。 |
| 添加新方法 | 如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。 | 如果你往接口中添加方法，那么你必须改变实现该接口的类。 |
什么时候使用抽象类和接口：
*   如果你拥有一些方法并且想让它们中的一些有默认实现，那么使用抽象类吧。
*   如果你想实现多重继承，那么你必须使用接口。由于**Java不支持多继承**，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。
*   如果基本功能在不断改变，那么就需要使用抽象类。如果不断改变基本功能并且使用接口，那么就需要改变所有实现了该接口的类。

# 大数据部分

1.  HDFS读写流程

2.  YARN怎么调整Memory和CPU的资源

3.  MR作业和Spark作业在YARN的流程 

1.  Hive的分组排序

2.  Hive的自定义函数

3.  Hive的left join、left outer join和left semi join的主要区别

1.  有没有阅读过Spark的源码（一定回答阅读过，可以在github上看一下RDD的源码 ） [https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rdd/RDD.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rdd/RDD.scala)

## 2.  Spark RDD的五大特性
   - 1. A list of partitions
   RDD是一个由多个partition（某个节点里的某一片连续的数据）组成的的list；将数据加载为RDD时，一般会遵循数据的本地性（一般一个hdfs里的block会加载为一个partition）
   - 2. A function for computing each split
   RDD的每个partition上面都会有function，也就是函数应用，其作用是实现RDD之间partition的转换。
   - 3. A list of dependencies on other RDDs
    RDD会记录它的依赖 ，为了容错（重算，cache，checkpoint），也就是说在内存中的RDD操作时出错或丢失会进行重算。
  - 4. Optionally,a Partitioner for Key-value RDDs
  可选项，如果RDD里面存的数据是key-value形式，则可以传递一个自定义的Partitioner进行重新分区，例如这里自定义的Partitioner是基于key进行分区，那则会将不同RDD里面的相同key的数据放到同一个partition里面
  - 5. Optionally, a list of preferred locations to compute each split on
    **最优的位置**去计算，也就是**数据的本地性**。


## 3.  Spark读取Kafka的两种方式，两者区别
  *   Receiver-base：实时读取缓存到内存中
  Spark官方最先提供了基于Receiver的Kafka数据消费模式。不过这种方式是先把数据从kafka中读取出来，然后缓存在内存，再定时处理。如果这时候集群退出，而偏移量又没处理好的话，数据就丢掉了，存在程序失败丢失数据的可能，后在Spark 1.2时引入一个配置参数spark.streaming.receiver.writeAheadLog.enable以规避此风险。
  操作的类： KafkaUtils.createStream
  *   Direct：定时批量读取
  操作的类： KafkaUtils.createDirectStream
  Direct方式采用Kafka简单的consumer api方式来读取数据，无需经由ZooKeeper，此种方式不再需要专门Receiver来持续不断读取数据。当batch任务触发时，由Executor读取数据，并参与到其他Executor的数据计算过程中去。driver来决定读取多少offsets，并将offsets交由checkpoints来维护。将触发下次batch任务，再由Executor读取Kafka数据并计算。从此过程我们可以发现Direct方式无需Receiver读取数据，而是需要计算时再读取数据，所以Direct方式的数据消费对内存的要求不高，只需要考虑批量计算所需要的内存即可；另外batch任务堆积时，也不会影响数据堆积.
  
**Receive_base VS   Direct两种方式的优缺点：**

Direct方式具有以下方面的优势：

1、简化并行(Simplified Parallelism)。不现需要创建以及union多输入源，Kafka topic的partition与RDD的partition一一对应

2、高效(Efficiency)。Receiver-based保证数据零丢失(zero-data loss)需要配置spark.streaming.receiver.writeAheadLog.enable,此种方式需要保存两份数据，浪费存储空间也影响效率。而Direct方式则不存在这个问题。

3、强一致语义(Exactly-once semantics)。High-level数据由Spark Streaming消费，但是Offsets则是由Zookeeper保存。通过参数配置，可以实现at-least once消费，此种情况有重复消费数据的可能。

4、降低资源。Direct不需要Receivers，其申请的Executors全部参与到计算任务中；而Receiver-based则需要专门的Receivers来读取Kafka数据且不参与计算。因此相同的资源申请，Direct 能够支持更大的业务。

5、降低内存。Receiver-based的Receiver与其他Exectuor是异步的，并持续不断接收数据，对于小业务量的场景还好，如果遇到大业务量时，需要提高Receiver的内存，但是参与计算的Executor并无需那么多的内存。而Direct 因为没有Receiver，而是在计算时读取数据，然后直接计算，所以对内存的要求很低。实际应用中我们可以把原先的10G降至现在的2-4G左右。

6、鲁棒性更好。Receiver-based方法需要Receivers来异步持续不断的读取数据，因此遇到网络、存储负载等因素，导致实时任务出现堆积，但Receivers却还在持续读取数据，此种情况很容易导致计算崩溃。Direct 则没有这种顾虑，其Driver在触发batch 计算任务时，才会读取数据并计算。队列出现堆积并不会引起程序的失败。

Direct方式的缺点：

*   提高成本。Direct需要用户采用checkpoint或者第三方存储来维护offsets，而不像Receiver-based那样，通过ZooKeeper来维护Offsets，此提高了用户的开发成本。
*   监控可视化。Receiver-based方式指定topic指定consumer的消费情况均能通过ZooKeeper来监控，而Direct则没有这种便利，如果做到监控并可视化，则需要投入人力开发。

Receive-base优点：

1、Kafka的high-level数据读取方式让用户可以专注于所读数据，而不用关注或维护consumer的offsets，这减少用户的工作量以及代码量而且相对比较简单。

Receive-base的缺点：

1、防数据丢失。做checkpoint操作以及配置spark.streaming.receiver.writeAheadLog.enable参数，配置spark.streaming.receiver.writeAheadLog.enable参数，每次处理之前需要将该batch内的日志备份到checkpoint目录中，这降低了数据处理效率，反过来又加重了Receiver端的压力；另外由于数据备份机制，会受到负载影响，负载一高就会出现延迟的风险，导致应用崩溃。

2、单Receiver内存。由于receiver也是属于Executor的一部分，那么为了提高吞吐量，提高Receiver的内存。但是在每次batch计算中，参与计算的batch并不会使用到这么多的内存，导致资源严重浪费。

3、在程序失败恢复时，有可能出现数据部分落地，但是程序失败，未更新offsets的情况，这导致数据重复消费。

4、提高并行度，采用多个Receiver来保存的数据。Receiver读取数据是异步的，并不参与计算。如果开较高的并行度来平衡吞吐量很不划算。5、Receiver和计算的Executor的异步的，那么遇到网络等因素原因，导致计算出现延迟，计算队列一直在增加，而Receiver则在一直接收数据，这非常容易导致程序崩溃。

6、采用MEMORY_AND_DISK_SER降低对内存的要求。但是在一定程度上影响计算的速度  
![Kafka结合Streaming的两种方式]($resource/Kafka%E7%BB%93%E5%90%88Streaming%E7%9A%84%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F.png)

## 4.  Spark调优



5.  数据倾斜

6.  压缩格式

7.  文件格式

     **大数据项目部分**

1.  集群的规模

2.  每秒、每分钟、每小时、每天的吞吐量

3.  一个作业多大？多少资源？多久跑完

4.  如何设置作业的资源

5.  高可靠

6.  生产的优化

     **其他**

1.  IQ测试题，如果安排在会议室，没有摄像头，可百度搜索这些题

2.  进去公司的时候工资一定要要到位

3.  你还有什么问题吗？ 如果是技术问你，可以问团队构成、方向、技术、分析的东西；如果是HR问你，可以问晋升机制、新人培养问题

4.  你怎么看待加班？可以回答：“如果是工作需要，我会义无反顾的加班，同时我也会提高自己的工作效率，避免不必要的加班”
