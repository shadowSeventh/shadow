---
title: HashMap深度解析
date: 2017-11-21 10:33:22
categories: Java
tags: [Java]
description: HashMap可以说是Java中最常用的集合类框架之一，是Java语言中非常典型的数据结构，我们总会在不经意间用到它，很大程度上方便了我们日常开发。那么，HashMap和HashTable有什么区别？HashMap为什么是线程不安全的？HashMap还有哪些不为人知的特点？
---

Object类的两个方法hashCode和equals。先来看一下这两个方法的默认实现：
```java
/** JNI，调用底层其它语言实现 */  
public native int hashCode();  
  
/** 默认同==，直接比较对象 */  
public boolean equals(Object obj) {  
    return (this == obj);  
}  
```
equals方法我们太熟悉了，我们经常用于字符串比较，String类中重写了equals方法，比较的是字符串值，看一下源码实现：
```java
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```