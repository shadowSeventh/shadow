---
title: Linux日常使用心得
date: 2018-01-27 10:34:22
categories: PC
tags: [OS,Linux]
description: linux日常使用心得
---

### 动态添加swap空间

```bash
dd if=/dev/zero of=/newdisk.swp bs=1M count=2048
```

bs默认是以字节为单位，count在大小就是以bs的值为基准的，还可以写成bs=1k count=64，大小可以自己写

```bash
mkswap /newdisk.swp
#创建swap空间,创建后会生成UUID，后面会用到
```

```bash
swapon /newdisk.swp
#启动swap空间
```

```bash
vim /etc/fstab
#系统启动自动加载swap分区的配置文件，UUID即上面生成的ID

UUID=83bb3f71-7310-4a28-826a-d807688aae2c swap swap defaults 0 0 
```

附：
```bash
/dev/zero       #伪设备，往固定大小里写0
```
dd可以创建一个固定（指定）大小的文件
```bash
dd if=/dev/sda of=/dev/sdb          #实现硬盘对考，if是input file，of是output file
#不可以用copy，copy是不能拷贝mbr
dd if=/dev/zero of=/var/swap/file.swp bs=1024 count=65536   #如果不指定大小，它会一直写，直到硬盘空间被占满
```