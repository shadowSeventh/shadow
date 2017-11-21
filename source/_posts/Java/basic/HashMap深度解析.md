---
title: HashMap深度解析
date: 2017-11-21 10:33:22
categories: Java
tags: [Java]
description: HashMap可以说是Java中最常用的集合类框架之一，是Java语言中非常典型的数据结构，我们总会在不经意间用到它，很大程度上方便了我们日常开发。那么，HashMap和HashTable有什么区别？HashMap为什么是线程不安全的？HashMap还有哪些不为人知的特点？
---

## HashMap认知
了解HashMap之前，我们先看看Object类的两个方法hashCode和equals。

详情参考 [深入equals方法](http://blog.seventh.party/2017/11/21/Java/basic/%E6%B7%B1%E5%85%A5equals%E6%96%B9%E6%B3%95/)

众所周知，HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做Entry。这些个键值对（Entry）分散存储在一个数组当中，这个数组就是HashMap的主干。

HashMap数组每一个元素的初始值都是Null。

![avatar](http://ozqzyzixv.bkt.clouddn.com/640.webp)

对于HashMap，我们最常使用的是两个方法：Get 和 Put。

### Put方法的原理
调用Put方法的时候发生了什么呢？

比如调用 hashMap.put("apple", 0) ，插入一个Key为“apple"的元素。这时候我们需要利用一个哈希函数来确定Entry的插入位置（index）：
**index =  Hash（“apple”）**

假定最后计算出的index是2，那么结果如下：

![avatar](http://ozqzyzixv.bkt.clouddn.com/641.webp)

但是，因为HashMap的长度是有限的，当插入的Entry越来越多时，再完美的Hash函数也难免会出现index冲突的情况。比如下面这样：

![avatar](http://ozqzyzixv.bkt.clouddn.com/642.webp)

这时候该怎么办呢？我们可以利用链表来解决。

HashMap数组的每一个元素不止是一个Entry对象，也是一个链表的头节点。每一个Entry对象通过Next指针指向它的下一个Entry节点。当新来的Entry映射到冲突的数组位置时，只需要插入到对应的链表即可：

![avatar](http://ozqzyzixv.bkt.clouddn.com/644.webp)

需要注意的是，新来的Entry节点插入链表时，使用的是“头插法”。至于为什么不插入链表尾部，后面会有解释。

### Get方法的原理


使用Get方法根据Key来查找Value的时候，发生了什么呢？

首先会把输入的Key做一次Hash映射，得到对应的index：

index =  Hash（“apple”）

由于刚才所说的Hash冲突，同一个位置有可能匹配到多个Entry，这时候就需要顺着对应链表的头节点，一个一个向下来查找。假设我们要查找的Key是“apple”：
![avatar](http://ozqzyzixv.bkt.clouddn.com/645.webp)

第一步，我们查看的是头节点Entry6，Entry6的Key是banana，显然不是我们要找的结果。

第二步，我们查看的是Next节点Entry1，Entry1的Key是apple，正是我们要找的结果。

之所以把Entry6放在头节点，是因为HashMap的发明者认为，后插入的Entry被查找的可能性更大。


## HashMap初始长度问题

来看看HashMap默认初始长度是多少？为什么这么规定？

首先第一个问题，我们可以在源码中找到答案：
```java
/**
    * The default initial capacity - MUST be a power of two.
    */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
在初始化给出的初始长度是16，并且规定了每次扩展或者手动初始化时，长度必须是2的幂

而为什么这么做在查阅了一些资料后，整理出原因如下：

为了实现高效率的hash运算，HashMap采用了位运算的方式。公式（Length是HashMap的长度）如下：

**index =  HashCode（Key） &  （Length - 1）**

长度16或者其他2的幂，Length-1的值是所有二进制位全为1，这种情况下，index的结果等同于HashCode后几位的值。只要输入的HashCode本身分布均匀，Hash算法的结果就是均匀的。

## HashMap线程不安全(hash碰撞与扩容导致)
### hash碰撞

如上面所说，HashMap底层是一个Entry数组，一旦发生Hash冲突的的时候，HashMap采用拉链法解决碰撞冲突，而，Entry内部使用的是链表。

如果多个线程，在某一时刻同时操作HashMap并执行put操作，而有大于两个key的hash值相同，这个时候需要解决碰撞冲突，最终的结果可想而知，两个数据中势必会有一个会丢失

看看put()方法
```java
public V put(K key, V value) {  
    // 处理key为null，HashMap允许key和value为null  
    if (key == null)  
        return putForNullKey(value);  
    // 得到key的哈希码  
    int hash = hash(key);  
    // 通过哈希码计算出bucketIndex  
    int i = indexFor(hash, table.length);  
    // 取出bucketIndex位置上的元素，并循环单链表，判断key是否已存在  
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {  
        Object k;  
        // 哈希码相同并且对象相同时  
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {  
            // 新值替换旧值，并返回旧值  
            V oldValue = e.value;  
            e.value = value;  
            e.recordAccess(this);  
            return oldValue;  
        }  
    }  
  
    // key不存在时，加入新元素  
    modCount++;  
    addEntry(hash, key, value, i);  
    return null;  
}  
```
put方法不是同步的，同时调用了addEntry方法：
```java
void addEntry(int i, Object obj, Object obj1, int j)
    {
        if(size >= threshold && null != table[j])
        {
            resize(2 * table.length);
            i = null == obj ? 0 : hash(obj);
            j = indexFor(i, table.length);
        }
        createEntry(i, obj, obj1, j);
    }
```
addEntry方法依然不是同步的，所以导致了线程不安全问题

### 扩容
HashMap存在扩容的情况，对应的方法为HashMap中的resize方法：
```java
void resize(int i)
    {
        Entry aentry[] = table;
        int j = aentry.length;
        if(j == 1073741824)
        {
            threshold = 2147483647;
            return;
        } else
        {
            Entry aentry1[] = new Entry[i];
            transfer(aentry1, initHashSeedAsNeeded(i));
            table = aentry1;
            threshold = (int)Math.min((float)i * loadFactor, 1.073742E+009F);
            return;
        }
    }
```
可以看到扩容方法也不是同步的，通过代码我们知道在扩容过程中，会新生成一个新的容量的数组，然后对原数组的所有键值对重新进行计算和写入新的数组，之后指向新生成的数组。

当多个线程同时检测到总数量超过门限值的时候就会同时调用resize操作，各自生成新的数组并rehash后赋给该map底层的数组table，结果最终只有最后一个线程生成的新数组被赋给table变量，其他线程的均会丢失。而且当某些线程已经完成赋值而其他线程刚开始的时候，就会用已经被赋值的table作为原始数组，这样也会有问题。

### 多线程下使用HashMap
多线程下使用HashMap，几种方案：

* 在外部包装HashMap，实现同步机制
* 使用Map m = Collections.synchronizedMap(new HashMap(...));，这里就是对HashMap做了一次包装
* 使用java.util.HashTable，效率最低
* 使用java.util.concurrent.ConcurrentHashMap，相对安全，效率较高

## HashMap和HashTable

### HashMap和Hashtable的区别
HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步(synchronization)，以及速度。

* HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
* HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。
* 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
* 由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
* HashMap不能保证随着时间的推移Map中的元素次序是不变的。


参考资料：
[HashMap为什么线程不安全](http://blog.csdn.net/a_lele123/article/details/47660869)
[HashMap深度解析(一)](http://blog.csdn.net/ghsau/article/details/16843543/)
[HashMap深度解析(二)](http://blog.csdn.net/ghsau/article/details/16890151)
[HashMap和Hashtable的区别](http://www.importnew.com/7010.html)

