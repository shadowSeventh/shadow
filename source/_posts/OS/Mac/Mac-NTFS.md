---
title: Mac对NTFS进行读写操作
date: 2017-12-20 10:34:22
categories: PC
tags: [OS,Mac]
description: Mac对NTFS进行读写操作
---

```bash
sudo -s #切换root用户
```

```bash
cd /sbin #进入/sbin文件夹
```
```bash
mv mount_ntfs mount_ntfs_orig #重命名mount_ntfs为mount_ntfs_orig
```
【这一步会出现问题：mv: rename mount_ntfs to mount_ntfs_orig: Operation not permitted. google了一下，找到解决方法，见后面】
```bash
vim mount_ntfs #（新建）编辑mount_ntfs脚本文件，内容为：

#!/bin/sh

/sbin/mount_ntfs_orig -o rw,nobrowse "$@"

```
```bash
chmod a+x mount_ntfs #修改权限增加可执行
```
mv: rename mount_ntfs to mount_ntfs_orig: Operation not permitted. 问题的解决，参照：[osx - Operation Not Permitted when on root El capitan (rootless disabled)
](https://stackoverflow.com/questions/32659348/operation-not-permitted-when-on-root-el-capitan-rootless-disabled)

具体如下：

1）重启mac，cmd+R进入恢复（recovery）模式
2）找到terminal(在“XX工具”里面）
3）输入：
```bash
csrutil disable
```
重启。

当然，改好后建议重新进入恢复（recovery）模式
```bash
$csrutil enable
```
要恢复对NTFS的不进行写功能
```bash
mv mount_ntfs_orig mount_ntfs #重命名mount_ntfs_orig为mount_ntfs
```
即可
