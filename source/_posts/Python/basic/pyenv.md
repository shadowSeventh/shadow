---
title: nvm-install
date: 2017-12-18 10:34:22
categories: Python
tags: [Python]
description: Mac下 Pyenv 的安装使用
---

Mac自带的python2.7不能删除，因为很多系统模块依赖，所以使用pyenv来安装python3.5版本，这样就可以并存。

通过homebrew安装:
```bash
brew install pyenv
```
但是在安装openssl依赖时提示一下信息：
```bash
A CA file has been bootstrapped using certificates from the SystemRoots
keychain. To add additional certificates (e.g. the certificates added in
the System keychain), place .pem files in
  /usr/local/etc/openssl/certs

and run
  /usr/local/opt/openssl/bin/c_rehash

This formula is keg-only, which means it was not symlinked into /usr/local,
because Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries.

If you need to have this software first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile

For compilers to find this software you may need to set:
    LDFLAGS:  -L/usr/local/opt/openssl/lib
    CPPFLAGS: -I/usr/local/opt/openssl/include
For pkg-config to find this software you may need to set:
    PKG_CONFIG_PATH: /usr/local/opt/openssl/lib/pkgconfig
```
上面提示，如果编译器需要这些依赖文件，那么你需要在在编译之前执行lib和include的设置。所以需要导入下面配置
```bash
echo 'export LDFLAGS="-L/usr/local/opt/openssl/lib"' >> ~/.bash_profile
echo 'export CPPFLAGS="-I/usr/local/opt/openssl/include"' >> ~/.bash_profile
echo 'export PKG_CONFIG_PATH="/usr/local/opt/openssl/lib/pkgconfig"' >> ~/.bash_profile
```

下面是一些常用的命令
```bash
pyenv install --list //查看可安装的python版本
pyenv install 3.5.0 安装python3.5.0
pyenv rehash
pyenv versions //查看已经安装好的版本，带*号的为当前使用的版本
pyenv global 3.5.0 //设置全局版本，即系统使用的将是此版本
pyenv local 3.5.0 //当前目录下的使用版本，有点类似virtualenv
```
### 网络问题
如anaconda之类大容量的版本，由于网络的问题，总是连接中断，安装失败。此时可以先从官方网站下载安装包，然后放在~/.pyenv/cache文件夹中，然后在pyenv install 此版本，pyenv会自动先从此文件夹中搜索