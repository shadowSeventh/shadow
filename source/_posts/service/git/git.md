---
title : git部分技巧
description: git部分技巧
date: 2017-10-21 06:49:50
comments: false
---

### 统计某人在该仓库提交的代码行数
```bash
git log --author="jingjiu" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
```
### 统计该仓库增加代码总行数
```bash
git log --stat|perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/'
```