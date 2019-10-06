<!-- GFM-TOC -->
* [一、数据类型](#一数据类型)
    * [基本类型](#基本类型)
    * [包装类型](#包装类型)
    * [缓存池](#缓存池)
* [二、String](#二string)
    * [概览](#概览)
    * [不可变的好处](#不可变的好处)
    * [String, StringBuffer and StringBuilder](#string,-stringbuffer-and-stringbuilder)
    * [String Pool](#string-pool)
    * [new String("abc")](#new-string"abc")
* [三、运算](#三运算)
    * [参数传递](#参数传递)
    * [float 与 double](#float-与-double)
    * [隐式类型转换](#隐式类型转换)
    * [switch](#switch)
* [四、继承](#四继承)
    * [访问权限](#访问权限)
    * [抽象类与接口](#抽象类与接口)
    * [super](#super)
    * [重写与重载](#重写与重载)
* [五、Object 通用方法](#五object-通用方法)
    * [概览](#概览)
    * [equals()](#equals)
    * [hashCode()](#hashcode)
    * [toString()](#tostring)
    * [clone()](#clone)
* [六、关键字](#六关键字)
    * [final](#final)
    * [static](#static)
* [七、反射](#七反射)
* [八、异常](#八异常)
* [九、泛型](#九泛型)
* [十、注解](#十注解)
* [十一、特性](#十一特性)
    * [Java 各版本的新特性](#java-各版本的新特性)
    * [Java 与 C++ 的区别](#java-与-c-的区别)
    * [JRE or JDK](#jre-or-jdk)
* [参考资料](#参考资料)
<!-- GFM-TOC -->



# 一、数据类型

## 1.1基本类型

- byte/8
- char/16  [长度和编码有关，ASCII码中char占一个字节，UTF-8是不定长编码，编码的长度是动态的]
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/8  [理论上只占据一个bit，但是实际是占据了一个byte]

其中 单位是bit, 一个bit代表一个1或者0，是计算机的基本单位

整数型：byte，short，int，long；

浮点型：float，double

布尔型：boolean

字符型：char

## 1.2 包装类型

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```



## 1.3 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```



# 二、String

## 概览

String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）

在 Java 8 中，String 内部使用 char 数组存储数据。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

## String, StringBuffer and StringBuilder

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步



