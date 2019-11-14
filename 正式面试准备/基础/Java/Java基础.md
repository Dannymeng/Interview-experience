


# 一、数据类型

## 1.1基本类型


| 数据类型   |  大小（B） | 位数  |  说明 |
| :---------:| :--------:|:----: |:-------:|
| byte   | 1 |8||
| char     | 2 | 16   |长度和编码有关，ASCII码中char占一个字节，UTF-8是不定长编码，编码的长度是动态的|
| short     | 2 | 16| 
| int | 4    |32|
|float|4|32|
|long|8|64|
|double|8|64|
|boolean|1| 8|理论上只占据一个bit，但是实际是占据了一个byte|

其中1B=8bit , 一个bit代表一个1或者0，是计算机的基本单位
![](https://github.com/Dannymeng/picture/blob/master/2018083015380793.png?raw=true)


**面试题： 如何实现byte和char转换？**
- char -> byte
 byte占8位，char占18位，1个char数据需要2个byte转换

```java
char a = 'a'; 
byte[] b = new byte[2]; 
b[0] = (byte) ((a & 0x00) >> 8); 
b[1] = (byte) ((a & 0xFF));
```

- byte -> char   上述方法相反方向
```java
byte a = 0; 
byte b = 0; 
char c = (char) ((a & 0xFF << 8) | (b & 0xFF));
```




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