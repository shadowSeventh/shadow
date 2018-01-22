---
title: npm那些坑
date: 2018-01-19 10:34:22
categories: 前端
tags: [angular,nvm]
description: npm遇到那些坑
---

## windows环境下：
### 1. node-gyp需要python环境
错误原因：node-gyp需要python环境

报错信息如下：
![avatar](http://ozqzyzixv.bkt.clouddn.com/905DEFB5-7FF8-4829-B9F7-6639FF4FF0D8.png)

解决方案：

以管理员权限运行CMD：

```js
npm install --global --production windows-build-tools
```

参考资料：[node-gyp](https://github.com/nodejs/node-gyp#on-windows)

### 2. Unexpected end of input at 1:10971 se,"deprecated":"We\\'re really excited that you\\'re trying to use E

错误原因：未知，猜测可能是npm-cache影响

报错信息如下：
![avatar](http://ozqzyzixv.bkt.clouddn.com/36D8795D-C4B5-41CF-88DB-84280F675536.png)
解决方案：

清空`C:\Users\lit\AppData\Roaming\npm-cache`目录

### 3. 'webpack-dev-server' 不是内部或外部命令，也不是可运行的程序或批处理文件。

错误原因：未安装webpack-dev-server

解决方案：建议全局安装webpack-dev-server

```js
npm i webpack-dev-server -g --save
```




