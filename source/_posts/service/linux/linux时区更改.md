---
title : linux时区更改
description: linux时区更改
date: 2017-10-20 06:49:50
comments: false
---

### 时区更改
```bash
#查看当前时区
date -R
#修改设置Linux服务器时区
tzselect
#复制相应的时区文件，替换系统时区文件；或者创建链接文件
cp /usr/share/zoneinfo/$主时区/$次时区 /etc/localtime
#例如：在设置中国时区使用亚洲/上海（+8）
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```