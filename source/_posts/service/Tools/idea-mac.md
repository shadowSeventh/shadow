---
title : idea-mac
date: Mon May 21 2018 09:21:54 GMT+0800 (CST)
categories:  mac
tags: [idea]
description: idea内存配置最佳实践
---

## idea 内存配置最佳实践

### 默认（灰色标识）
JetBrains 提供的默认设置：

```
-Xms128m
-Xmx750m
-XX:MaxPermSize=350m
-XX:ReservedCodeCacheSize=240m
-XX:+UseCompressedOops
```

### Big（大）（红色标识）
给 Xmx 配 4096MB， ReservedCodeCacheSize 设置 1024MB，这已经是相当多的内存了：

```
-Xms1024m
-Xmx4096m
-XX:ReservedCodeCacheSize=1024m
-XX:+UseCompressedOops
```

### Balanced(平衡的)（蓝色标识）
Xmx 和 Xms 都分配 2GB ，这是相当平衡的内存消耗：

```
-Xms2g
-Xmx2g
-XX:ReservedCodeCacheSize=1024m
-XX:+UseCompressedOops
```

### Sophisticated（复杂的）（橘色标识）
和上面一样， Xmx 和 Xms 都分配2GB，但是给 GC 和内存管理指定不同的垃圾回收器和许多不同的标志：

-server
-Xms2g
-Xmx2g
-XX:NewRatio=3
-Xss16m
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled
-XX:ConcGCThreads=4
-XX:ReservedCodeCacheSize=240m
-XX:+AlwaysPreTouch
-XX:+TieredCompilation
-XX:+UseCompressedOops
-XX:SoftRefLRUPolicyMSPerMB=50
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djsse.enableSNIExtension=false
-ea