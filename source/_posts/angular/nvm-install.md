---
title: nvm-install
date: 2017-12-18 10:34:22
categories: 框架
tags: [angular,nvm]
description: Mac下安装Node并切换taobao.org源
---

## Mac下安装Node并切换taobao.org源

### 安装 nvm

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```
或
```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```
载入环境变量
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```

### 安装Node

```bash
nvm install 8.2.1
```
### 设置淘宝镜像源

```bash
npm config set registry https://registry.npm.taobao.org
```
