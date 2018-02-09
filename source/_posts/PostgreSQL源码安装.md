title: PostgreSQL源码安装
author: Walleipt.Wang
tags:
  - PostgreSQL
categories:
  - PostgreSQL
date: 2018-02-09 09:53:00
---
### 1. 下载源码

```
wget https://ftp.postgresql.org/pub/source/v9.6.2/postgresql-9.6.2.tar.gz
```

### 2. 配置编译安装

```
A.解压:
tar -zxvf ./postgresql-9.5.5.tar.gz

B.设置安装目录：
./configure --prefix=/usr/local/postgresql

C.编译源码：
make

D.安装：
make install
```


### 3. 用户权限与环境变量
```
A.添加用户(postgreSQL不允许使用root身份操作)：
useradd postgres

B.配置目录访问权限：
chown -R postgres:postgres /usr/local/postgresql/

C.环境变量(主要为了方便操作postgreSQL命令)：
PGHOME=/usr/local/postgresql
export PGHOME
PGDATA=/usr/local/postgresql/data
export PGDATA

PATH=$PATH:$HOME/bin:$HOME/.local/bin:$PGHOME/bin

export PATH

D.测试环境变量：
which psql
psql -V
```

### 4. 初始化数据库
```
A.初始化数据库：
initdb

B.配置信任的ip:
vi /usr/local/postgresql/data/pg_hba.conf

C.修改IP4信任地址：
127.0.0.1/32修改为：0.0.0.0/0


D.修改postgresql.conf文件
listen_addresses='*'
port = 5432


```

**data目录介绍：**
- base:目录是表空间目录
- global:目录是相关全局变量的目录
- pg_hba.conf:访问控制配置（127.0.0.1改为信任的客户端ip网段使其可以远程访问）
- postgresql.conf:一个是postgresql主配置文件（listen_address=localhost改为星号使其监听整个网络）

### 5. 启动和连接
```
A.创建日志目录：
mkdir -v /usr/local/postgresql/log

B.启动数据：
pg_ctl start -l /usr/local/postgresql/log/pg_server.log

C.连接数据库:
psql

D.设置postgres的密码：
\password

E.查看数据列表:
\l


```

### 额外补充
在调试postgreSQL还需要一些源码提供的额外工具；例如oid2name和pg_buffercache等可查找到相关源码然后安装参考如下命令:
```
搜索源码关键字
grep pg_buffercache

进入源码目录
cd contrib/pg_buffercache

编译&&安装
make
make install
```