# Spark



## 1.spark优化?
架构参数优化：shuffle，内存管理，推测执行，数据本地化：HDFS的DataNode和Spark Worker共享一台机器
代码层面：并行度--调整finalRDD partition；缓存机制的选择--CPU使用和内存使用的权衡：checkpoint；算子的使用和选择-groupbykey，map vs mappartitions等，使用广播变量，累加器等； 序列化：压缩，存储格式的选择
数据倾斜：重写partition规则，抽样看数据的分布，结合具体的业务
架构的选择：统一使用yarn结合hadoop，还是使用自己的standalone计算框架

## 2. spark源码-DAG-Task--任务调度部分？
spark是粗粒度的资源申请，任务调度：sparkContext-DAGSheduler切分stage，TaskSheduler发送任务到申请好的Executor中的线程池执行

## 3.submit相关配置？一般指定多大的资源？
submit --master/yarn --class --deploy model clster/client
Executor cores 默认一个Executor 1 core，lg内存，1G，2--3个task

## 4.写完spark程序如何知道多少个task？（即资源如何调配的）
看你的并行度的设置，block的数量，web UI

## 5.spark和mr性能是不是差别很多？
一般来说Spark比Hadoop快：
原因：
a.MR有大量的磁盘io,溢写等，Spark则可以基于内存缓存机制计算
b.MR和Spark的资源申请的方式：粗粒度和细粒度的区别
c.DAG计算引擎中的pipeline计算模型，MR就是MapReduce模型
d.算子的丰富程度
使用场景：大于pb级别的数据量一般选择MR
生态的区别：Spark一站式的大数据处理平台，Hadoop还需要和其他的整合，升级，版本兼容等一堆问题，CDH版本如果需要更多的功能需要考虑成本的问题

## 6.spark任务yarn执行流程(client)?



## 7.spark运行在Yarn上流程（cluster）?
使用场景的区别：基于yarn的好处，兼容hadoop，一套计算框架，能好的维护

## 8.spark调优？



## 9.shuffle主要介绍下？
shuffle发生？---shuffle的过程---shuffle实现的选择---shuffle的优化



## 10.宽窄依赖？
看父RDD和子RR的关系，除了父RDD和子RDD一对多外，其他的都是窄依赖

## 11.shuffle怎么落地的？
shuffle的实现类型：hash Shuffle还是sortShuffle？Shuffle数据落地？

## 12.Spark RDD 是什么?
弹性分布式数据集---源码的五大特性-----RDD的计算模型：pipeline计算模型

## 13.Spark算子?
map，flatmap，filter，foreach，first，take(n),join,cogroup,reducebykey,sortBy,distinct,mapPartition等等

## 14.spark 优势？
一栈式大数据处理平台,灵活的编程模型,相比MR速度快

## 15.spark on yarn 和mapreduce中yarn有什么区别？
没什么区别，yarn就是一个资源管理框架

## 16.spark原理？
pipeline计算模型+任务调度和资源调度

## 17.spark运行的job在哪里可以看到？
Driver进程所在的节点；web UI

## 18.如何监测集群中cpu，内存的使用情况，比如说：有一个spark特别占资源，特别慢，怎么排查这种情况?
spark WEB UI；集群监控工具，找到taskid

## 20.rdd的处理过程是什么，不要说概念?
画切分Stage，pipeline的计算模型的图

## 21.spark的工作流程？
Spark的资源调度和任务调度+pipeline的计算模型

## 22.SparkSQL和Spark架构，运行流程图，Spark运行的两种方式。常用的Spark函数有哪些？
spark架构图+运行流程图(资源的调度+任务调度)+Spark client和SparkCluster+transformation算子+action算子+持久化操作算子

## 23.Spark了解多少？
spark生态-架构-运行模式+任务调度和资源调度

## 24：GroupByKey的作用？
根据key分组

## Spark Sql的面试题：

## 1.sparkSQL介绍下（RDD、DataFrame）

关于Spark Streaming的面试题：

## 2.sparkStreaming怎么跟kafka对接的,数据拉取到哪里？

## 3.日流量10G没必要sparkstreaming？

## 4.spark streaming 例子。问维护做过没？说sparkStreaming的维护成本很高。 我告诉他是的，比如说可能会丢数据，wal会慢。这一块儿不是我维护。没细问。

## 5.spark streming调优?

## 6.sparkstreaming原理？

## 7.spark Streaming介绍下？和Storm比较？

## 8.spark Streaming某一个task挂了，怎么解决的?

## 9.spark streaming?spark的相关算法，比如推荐系统需要什么算法?

## 10.spark streaming工作流程？

## 11.sparkstreanming没有问题，但无法计算，怎么排查问题?

## 12.storm和spark streaming的区别？
