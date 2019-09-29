[TOC]



# #面向对象特性

多态、继承、封装

**多态类型:**
* 编译时多态 方法重载
* 运行时多态 运行时才可以确定引用指向的具体类型



# #数据类型

## ##8种基本数据类型

* 整型:4种. 
  | 类型  | size [bit/byte] | 范围 |
  | :----- | :--------- | --- |
  | short | 16/2          |   $[-2^{15}, 2^{15}-1] $   |
  | int | 32/4 | $[-2^{31}, 2^{31}-1] $ |
  | long | 64/8 | $[-2^{63}, 2^{63}-1]$ |
  | byte | 8/1 | $[-2^{7}, 2^{7}-1]$ |
  
* 浮点型:2种. 
  | 类型  | size [bit/byte] |
  | :----- | :--------- |
  | float | 32/4         |
  | double | 64/8 |
* 字符型:1种. 
  
  * char 16bits
  
* 布尔型:1种. 
  
  * boolean 1bits	

## ##包装/封装类型

8种基本类型都有对应的包装/封装类型

### ###自动装箱、拆箱

```java
Integer x = 2;     // 装箱 Integer.valueOf(2);
int y = x;         // 拆箱 x.intValue();
```



### ###常量池

Integer.valueOf(int i): Integer 常量池的大小默认为 **-128~127**，如果**i**在范围里将会返回常量池的数据，否则会新建一个对象。

``` java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

#### ####包装类型使用 ==  equals()情况

* ==
  * 当==两边的表达式只是引用，则对比的是引用的地址；
  * 当有其中一个带有运算，则对比的是两边的数值。
* equals
  * 判断类型
  * 判断数值

```java
		Integer a = 1;	//Integer.valueOf(1);
		Integer b = 2;	//Integer.valueOf(2);
		Integer c = 3;	//Integer.valueOf(3);
		Integer d = 3;	//Integer.valueOf(3);
		Integer e = 321;	//Integer.valueOf(321);
		Integer f = 321;	//Integer.valueOf(321);
		Long g = 3L;	//Long.valueOf(3L);
		Long h = 2L;	//Long.valueOf(3L);
		System.out.println(c == d);// true，缓存池 
		System.out.println(e == f);//false，不在缓存池 因为321不在缓存范围内
		System.out.println(c == (a + b));//true 
		System.out.println(c.equals(a + b));//true 
		System.out.println(g == (a + b));//true 
		System.out.println(g.equals(a + b));//false 类型不相同返回false 查看equals源码
		System.out.println(g.equals(a + h));//true 
```

#### ####包装类型的 equals 源码

**Long类型作为例子:**

```java
public boolean equals(Object obj) {
        if (obj instanceof Long) { // 判断类型
            return value == ((Long)obj).longValue(); //转换类型并判断值
        }
        return false;
}
```

## ##String

String 被final声明，不可被继承，并且实例对象是不可变的immutable

### ###immutable的好处

1. 缓存hash值

2. String Pool需要

   如果一个String对象被创建过，那就可以从String Pool中获得引用。
   **注意:不是所有String对象都会进入String Pool。**

3. 安全性

4. 线程安全性

   不可变性具备线程安全，可以在多线程中安全使用。

### ###String、StringBuilder、StringBuffer

1. 可变性
   String 不可变
   StringBuilder StringBuffer可变
2. 线程安全性
   String线程安全
   StringBuilder线程不安全
   StringBuffer线程安全，内部使用synchronized进行同步，因此性能跟StringBuilder比稍差

### ###String Pool

字符串常量池(String Pool) 保存:
    -**字符串字面量(literal strings)** 这些字符串在编译时就已经确定。**"abc": 这是一个字符串字面量**
    -**调用了inter()的String对象**

#### ####inter()

**当一个字符串调用inter():**

1. String Pool中已经存在一个相同的字符串(使用equals方法进行比较)，返回String Pool中字符串的引用地址。
2. 否则，将此字符串加入String Pool中，并返回引用地址。

#### ####new String("abc")

此语句会产生两个对象:

1. "abc": 字符串字面量，被存进String Pool
2. 新建一个String对象值为"abc"

**new String 源码:**
String 作为构造参数时，新String对象value直接指向旧对象的value，而不是复制旧对象的value。

``` java
public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
}
```

### ###jdk8和jdk9里的区别

Jdk 8中，使用char数组存储

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
}
```

