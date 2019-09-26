# Spark整理

[Spark原理精解](https://blog.csdn.net/liuxiangke0210/article/details/79687240)
## Spark基本概念
1. **Application：Spark应用程序**
    - 指的是用户编写的Spark应用程序，包含了Driver功能代码和分布在集群中多个节点上运行的Executor代码。
    
![Application]($resource/Application.jpg)
2. **Driver（也叫做SparkSubmit）：驱动程序**
    - Spark中的Driver即运行上述Application的Main()函数并且创建SparkContext，其中创建SparkContext的目的是**为了准备Spark应用程序的运行环境。在Spark中由SparkContext负责和ClusterManager通信，进行资源的申请、任务的分配和监控等**;当Executor部分运行完毕后，Driver负责将SparkContext关闭。通常SparkContext代表Driver，如下图所示: 
    ![2.jpg](http://www.raincent.com/uploadfile/2018/0320/20180320013155532.jpg)
    
3. **Cluster Manager：资源管理器**
  -  指的是在集群上获取资源的外部服务，常用的有：**Standalone**，Spark原生的资源管理器，由Master负责资源的分配;**Hadoop Yarn**，由Yarn中的ResearchManager负责资源的分配;Messos，由Messos中的Messos Master负责资源管理，如下图所示:
  
![Cluster Manager]($resource/Cluster%20Manager.jpg)

4. **Executor：执行器**
       Application运行在Worker节点上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上，每个Application都有各自独立的一批Executor，如下图所示:
  
![Executor]($resource/Executor.jpg)

5. **Worker：计算节点**
        集群中任何可以运行Application代码的节点，类似于Yarn中的NodeManager节点。在Standalone模式中指的就是通过Slave文件配置的Worker节点，在Spark on Yarn模式中指的就是NodeManager节点，在Spark on Messos模式中指的就是Messos Slave节点，如下图所示:

![Worker]($resource/Worker.jpg)

6. **RDD：弹性分布式数据集**
Resillient Distributed Dataset，Spark的基本计算单元，可以通过一系列算子进行操作(主要有Transformation和Action操作)，如下图所示：

![RDD]($resource/RDD.jpg)

7. **父RDD每一个分区最多被一个子RDD的分区所用**;表现为一个父RDD的分区对应于一个子RDD的分区，或两个父RDD的分区对应于一个子RDD 的分区。如图所示:

![窄依赖]($resource/%E7%AA%84%E4%BE%9D%E8%B5%96.jpg)

8. **宽依赖**

父RDD的每个分区都可能被多个子RDD分区所使用，子RDD分区通常对应所有的父RDD分区。如图所示:

![宽依赖]($resource/%E5%AE%BD%E4%BE%9D%E8%B5%96.jpg)

常见的窄依赖有：map、filter、union、mapPartitions、mapValues、join(父RDD是hash-partitioned ：如果JoinAPI之前被调用的RDD API是宽依赖(存在shuffle), 而且两个join的RDD的分区数量一致，join结果的rdd分区数量也一样，这个时候join api是窄依赖)。

常见的宽依赖有groupByKey、partitionBy、reduceByKey、join(父RDD不是hash-partitioned ：除此之外的，rdd 的join api是宽依赖)。

9. **DAG：有向无环图**
Directed Acycle graph，反应RDD之间的依赖关系，如图所示:
![DAG]($resource/DAG.jpg)

10. **DAGScheduler：有向无环图调度器**
基于DAG划分Stage 并以TaskSet的形式提交Stage给TaskScheduler;负责将作业拆分成不同阶段的具有依赖关系的多批任务;最重要的任务之一就是：计算作业和任务的依赖关系，制定调度逻辑。在SparkContext初始化的过程中被实例化，一个SparkContext对应创建一个DAGScheduler。
![DAGScheduler]($resource/DAGScheduler.jpg)

11. **TaskScheduler：任务调度器**
将Taskset提交给worker(集群)运行并回报结果;负责每个具体任务的实际物理调度。如图所示:

![TaskScheduler]($resource/TaskScheduler.jpg)

12. **Job：作业**
由一个或多个调度阶段所组成的一次计算作业;包含多个Task组成的并行计算，往往由Spark Action催生，一个JOB包含多个RDD及作用于相应RDD上的各种Operation。如图所示:

![Job]($resource/Job.jpg)

13. **Stage：调度阶段**

一个任务集对应的调度阶段;每个Job会被拆分很多组Task，每组任务被称为Stage，也可称TaskSet，一个作业分为多个阶段;Stage分成两种类型ShuffleMapStage、ResultStage。如图所示:
![Stage]($resource/Stage.jpg)

14. **TaskSet：任务集**

由一组关联的，但相互之间没有Shuffle依赖关系的任务所组成的任务集。如图所示:
![TaskSet]($resource/TaskSet.jpg)

提示：

1)一个Stage创建一个TaskSet;
2)为Stage的每个Rdd分区创建一个Task,多个Task封装成TaskSet

15. **Task：任务**

被送到某个Executor上的工作任务;单个分区数据集上的最小处理流程单元。如图所示:

![Task]($resource/Task.jpg)

总体如图所示：
![总体图]($resource/%E6%80%BB%E4%BD%93%E5%9B%BE.jpg)

