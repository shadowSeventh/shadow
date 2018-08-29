---
title: linux一些好用的命令
date: 2017-12-20 10:34:22
categories: DB
tags: [DB,Mongo]
description: mongo 基础操作
---

db.auth("siteRootAdmin", "siteRootAdmin");

use qhMoreShop_prod

db.createUser( {
    user: "qhMoreShop_prod",
    pwd: "qhMoreShop_prod",
    roles: [ { role: "dbOwner", db: "qhMoreShop_prod" } ]
});