Jdk 9中，使用byte数组存储，用变量coder确定编码

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    private final byte[] value;
    private final byte coder;
}
```

# #抽象类与接口
JDK1.8之后，接口可以拥有方法的实现，这个方法需要添加新的关键字default。

```java
interface C{
	default public String getName() {
		return "C";
	}
}
```



# #初始化顺序

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）



# #集合

## ##Collection

* List
  * ArrayList 
  * Vectory
  * LinkedList 可作Heap Stack Queue 
* Set
  * HashSet 无序 内部使用HashMap，查找**$$O(1)$$**
  * TreeSet 有序 内部使用TreeMap， 红黑树实现 查找**$$O(\log_2 N)$$**
  * LinkedHashSet 具有HashSet查找效率，使用链表维护插入顺序
* Queue
  * LinkedList
  * PriorityQueue 最小堆、最大堆
* Map
  * HashMap 线程不安全
  * HashTable 线程安全
  * TreeMap 红黑树

## ##List

### ###ArrayList

动态数组，初始默认数组size = 10
**扩容**
   (size+size/2)=size*1.5倍
**线程不安全**

### ###Vector

动态数组，初始默认数组size = 10，increasement默认为0
**扩容**
   [increasement>0]：size+increasement
   [increasement == 0]：size+size = 2*size
**线程安全**

## ##Set

### ###HashSet HashMap

初始默认Node<Key,Value>数组size=16，加载因子loadFactor=0.75，threshold=size*loadFactor
**扩容** 发生在每当size > threshold
	threshold * 2
	size * 2
**线程不安全**


## ##Tree

### ###结构

红黑树: 自平衡二叉树，有地方与234树对应
234树: 一种特殊的B树
B树/B-树: 多叉树，每个节点可以有多个子节点
B+树: B树的一种变形



# #多线程

**线程与进程的区别:**
	进程是CPU分配调度资源的独立单位
	一个进程拥有多个线程，这些线程之间共享所属进程的资源

**死锁产生的4个条件:**
	**资源条件互斥条件**：一个资源只能被一个线程占用
	**请求与保持条件**：一个线程因请求资源阻塞时(资源被其他线程占用)，									对已有的资源保持占用
	**不剥夺条件**：线程已有的资源不能被剥夺
	**循环等待条件**：线程之间循环等待资源

**线程不安全:**
	指多个线程同时操作一个数据，最后得到的数据是脏数据。

## ##volatile

### ###特性

#### ####内存可见性: 

线程A对volatile变量进行修改对其他线程来说是可见的，即其他线程读取volatile变量的值总是最新的。但这个变量依旧**线程不安全**。 

#### ####禁止重新排序:

编译器为了优化代码，进行代码执行顺序的重新排序。

#### ####线程不安全

#### ####非原子性

### ###使用场景:

1. 单例类中可用，为保证双重检查锁不失效。

下面代码是单例类的**double check locking 双重检查锁**，对变量添加volatile可以防止编译器重新排序导致线程A释放锁后，instance还是null，然后其他线程进入构造instance同步代码块内，再次新建对象。

```java
class Singleton {
    private volatile static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) { // 拿到锁
                if (instance == null) { //
                    instance = new Singleton();
                }
            }// 释放锁
        }
        return instance;
    } 
}
```



## ##ThreadLocal 线程局部变量

[ThreadLocal Link](https://www.cnblogs.com/dolphin0520/p/3920407.html)



# #事务的ACID

原子性：不可分割的工作单元。要么都发生，要么都不发生。
一致性：事务开始之前和之后，数据的完整性和逻辑性一致。
隔离性：多个事务同时发生，它们之间是隔离的。
持久性：事务完成把数据保存在数据库后不会被回滚