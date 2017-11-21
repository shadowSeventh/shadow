---
title: 深入equals方法
date: 2017-11-21 10:33:22
categories: Java
tags: [Java]
description: 在java编程或者面试中经常会遇到 == 、equals()的比较。自己看了看源码，结合自己的理解记录一下。
---

## equals
先来看一下的默认实现：
```java
/** 默认同==，直接比较对象 */  
public boolean equals(Object obj) {  
    return (this == obj);  
}  
```
默认的equals方法，直接调用==，比较对象地址。String类中重写了equals方法，比较的是字符串值，看一下源码实现：
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
String类中的equals首先比较地址，如果是同一个对象的引用，可知对象相等，返回true。

若果不是同一个对象，equals方法挨个比较两个字符串对象内的字符，只有完全相等才返回true，否则返回false。

## hashcode

hashCode是根类Obeject中的方法。

默认情况下，Object中的hashCode() 返回对象的32位jvm内存地址。也就是说如果对象不重写该方法，则返回相应对象的32为JVM内存地址。

String类源码中重写的hashCode方法如下:
```java
public int hashCode() {
    int h = hash;    //Default to 0 ### String类中的私有变量，
    if (h == 0 && value.length > 0) {    //private final char value[]; ### Sting类中保存的字符串内容的的数组
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
String源码中使用private final char value[];保存字符串内容，因此String是不可变的。

## 总结

### 绑定。
当equals方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。
### 绑定原因。
Hashtable实现一个哈希表，为了成功地在哈希表中存储和检索对象，用作键的对象必须实现 hashCode 方法和 equals 方法。同(1)，必须保证equals相等的对象，hashCode 也相等。因为哈希表通过hashCode检索对象。
### 默认
    * ==默认比较对象在JVM中的地址。

    * hashCode 默认返回对象在JVM中的存储地址。

    * equal比较对象，默认也是比较对象在JVM中的地址，同==
