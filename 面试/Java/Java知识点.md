# Java知识点


##  集合、线程



### **HashTable实现原理，HashMap实现原理，ConcurrentHashMap实现原理**
[1、jdk1.7 HashMap原理](https://www.cnblogs.com/chengxiao/p/6059914.html)

[ 2、Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](http://www.importnew.com/28263.html)

[ 3、ConcurrentHashMap在jdk1.8中的改进](https://www.cnblogs.com/everSeeker/p/5601861.html)

**jdk1.8 HashMap结构图：**
![2](https://javadoop.com/blogimages/map/2.png)

**jdk1.8实现方式与jdk1.7不同：**
- HashMap和ConcurrentHashMap都加入了**红黑树**，对于个数超过8个（默认值）的列表，jdk1.8采用红黑树的结构，查询的时间复杂度可以降到O(logN)
- ConcurrentHashMap：取消segments字段，直接采用transient volatile HashEntry<K,V>[] table保存数据，采用table数组元素作为锁，从而实现了对每一行数据进行加锁，进一步减少并发冲突的概率

**以下为jdk1.7实现方式**
**HashTable**
*   底层数组+单向链表实现，无论key还是value都**不能为null**，线程**安全**，实现线程安全的方式是在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化
*   初始size为**11**，扩容：newsize = olesize*2+1
*   计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length

**HashMap**
*   底层数组+单向链表实现，可**以存储null键和null值**，线程**不安全**
*   初始size为**16**，扩容：newsize = oldsize*2，size一定为2的n次幂
*   扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入
*   插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）
*   当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀
*   计算index方法：index = hash & (tab.length – 1)

HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。
```java
//HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}，主干数组的长度一定是2的次幂，至于为什么这么做，后面会有详细分析。
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```
HashMap和Hashtable都是用hash算法来决定其元素的存储，因此HashMap和Hashtable的hash表包含如下属性：

*   容量（capacity）：hash表中桶的数量
*   初始化容量（initial capacity）：创建hash表时桶的数量，HashMap允许在构造器中指定初始化容量
*   尺寸（size）：当前hash表中记录的数量
*   负载因子（load factor）：负载因子等于“size/capacity”。负载因子为0，表示空的hash表，0.5表示半满的散列表，依此类推。轻负载的散列表具有冲突少、适宜插入与查询的特点（但是使用Iterator迭代元素时比较慢）

除此之外，hash表里还有一个“负载极限”，“负载极限”是一个0～1的数值，“负载极限”决定了hash表的最大填满程度。当hash表中的负载因子达到指定的“负载极限”时，hash表会自动成倍地增加容量（桶的数量），并将原有的对象重新分配，放入新的桶内，这称为rehashing。

**“负载极限”的默认值（0.75）是时间和空间成本上的一种折中**

*   较高的“负载极限”可以降低hash表所占用的内存空间，但会增加查询数据的时间开销，而查询是最频繁的操作（HashMap的get()与put()方法都要用到查询）
*   较低的“负载极限”会提高查询数据的性能，但会增加hash表所占用的内存开销

![](https://images2015.cnblogs.com/blog/1024555/201611/1024555-20161113235348670-746615111.png)


**ConcurrentHashMap**
*   底层采用分段的数组+单向链表实现，线程**安全**
*   通过把整个Map分为N个Segment，可以提供相同的线程安全，但是效率提升N倍，默认提升16倍。(读操作不加锁，由于HashEntry的value变量是 volatile的，也能保证读取到最新的值。)
*   Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术
*   有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁
*   扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容


ConcurrentHashMap比HashMap多出了一个类Segment，而Segment是一个可重入锁。

ConcurrentHashMap是使用了锁分段技术来保证线程安全的。

**锁分段技术**：首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。 

ConcurrentHashMap提供了与Hashtable和SynchronizedMap不同的锁机制。**Hashtable中采用的锁机制是一次锁住整个hash表**，从而在同一时刻只能由一个线程对其进行操作；而**ConcurrentHashMap中则是一次锁住一个桶**。

ConcurrentHashMap默认将hash表分为16个桶，诸如get、put、remove等常用操作只锁住当前需要用到的桶。这样，原来只能一个线程进入，现在却能同时有16个写线程执行，并发性能的提升是显而易见的。


### **HashMap在高并发环境下会产生的问题**
 问题：
 - HashMap死循环，照成CPU100%负载
 - 触发fail-fast

**1、HashMap死循环的原因**

HashMap进行存储时，如果size超过（当前最大容量*负载因子）时候会发生resize，首先看一下resize源代码：
```java
void resize(int newCapacity) { 
  Entry[] oldTable = table; 
  int oldCapacity =  oldTable.length; 
  if (oldCapacity == MAXIMUM_CAPACITY) { 
      threshold = Integer.MAX_VALUE; 
      return; 
   } 
  Entry[] newTable = new Entry[newCapacity]; 
  // transfer方法是真正执行rehash的操作，容易在高并发时发生问题 
  transfer(newTable); 
  table = newTable; 
  threshold = (int)(newCapacity * loadFactor); 
}
```
**而这段代码中又调用了transfer()方法，而这个方法实现的机制就是将每个链表转化到新链表，并且链表中的位置发生反转，而这在多线程情况下是很容易造成链表回路，从而发生死循环**，我们看一下他的源代码:
```java
void tansfer(Entry[] newTable) {
	   Entry[] src = table;
	   int newCapacity = newTable.length;
	   for (int j = 0; j < src.length; j++) {
	       Entry<K,V> e = src[j];
	       if (e != null) {
	           src[j] = null;
	           do {
	               Entry<K,V> next = e.next;
	               int i = indexFor(e.hash, newCapacity);
	               e.next = newTable[i];
	               newTable[i] = e;
	               e = next;
	           } while (e != null);
	       }
	   }
	}
```
**2、触发fail-fast：**
**一个线程利用迭代器迭代时，另一个线程做插入删除操作，造成迭代的fast-fail。**
```java
public class TestFailFast {
    
    private static final String USER_NAME_PREFIX = "User-";
    // Key: User Name, Value: User Age
    private static Map<String, Integer> userMap = new HashMap<>();
    
    // ThreadA 用于向HashMap添加元素
    static class ThreadA implements Runnable {
        @Override
        public void run() {
            System.out.println("ThreadA starts to add user.");
            for (int i = 1; i < 100000; i++) {
                userMap.put(USER_NAME_PREFIX+i, i%100);
            }
            System.out.println("ThreadA done.");
        }
    }
    
    // ThreadB 用于遍历HashMap中元素输出
    static class ThreadB implements Runnable {
        @Override
        public void run() {
            System.out.println("ThreadB starts to iterate.");
            for (Map.Entry<String, Integer> user : userMap.entrySet()) {
                System.out.println("UserName=" + user.getKey()
                    + ", UserAge=" + user.getValue());
            }
            System.out.println("ThreadB done.");
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread threadA = new Thread(new ThreadA());
        Thread threadB = new Thread(new ThreadB());
    
        threadA.start();
        threadB.start();
  
        threadA.join();
        threadB.join();
        System.exit(0);
    }
}
```
运行结果：抛出ConcurrentModificationException












### **红黑树，为什么允许局部不平衡**





### **什么是一致性Hash算法**

原因：
应用场景：redis缓存假设使用普通hash取模的方式。在增加节点或减少节点时，映射规则将被改变，当应用无法从缓存中获取数据，则会想后端数据库请求数据

![640](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/UtWdDgynLdbhiae1AfNYAibdp7ib2wTZTrpnNGzjcWYy3ylL7s1Bq2UKicU5mYG8SHsuIFTOf2PMe2FstpM2gMeQbw/640)

- hash值是个整数非负，非负整数的值范围做成一个圆环`Hash环`（0——2^32-1）；
- 对集群的节点的某个属性（比如节点名）求hash值，放在环上；
- 对数据key求hash值，也放在环上，按顺时针方向找到离它最近的节点，放在它上面。

`附` 如何避免Hash环的数据倾斜？

引入`虚拟节点`，例如将每台服务器就算三个虚拟节点，于是可以分别计算 “Node A#1”、“Node A#2”、“Node A#3”、“Node B#1”、“Node B#2”、“Node B#3”的哈希值，于是形成六个虚拟节点
![640](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/UtWdDgynLdbhiae1AfNYAibdp7ib2wTZTrpnNGzjcWYy3ylL7s1Bq2UKicU5mYG8SHsuIFTOf2PMe2FstpM2gMeQbw/640)

[1、参考文章](https://blog.csdn.net/bntX2jSQfEHy7/article/details/79549368)

### 有哪几种常用的线程池

线程简介：线程是操作系统能够进行运算调度的最小单位，线程有就绪，阻塞，运行三种基本状态。

线程池概念：线程池是一种多线程处理形式，处理过程中将任务添加队列，然后在创建线程后自动启动这些任务，每个线程都使用默认的堆栈大小，以默认的优先级运行，并处在多线程单元中，如果某个线程在托管代码中空闲，则线程池将插入另一个辅助线程来使所有处理器保持繁忙。如果所有线程池都始终保持繁忙，但队列中包含挂起的工作，则线程池将在一段时间后辅助线程的数目永远不会超过最大值。超过最大值的线程可以排队，但他们要等到其他线程完成后才能启动。

java里面的线程池的顶级接口是**Executor**，Executor并不是一个线程池，而只是一个执行线程的工具，而真正的线程池是**ExecutorService**。

**java中的有哪些线程池？**
- 1.newCachedThreadPool创建一个可缓存线程池程
- 2.newFixedThreadPool 创建一个定长线程池
- 3.newScheduledThreadPool 创建一个 定时的加载线程池（定时以及周期性执行任务）
- 4.newSingleThreadExecutor 创建一个单线程化的线程池

**1、newCachedThreadPool**是一种线程数量不定的线程池，并且其最大线程数为Integer.MAX_VALUE，这个数是很大的，一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。但是线程池中的空闲线程都有超时限制，这个超时时长是60秒，超过60秒闲置线程就会被回收。调用execute将重用以前构造的线程(如果线程可用)。这类线程池比较适合执行大量的耗时较少的任务，当整个线程池都处于闲置状态时，线程池中的线程都会超时被停止。

**2.newFixedThreadPool** ：创建一个指定工作线程数量的线程池，每当提交一个任务就创建一个工作线程，当线程 处于空闲状态时，它们并不会被回收，除非线程池被关闭了，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列（没有大小限制）中。由于newFixedThreadPool只有核心线程并且这些核心线程不会被回收，这样它更加快速底相应外界的请求。

**3.newScheduledThreadPool**：创建一个线程池，它的核心线程数量是固定的，而非核心线程数是没有限制的，并且当非核心线程闲置时会被立即回收，它可安排给定延迟后运行命令或者定期地执行。这类线程池主要用于执行定时任务和具有固定周期的重复任务。

**4.newSingleThreadExecutor**这类线程池内部只有一个核心线程，以无界队列方式来执行该线程，这使得这些任务之间不需要处理线程同步的问题，它确保所有的任务都在同一个线程中按顺序中执行，并且可以在任意给定的时间不会有多个线程是活动的。
[ 1、java中的线程池有哪些，分别有什么作用？](https://blog.csdn.net/qq_33453910/article/details/81413285)

[2、Callable,Runnable比较及用法以及创建线程的4种方法](https://blog.csdn.net/xyw591238/article/details/51900325)


### 什么情况下使用Runnable和Thread创建线程，Runnable和Callable的区别

**1、什么情况下使用Runnable和Thread创建线程？**
- 如果这句话问的是什么情况下创建线程？：
  多线程是为了提高程序的效率
- 如果问的是Runnable和Thread的区别？：
 Runnable和Thread都可以实现run方法，但一个是接口，一个是类，前者可以无限地创建Thread进行run，而后者进行一次run之后就无法再次run。注意：Thread执行了start之后不可以再次执行start，**因此，要实现线程能重复运行，如果采用XXX extends Thread，那么每次运行都必须new一个XXX，这十分损耗资源；如果使用XXX implements Runnable，那每次运行只需要新开一个线程new Thread(xxx)即可，节省了很多时空消耗**

**2、Runnable和Callable的区别**

- 相同点：
  - 1.  两者都是接口；
  - 2.  两者都可用来编写多线程程序；
  - 3.  两者都需要调用Thread.start()启动线程；
 - 不同点：
   - 1. Callable规定的方法是call(),Runnable规定的方法是run();
   - 2. 两者最大的不同点是：实现Callable接口的任务线程能返回执行结果；而实现Runnable接口的任务线程不能返回结果；
   - 3. Callable接口的call()方法允许抛出异常；而Runnable接口的run()方法的异常只能在内部消化，不能继续上抛
   - 4. 运行Callable任务可以拿到一个Future对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果.

[1、Runnable与Callable异同](https://www.cnblogs.com/frinder6/p/5507082.html)

[2、 Callable,Runnable比较及用法以及创建线程的4种方法](https://blog.csdn.net/xyw591238/article/details/51900325)

### 线程方法中的异常如何处理，副线程可以捕获到吗?

捕获线程中异常的方式：（单纯使用try{}catch（）方式捕获不到异常）
-  Thread方式通过线程组,线程名，并设置**UncaughtExceptionHandler**来捕获异常
```java
public static void main(String[] args) {
        try{
            Thread t =new Thread(new Runnable(){
                @Override
                public void run() {
                    int i = 10/0;
                    System.out.println("run....");
                }
            });
            t.setUncaughtExceptionHandler(
                new UncaughtExceptionHandler() {
                @Override
                public void uncaughtException(Thread t,   Throwable e) {
                    System.out.println("catch 到了");
                }
            });
            t.start();
        }catch(Exception e){
            System.out.println("catch 不到");
        }
    }
    //输出： catch 到了
```


### **synchronized和锁的区别，什么情况下使用synchronized和ReentrantLock**

**1、synchronized和锁的区别**
- Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；
- 采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象;
- 用法不一样。synchronized既可以加在方法上，也可以加载特定的代码块上，括号中表示需要锁的对象。而Lock需要显示地指定起始位置和终止位置。synchronzied是托管给jvm执行的，Lock锁定是通过代码实现的;
- 锁的机制不一样。synchronized获得锁和释放的方式都是在块结构中，而且是自动释放锁。而Lock则需要开发人员手动去释放，并且必须在finally块中释放，否则会引起死锁问题的发生;
- synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；
- Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。Lock可以提高多个线程进行读操作的效率

**什么情况下使用synchronized和ReentrantLock**





## **网络**

### **TCP三次握手**
  - 第一次握手：建立连接时，客户端发送syn（Synchronize Sequence Numbers：同步序列编号）包（sny=1）到服务器，并进入SYN_SEND（请求连接）状态，等待服务器确认;
  - 第二次握手：服务器接收到syn包，必须确认客户的syn（ack=j+1）（ack：确认字符，表示发来的数据已确认接收无误），同时自己也发送一个syn包（sny=k），既ack+syn包，此时服务器进入SYN_RECV（发送了ACK时的状态）状态。
  - 第三次握手：客户端收到服务端发送的syn+ack包，向服务端发送确认包ack（sny+1既ack=k+1），此包发送完毕，客户端与服务器进入ESTABLISHED（TCP连接成功）状态，完成三次握手。
  
![TCP三次握手图](https://img-blog.csdn.net/20180830135313575?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nemhhb3lhbmcxMjI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### **TCP四次挥手**

  - 第一次挥手：TCP客户端发送一个FIN+ACK+SEQ，用来传输关闭客户端到服务端的数据。进入FIN_WAIT1状态。
  - 第二次挥手：服务端收到FIN，被动发送一个ACK（SEQ+1）,进入CLOSE_WAIT状态，客户端收到服务端发送的ACK，进入FIN_WAIT2状态。
  - 第三次挥手：服务器关闭客户端连接，发送一个FIN给+ACK+SEQ客户端。进入LAST_ACK状态。
  - 第四次挥手：客户端发送ACK（ACK=SQE序号+1）报文确认，客户端进入TIME_WAIT状态，服务端收到ACK进入CLOSE状态。
  
![](https://img-blog.csdn.net/20180830142208139?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nemhhb3lhbmcxMjI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### **说说几种HTTP响应码**
      ☆ 200 OK：表示客户端请求成功。
      ☆ 400 Bad Request 语义有误，不能被当前服务器理解。
      ☆ 401 Unauthorized 当前请求需要用户验证。
      ☆ 403 Forbidden 服务器收到消息，但是拒绝提供服务。
      ☆ 404 Not Found 请求资源不存在。
      ☆ 408 Request Timeout 请求超时，客户端没有在服务器预备等待的时间内完成发送。
      ☆ 500 Internal Server Error 服务器发生不可预期的错误。
      ☆ 503 Server Unavailable 由于临时的服务器维护或过载，服务器当前不能处理请求，此状况知识临时的，可恢复

### **当你用浏览器打开一个链接时，计算机做了哪些步骤**
  - 1、解析域名
  - 2、发次TCP的3次握手
  - 3、建立TCP请求后发起HTTP请求
  - 4、服务器相应的HTTP请求
  - 5、浏览器得到HTML代码，进行解析和处理JSON数据，并请求HTML代码中的静态资源（JS、CSS、图片等）
  - 6、浏览器对页面进行渲染 

[1、网络面试知识点1](https://blog.csdn.net/zhengzhaoyang122/article/details/82184072)

[2、网络面试知识点2](https://www.cnblogs.com/huajiezh/p/7492416.html)

### **TCP，UDP区别，为什么可靠和不可靠**

![1553517637]($resource/1553517637.png)

- 1、TCP面向连接（如打电话要先拨号建立连接）;UDP是无连接的，即发送数据之前不需要建立连接
- 2、TCP提供可靠的服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达;UDP尽最大努力交付，即不保证可靠交付
- 3、TCP面向字节流，有流量拥塞控制。实际上是TCP把数据看成一连串无结构的字节流;UDP是面向报文的，UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如IP电话，实时视频会议等）
- 4、每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信 
- 5、TCP首部开销20字节;UDP的首部开销小，只有8个字节 
- 6、TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道

**为什么可靠和不可靠：**
UDP:
UDP只有一个socket接受缓冲区，没有socket发送缓冲区，即只要有数据就发，不管对方是否可以正确接受。而在对方的socket接受缓冲区满了之后，新来的数据报无法进入到socket接受缓冲区，此数据报就会被丢弃，udp是没有流量控制的，故UDP的数据传输是不可靠的。
TCP：
每个Tcp socket在内核中都有一个发送缓冲区和一个接受缓冲区。tcp协议要求对端在接受到tcp数据报之后，要对其序号进行ACK，只有当接受到一个tcp数据报的ACK之后，才可以把这个tcp数据报从socket的发送缓冲区清除，另外tcp还有一个流量控制功能，tcp的socket接受缓冲区接受到网络上来的数据缓存起来后，如果应用程序一直没有读取，

socket接受缓冲区满了之后，发生的动作是：通知对端TCP协议中的窗口关闭，这便是滑动窗口的实现，保证TCP socket接受缓冲区不会溢出，因为对方不允许发送超过所通知窗口大小的数据， 这就是TCP的流量控制，如果对方无视窗口大小而发出了超过窗口大小的数据，则接收方TCP将丢弃它。这两点保证了tcp是可靠传输的。

## MySQL事务是什么？四大特性、四大隔离级别

**事务**：事务是由一步或几步数据库操作序列组成逻辑执行单元，这系列操作要么全部执行，要么全部放弃执行。程序和事务是两个不同的概念。一般而言：**一段程序中可能包含多个事务**。（**说白了就是几步的数据库操作而构成的逻辑执行单元**）

**四大特性：**
- 原子性：事务是一个不可分割的整体，事务开始的操作，要么全部执行，要么全部不执行
- 隔离性：同一时间，只允许一个事务请求同一组数据。不同的事务彼此之间没有干扰。
- 一致性：事务开始前和结束后，数据库的完整性约束没有被破坏 。
- 稳定性：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。

**四大隔离级别**
- 1、read uncommitted（读取未提交内容）。事务A对数据进行修改，但未提交。此时开启事务B，在事务B中能读到事务A中对数据库进行的未提交数据的修改。（这种方式也称为**脏读**）
- 2、read committed（读取提交内容）。事务A对数据进行修改，但还未提交。此时开启事务B，在事务B中不能读到事务A中对数据库的修改。在事务B还没有关闭时，此时事务A提交对数据库的修改。这时候，我们在事务B中，可以查到事务A中对数据库的修改。这时存在一个问题，我们在同一个事务中，对数据库查询两次，但两次的结果是不一样的。（这种方式称为不可重复读。）
- 3、repetition read（可重读：**默认情况下使用**）。事务A对数据进行修改，但未提交，此时开启事务B，在事务B中不能读到事务A对数据库的修改。在事务A提交对数据库修改时，此时在事务B中，仍不能读到事务A对数据库的修改。（这种方式称为可重复读）但此时有一个弊端，比如我们在事务A中对数据库增加一条数据，id 为 n ，这时候我们在事务B中查询数据，此时查不到id为n的数据。但当我们在事务B中增加id为n的数据时，系统会提示id为n的数据已经存在，我们添加失败。但此时此刻，我们在事务B中仍不能查询到id为n的数据。这种方式存在一个幻读的概念。举个例子，（系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后关闭事务发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。）
- 4、serializable（可串行化）。在开启事务A时，会产生锁表，此时别的事务会等待，等事务A结束时才会开启。

[ 1、 MySQL数据库事务的四大特性以及事务的隔离级别](https://blog.csdn.net/l1394049664/article/details/81814090)
## 算法
### 快排


```java
void quickSort(int[] num, int start, int end) {
	if (num.length == 0)
		return;
	if (start >= end)
		return;

	int low = start;
	int high = end;
	int temp = num[low];
	while (low < high) {
		while (low < high && num[high] > temp) {
			high--;
		}
		num[low] = num[high];
		while (low < high &&num[low] <= temp) {
			low++;
	  }
	   num[high] = num[low];
	}
	num[low] = temp;
	quickSort(num, start, low - 1);
	quickSort(num, high + 1, end);

}
```
[1、快速排序基本思想](https://blog.csdn.net/nrsc272420199/article/details/82587933)


