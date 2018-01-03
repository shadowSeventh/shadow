---
title: Scrapy踩过的那些坑
date: 2017-12-18 10:34:22
categories: Python
tags: [Python,Scrapy]
description: Mac下 Scrapy踩过的那些坑
---

```bash
-bash: Scrapy: command not found
```
因为按照时安装在了用户目录下`pip install Scrapy --user` ，不知道为什么pip没有自动导入环境变量导致的
```bash
sudo pip install Scrapy
```
即可

```bash
lit@litdeMacBook-Pro github$ scrapy startproject fang_scrapy
Traceback (most recent call last):
  File "/usr/local/bin/scrapy", line 7, in <module>
    from scrapy.cmdline import execute
  File "/Library/Python/2.7/site-packages/scrapy/cmdline.py", line 9, in <module>
    from scrapy.crawler import CrawlerProcess
  File "/Library/Python/2.7/site-packages/scrapy/crawler.py", line 7, in <module>
    from twisted.internet import reactor, defer
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/reactor.py", line 38, in <module>
    from twisted.internet import default
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/default.py", line 56, in <module>
    install = _getInstallFunction(platform)
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/default.py", line 50, in _getInstallFunction
    from twisted.internet.selectreactor import install
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/selectreactor.py", line 18, in <module>
    from twisted.internet import posixbase
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/posixbase.py", line 18, in <module>
    from twisted.internet import error, udp, tcp
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/tcp.py", line 28, in <module>
    from twisted.internet._newtls import (
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/_newtls.py", line 21, in <module>
    from twisted.protocols.tls import TLSMemoryBIOFactory, TLSMemoryBIOProtocol
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/protocols/tls.py", line 63, in <module>
    from twisted.internet._sslverify import _setAcceptableProtocols
  File "/Users/lit/Library/Python/2.7/lib/python/site-packages/twisted/internet/_sslverify.py", line 38, in <module>
    TLSVersion.TLSv1_1: SSL.OP_NO_TLSv1_1,
AttributeError: 'module' object has no attribute 'OP_NO_TLSv1_1'
```
该错误的原因是twisted的版本和scrapy要求的版本不匹配
```bash
pip uninstall twisted
sudo pip install twisted==13.1.0
```
即可