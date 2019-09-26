# 尽量使用ReduceByKey代替GroupByKey

ReduceByKey具体流程：

![](https://img-blog.csdn.net/20151121161604854?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

GroupByKey具体流程：
![](https://img-blog.csdn.net/20151121162115121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

```scala
val words = Array("one", "two", "two", "three", "three", "three")
val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))
 
val wordCountsWithReduce = wordPairsRDD
  .reduceByKey(_ + _)
  .collect()
 
val wordCountsWithGroup = wordPairsRDD
  .groupByKey()
  .map(t => (t._1, t._2.sum))
  .collect()
```
虽然两个函数都能得出正确的结果， 但`reduceByKey`函数更适合使用在大数据集上
groupByKey会加大超过内存容量会存在内存溢出的异常风险


