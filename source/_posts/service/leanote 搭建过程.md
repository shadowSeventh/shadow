---
title: leanote 搭建过程
categories: 服务器
description: 久闻leanote大名，今天入坑试一试
---
## 安装 golang

```bash
wget http://www.golangtc.com/static/go/1.6/go1.6.linux-amd64.tar.gz
cd /home/leanote
tar -xzvf go1.6.linux-amd64.tar.gz
mkdir /home/user1/gopackage #这里面会放go的包和编译后的文件
mkdir /home/leanote/gopackage
```
    
配置环境变量, 编辑/etc/profile文件：
```bash
sudo vim /etc/profile
export GOROOT=/home/leanote/go
export GOPATH=/home/leanote/gopackage
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
source /etc/profile
```
    
查看go是否安装成功:
```bash
go version
```
## 获取Revel和 Leanote 的源码

请下载 leante-all-master.zip。解压后，将src文件夹复制到 /home/leanote/gopackage/使用如下命令生成revel二进制命令, 稍后运行Leanote需要用到：

```bash
go install github.com/revel/cmd/revel
```
    
## 安装Mongodb
### 安装Mongodb
```bash
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.1.tgz
cd /home/leanote
tar -xzvf mongodb-linux-x86_64-3.0.1.tgz/
sudo vim /etc/profile
export PATH=$PATH:/home/leanote/mongodb-linux-x86_64-3.0.1/bin
source /etc/profile
```
    
### 测试Mongodb安装
```bash
mkdir /home/leanote/data
mongod --dbpath /home/leanote/data
```
    
```
# 首先切换到leanote数据库下
> use leanote;
# 添加一个用户root, 密码是abc123
> db.createUser({
    user: 'root',
    pwd: '19930317.tong',
    roles: [{role: 'dbOwner', db: 'leanote'}]
});
# 测试下是否正确
> db.auth("root", "19930317.tong");
1 # 返回1表示正确
```
    
添加到服务中：

```
[Unit]
Description=mongodb
After=network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
PIDFile=/home/leanote/data/mongod.lock
ExecStart=/home/leanote/mongodb-linux-x86_64-3.0.1/bin/mongod --dbpath=/home/leanote/data --logpath=/home/leanote/log/mongodb.log --logappend --fork
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```


## 导入初始数据
```bash
mongorestore -h localhost -d leanote --dir /home/leanote/gopackage/src/github.com/leanote/leanote/mongodb_backup/leanote_install_data
```
    
## 配置Leanote

```bash
vi conf/app.conf
```
    
```
http.port=80
site.url=http://a.com
app.secret=XXXXXXXXXXXXXXXX
db.host=localhost
db.port=27017
db.dbname=leanote # required
db.username=root # if not exists, please leave blank
db.password=abc123 # if not exists, please leave blank
```
    
配置开机启动：
```
#!/bin/bash
    nohup mongod --dbpath /root/mongodb/data --auth 2>&1 &
    nohup revel run github.com/leanote/leanote 2>&1 &
    sstr=$(echo -e $str)
    echo "$sstr"
    
    vi /etc/rc.local
    
    source /etc/profile
    /root/gopackage/run.sh
```

## 运行Leanote

```
revel run github.com/leanote/leanote
```