---
title: HashMap深度解析
date: 2017-11-21 10:33:22
categories: Java
tags: [Java]
description: HashMap可以说是Java中最常用的集合类框架之一，是Java语言中非常典型的数据结构，我们总会在不经意间用到它，很大程度上方便了我们日常开发。那么，HashMap和HashTable有什么区别？HashMap为什么是线程不安全的？HashMap还有哪些不为人知的特点？
---

## HashMap认知
了解HashMap之前，我们先看看Object类的两个方法hashCode和equals。

详情参考* [深入equals方法](http://git-scm.com/book/zh)

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


## HashMap更多补充

来看看HashMap默认初始长度是多少？为什么这么规定？

首先第一个问题，我们可以在源码中找到答案：
```java
/**
    * The default initial capacity - MUST be a power of two.
    */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
在初始化给出的初始长度是16，并且规定了每次扩展或者手动初始化时，长度必须是2的幂