---
title: 深入equals方法
date: 2017-11-21 10:33:22
categories: Java
tags: [Java]
description: Java中String创建原理深入分析
---
### 基本概念
1、  使用new关键字 
String s1 = new String(“ab”);  //
2、  使用字符串常量直接赋值
String s2 = “abc”;
3、  使用”+”运算符进行字符串连接
String s3 = “abc” + “d”;
String s4 = s3 + 5;  //abcd5
**常量池概念：**
Java运行时会维护一个String Pool（String池）， 也叫“字符串缓冲区”。String池用来存放运行时中产生的各种字符串，并且池中的字符串的内容不重复。而一般对象不存在这个缓冲池，并且创建的对象仅仅存在于方法的堆栈区。

### 原理
* 原理1：当使用任何方式来创建一个字符串对象s时，Java运行时（运行中JVM）会拿着这个s在String池中找是否存在内容相同的字符串对象，如果不存在，则在池中创建一个字符串s，否则，不在池中添加。
* 原理2：Java中，只要使用new关键字来创建对象，则一定会（在堆区或栈区）创建一个新的对象。
* 原理3：使用直接指定或者使用纯字符串串联来创建String对象，则仅仅会检查维护String池中的字符串，池中没有就在池中创建一个，有则罢了！但绝不会在堆栈区再去创建该String对象。
* 原理4：使用包含变量的表达式来创建String对象，则不仅会检查维护String池，而且还会在堆栈区创建一个String对象。最后指向堆内存中的对象

### 栗子
```java
public class StringTest {
    publicstaticvoid main(String args[]) {
       // 在池中和堆中分别创建String对象"abc",s1指向堆中对象
       String s1 = new String("abc");
       // s2直接指向池中对象"abc"
       String s2 = "abc";
       // 在堆中新创建"abc"对象，s3指向该对象
       String s3 = new String("abc");
       // 在池中创建对象"ab"  和 "c"，并且s4指向池中对象"abc"
       String s4 = "ab" + "c";
       // c指向池中对象"c"
       String c = "c";
       // 在堆中创建新的对象"abc"，并且s5指向该对象
       String s5 = "ab" + c;
 
       System.out.println("------------实串-----------");
       System.out.println(s1 == s2); // false
       System.out.println(s1 == s3); // false
       System.out.println(s2 == s3); // false
       System.out.println(s2 == s4); // true
       System.out.println(s2 == s5); // false
    }
}
```