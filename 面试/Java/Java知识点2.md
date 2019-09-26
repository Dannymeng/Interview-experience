# Java知识点2


## String、StringBuffer、StringBuilder的区别，怎么理解String不变性
- 可变与不可变
  - String 对象不可变
  - StringBuilder与StringBuffer对象可变
- 是否线程安全
  - String对象不可变，线程安全
  - StringBuffer使用同步锁，线程安全
  - StringBuilder线程不安全（效率高于StringBuffer）

## ==和equals的区别，如果重写了equals()不重写hashCode()会发生什么

answer1:
 - == : 比较的是对象的内存地址，用来判断两个对象是否是同一个对象
 
 - equals：比较的是对象的内容，但假如不重写equals,默认调用Object超类的equals（内部实现是“==” 运算符）

answers2:
**1、重写equals方法时需要重写hashCode方法，主要是针对Map、Set等集合类型的使用；**

a: Map、Set等集合类型存放的对象必须是唯一的；

b: 集合类判断两个对象是否相等，是先判断equals是否相等，如果equals返回TRUE，还要再判断HashCode返回值是否ture,只有两者都返回ture,才认为该两个对象是相等的。

2、由于Object的hashCode返回的是对象的hash值，所以即使equals返回TRUE，集合也可能判定两个对象不等，所以必须重写hashCode方法，以保证当equals返回TRUE时，hashCode也返回Ture，这样才能使得集合中存放的对象唯一。



总结重写equals()和重写hashcode()关系：
- 1、如果两个对象相同（即用equals比较返回true），那么它们的hashCode值一定要相同；

- 2、如果两个对象的hashCode相同，它们并不一定相同(即用equals比较返回false)





