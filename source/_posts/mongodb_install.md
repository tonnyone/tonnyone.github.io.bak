---
title: mongodb3.X 单机安装与配置
category: 数据库 
tag: mongdb  
date: 2018-05-05 10:40:12
---

![](/uploads/mongodb.png)
[官网生产环境安装mongodb注意事项说明,装前必看!](https://docs.mongodb.com/manual/administration/production-notes/ "官网生产环境安装mongodb注意事项说明")

## [下载安装包](https://www.mongodb.com/download-center?jmp=docs&_ga=1.201102574.1776437196.1479373354#community "下载地址")

企业版和社区版本区别
我用的是社区版通用64位安装包(至于直接解压安装,还是用包管理器安装看个人习惯,大多数人应该是解压安装方便管理)

<!--more-->

## 安装

### 解压文件

我的在(/usr/local/mongodb)

### 新建conf目录添加配置文件 mongo.conf

**注意**: 此时配置项authorization是disable的

1. 新建启动脚本 startmq.sh(如下:)

```bash
#!/bin/bash
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
echo 0 | sudo tee /proc/sys/vm/zone_reclaim_mode
numactl --interleave=all /home/eversec/mongodb/bin/mongod --config ../conf/mongod.conf
```

- 启动前: 禁用大内存页[官方详细解释](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/ "官方详细解释")
- 以numacl启动
- 启动成功添加管理用户

这里我们先添加一个超级管理员 role为root
[mongodb内建角色](https://docs.mongodb.com/v3.2/reference/built-in-roles/ "mongodb内建角色")
[给用户赋权限命令](https://docs.mongodb.com/manual/reference/method/db.grantRolesToUser/#db.grantRolesToUser "给用户赋权限")

```javascript
use admin
db.createUser({
  user:"root",
  pwd:"root123456",
  roles:[{
      role:"root",
      db:"admin"
  }]
});
```

### shutdown数据库

我们用kill -2 杀掉进程
**注意:** 官方特意说明千万不要kill -9 损坏数据
[官网mongodb 进程说明](https://docs.mongodb.com/v3.4/tutorial/manage-mongodb-processes/ "mongodb 进程说明")

### 修改开启鉴权(authorization)为enable重启

- 修改conf文件mongo.conf 里面
- 修改authorization为enabled
- 执行startmq.sh
- 登录mongodb

```bash
./mongo -u root -p root123456 --host 192.168.243.140/admin
```

- 新建数据库,添加用户

```javascript
use testDb
db.createUser({
  user:"test",
  pwd:"test123456",
  roles:[{
      role:"readWrite",
      db:"testDb"
  }]
});
```

## 说明

[mongodb配置文件配置项官网说明](https://docs.mongodb.com/manual/reference/configuration-options/ "mongodb配置文件")

## 关于企业版和社区版的区别

1. 官网给出的解释:

- **In-memory Storage Engine**
> 高吞吐量,低延迟
- **Encrypted Storage Engine**
> 数据加密
- **Advanced Security**
> 使用LDAP和Kerberos访问控制，全面的审计功能
- 除了Ops Manager，Compass和BI连接器之外，还提供对MongoDB和最佳SLA的最全面的支持

[stackoverflow关于社区版和企业版](http://stackoverflow.com/questions/26527603/mongodb-opensource-vs-mongodb-enterprise "stackoverflow")