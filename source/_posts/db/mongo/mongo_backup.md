---
title : 一个mongo备份的脚本
description: 一个mongo备份的脚本
date: 2017-10-21 06:49:50
tags: [mongo,shell]
comments: false
---
### 一个mongo备份的脚本

```bash
    #!/bin/sh                                     
    # ==================================================== 配置项
    
    # mongodump备份文件执行路径 
    #DUMP=/data0/soft/mongodb/mongodb-linux-x86_64-rhel70-3.0.5/bin/mongodump 
    DUMP=/data0/soft/mongodb/mongodb-linux-x86_64-rhel70-3.2.7/bin/mongodump 
    # 当前shell脚本所在的目录
    DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
    # 临时备份目录
    WORK_DIR=$DIR/work 
    # 备份存放路径
    BAK_DIR=$DIR/bak
    # 获取当前系统时间
    NOW=`date +%Y-%m-%d.%H:%M:%S` 
    NOW1=`date +%Y%m%d.%H%M%S` 
    TIME="date +%Y-%m-%d.%H:%M:%S"
    # 备份的是哪个数据库
    DB=agency 
    # 数据库账号
    DB_USER=agency
    # 数据库密码
    DB_PASS="agency"
    # 保存多少份数量
    COUNT=20
    # 最终保存的数据库备份文件名
    BAK_NAME="mongo_${DB}_$NOW1"
    
    # ==================================================== 代码区
    cd $DIR
    
    echo -e "\n\n\n======================================================"
    echo "`$TIME` mongodb 定时备份 ${DB} 开始"
    echo "`$TIME` ---- 当前目录是 : ${DIR}"
    echo "`$TIME` ---- 清空临时工作目录 : ${WORK_DIR}"
    rm -fr ${WORK_DIR}/ *
    [ $? = 0 ] || exit -1
    
    echo "`$TIME` ---- 创建备份文件目录 : $WORK_DIR/$BAK_NAME"
    mkdir -p $WORK_DIR/$BAK_NAME
    [ $? = 0 ] || exit -1
    
    echo "`$TIME` ---- 备份 mongodb 数据库: ${DB}"
    $DUMP -d $DB -u $DB_USER -p $DB_PASS -o $WORK_DIR/$BAK_NAME
    [ $? = 0 ] || exit -1
    
    echo "`$TIME` ---- 将数据文件打包: $BAK_DIR/${BAK_NAME}.tar.gz "
    tar -zcvf $BAK_DIR/${BAK_NAME}.tar.gz  $WORK_DIR/$BAK_NAME
    [ $? = 0 ] || exit -1
    
    echo "`$TIME` ---- 删除过期的备份文件（最多保留${COUNT}份)"
    dirList=(`find $BAK_DIR -maxdepth 1 -type f -name *.tar.gz|sort`)
    [ $? = 0 ] || exit -1
    
    # 下面的循环为闭区间。所以在长度在减去1，下标是从0开始的
    let loopEnd=${#dirList[@]}-$COUNT-1
    for i in `seq 0 $loopEnd`
    do
      echo "`$TIME` -------- 删除: ${dirList[i]}"
      rm -fr ${dirList[i]} 2>&1  || {
        exit $?
      }
    done
    [ $? = 0 ] || exit -1
    
    echo "`$TIME` mongo定时备份 ${DB} 结束"

```