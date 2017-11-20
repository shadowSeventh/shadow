---
title: hexo+gitPage+travis-ci自动部署
date: 2017-11-20 15:33:22
tags:
description: 之前整理笔记一直在自己的leanote私服上放着，后来发现笔记越来越多，也很乱！于是，狠下决心花些时间把笔记好好整理一下。所以，这次打算放gitPage上，travis-ci做持续构建,做到本地markdown文件提交到git后，自动部署到gitPage上，简单省事！而且Hexo提供了很多主题模板，也就不重复造轮子了！
---

## Hexo 填坑之路
### 准备
安装nvm
```bash
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```
安装node
```bash
nvm install v8.1.2
```
在此处遇到一个小问题，nvm安装node之后，`node -v`一直找不到node，后来`nvm ls`看了一下，当前node版本没有指定所以需要`nvm use v8.1.2`手动指定一下

安装hexo脚手架
```bash
npm install -g hexo-cli
```
创建Hexo工程
```bash
hexo init <folder>
cd <folder>
npm install

```
### Hexo相关配置