## Spark运行流程
![Spark运行基本流程1]($resource/Spark%E8%BF%90%E8%A1%8C%E5%9F%BA%E6%9C%AC%E6%B5%81%E7%A8%8B1.jpg)

![Spark运行基本流程2]($resource/Spark%E8%BF%90%E8%A1%8C%E5%9F%BA%E6%9C%AC%E6%B5%81%E7%A8%8B2.jpg)

## Spark核心原理透视

**1. 计算流程**
![计算流程]($resource/%E8%AE%A1%E7%AE%97%E6%B5%81%E7%A8%8B.jpg)

**2、从代码构建DAG图**

Spark program
Val lines1 = sc.textFile(inputPath1). map(···)). map(···)
Val lines2 = sc.textFile(inputPath2) . map(···)
Val lines3 = sc.textFile(inputPath3)
Val dtinone1 = lines2.union(lines3)
Val dtinone = lines1.join(dtinone1)
dtinone.saveAsTextFile(···)
dtinone.filter(···).foreach(···)

Spark的计算发生在RDD的Action操作，而对Action之前的所有Transformation，Spark只是记录下RDD生成的轨迹，而不会触发真正的计算。

Spark内核会在需要计算发生的时刻绘制一张关于计算路径的有向无环图，也就是DAG。

![构建DAG图]($resource/%E6%9E%84%E5%BB%BADAG%E5%9B%BE.jpg)

**3、将DAG划分为Stage核心算法**

Application多个job多个Stage：Spark Application中可以因为不同的Action触发众多的job，一个Application中可以有很多的job，每个job是由一个或者多个Stage构成的，后面的Stage依赖于前面的Stage，也就是说只有前面依赖的Stage计算完毕后，后面的Stage才会运行。

划分依据：**Stage划分的依据就是是否存在ShuffleDependency宽依赖**，何时产生宽依赖，reduceByKey, groupByKey等算子，会导致宽依赖的产生。

核心算法：**从后往前回溯，遇到窄依赖加入本stage，遇见宽依赖进行Stage切分**。Spark内核会从触发Action操作的那个RDD开始从后往前推，首先会为最后一个RDD创建一个stage，然后继续倒推，如果发现对某个RDD是宽依赖，那么就会将宽依赖的那个RDD创建一个新的stage，那个RDD就是新的stage的最后一个RDD。然后依次类推，继续继续倒推，根据窄依赖或者宽依赖进行stage的划分，直到所有的RDD全部遍历完成为止。

**6、提交Stages**

调度阶段的提交，最终会被转换成一个任务集的提交，DAGScheduler通过TaskScheduler接口提交任务集，这个任务集最终会触发TaskScheduler构建一个TaskSetManager的实例来管理这个任务集的生命周期，对于DAGScheduler来说，提交调度阶段的工作到此就完成了。而TaskScheduler的具体实现则会在得到计算资源的时候，进一步通过TaskSetManager调度具体的任务到对应的Executor节点上进行运算。
![提交stage]($resource/%E6%8F%90%E4%BA%A4stage.jpg)

**8、监控Job、Task、Executor**

DAGScheduler监控Job与Task：要保证相互依赖的作业调度阶段能够得到顺利的调度执行，DAGScheduler需要监控当前作业调度阶段乃至任务的完成情况。这通过对外暴露一系列的回调函数来实现的，对于TaskScheduler来说，这些回调函数主要包括任务的开始结束失败、任务集的失败，DAGScheduler根据这些任务的生命周期信息进一步维护作业和调度阶段的状态信息。

DAGScheduler监控Executor的生命状态：TaskScheduler通过回调函数通知DAGScheduler具体的Executor的生命状态，如果某一个Executor崩溃了，则对应的调度阶段任务集的ShuffleMapTask的输出结果也将标志为不可用，这将导致对应任务集状态的变更，进而重新执行相关计算任务，以获取丢失的相关数据。

**9、获取任务执行结果**

结果DAGScheduler：一个具体的任务在Executor中执行完毕后，其结果需要以某种形式返回给DAGScheduler，根据任务类型的不同，任务结果的返回方式也不同。

两种结果，中间结果与最终结果：对于FinalStage所对应的任务，返回给DAGScheduler的是运算结果本身，而对于中间调度阶段对应的任务ShuffleMapTask，返回给DAGScheduler的是一个MapStatus里的相关存储信息，而非结果本身，这些存储位置信息将作为下一个调度阶段的任务获取输入数据的依据。

两种类型，DirectTaskResult与IndirectTaskResult：根据任务结果大小的不同，ResultTask返回的结果又分为两类，如果结果足够小，则直接放在DirectTaskResult对象内中，如果超过特定尺寸则在Executor端会将DirectTaskResult先序列化，再把序列化的结果作为一个数据块存放在BlockManager中，然后将BlockManager返回的BlockID放在IndirectTaskResult对象中返回给TaskScheduler，TaskScheduler进而调用TaskResultGetter将IndirectTaskResult中的BlockID取出并通过BlockManager最终取得对应的DirectTaskResult。

**10、任务调度总体诠释**

![任务调度总体诠释]($resource/%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6%E6%80%BB%E4%BD%93%E8%AF%A0%E9%87%8A.jpg)

## Spark shuffle过程
shuffle发生在两个stage之间




