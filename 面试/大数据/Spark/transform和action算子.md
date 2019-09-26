# transform和action算子


## **常见transform算子：**
1\. map(func):对调用map的RDD数据集中的每个element都使用func，然后返回一个新的RDD,这个返回的数据集是分布式的数据集
2\. filter(func) : 对调用filter的RDD数据集中的每个元素都使用func，然后返回一个包含使func为true的元素构成的RDD
3\. flatMap(func):和map差不多，但是flatMap生成的是多个结果
4\. mapPartitions(func):和map很像，但是map是每个element，而mapPartitions是每个partition
5\. mapPartitionsWithSplit(func):和mapPartitions很像，但是func作用的是其中一个split上，所以func中应该有index
6\. sample(withReplacement,faction,seed):抽样
7\. union(otherDataset)：返回一个新的dataset，包含源dataset和给定dataset的元素的集合
8\. distinct([numTasks]):返回一个新的dataset，这个dataset含有的是源dataset中的distinct的element
9\. groupByKey(numTasks):返回(K,Seq[V])，也就是hadoop中reduce函数接受的key-valuelist
10\. reduceByKey(func,[numTasks]):就是用一个给定的reduce func再作用在groupByKey产生的(K,Seq[V]),比如求和，求平均数
11\. sortByKey([ascending],[numTasks]):按照key来进行排序，是升序还是降序，ascending是boolean类型
12\. join(otherDataset,[numTasks]):当有两个KV的dataset(K,V)和(K,W)，返回的是(K,(V,W))的dataset,numTasks为并发的任务数
13\. cogroup(otherDataset,[numTasks]):当有两个KV的dataset(K,V)和(K,W)，返回的是(K,Seq[V],Seq[W])的dataset,numTasks为并发的任务数
14\. cartesian(otherDataset)：笛卡尔积就是m*n，大家懂的

## **action常见算子**
1\. reduce(func)：说白了就是聚集，但是传入的函数是两个参数输入返回一个值，这个函数必须是满足交换律和结合律的
2\. collect()：一般在filter或者足够小的结果的时候，再用collect封装返回一个数组
3\. count():返回的是dataset中的element的个数
4\. first():返回的是dataset中的第一个元素
5\. take(n):返回前n个elements，这个士driver program返回的
6\. takeSample(withReplacement，num，seed)：抽样返回一个dataset中的num个元素，随机种子seed
7\. saveAsTextFile（path）：把dataset写到一个text file中，或者hdfs，或者hdfs支持的文件系统中，spark把每条记录都转换为一行记录，然后写到file中
8\. saveAsSequenceFile(path):只能用在key-value对上，然后生成SequenceFile写到本地或者hadoop文件系统
9\. countByKey()：返回的是key对应的个数的一个map，作用于一个RDD
10\. foreach(func):对dataset中的每个元素都使用func

[常见算子](https://my.oschina.net/134596/blog/3037972)
