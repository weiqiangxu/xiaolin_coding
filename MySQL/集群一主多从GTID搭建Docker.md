# GTID的基本概念和组成（Global Transaction ID）

```
GTID (Global Transaction ID) 是对于一个已提交事务的编号，并且是一个全局唯一的编号。 GTID实际上是由server_uuid+transaction_id组成的。
```

### 搭建步骤

### 1.创建容器内网络
```
docker network create mysql_net
docker network ls
```

### 2.配置主库的数据库配置
```
mkdir -p /Users/xuweiqiang/Documents/mysql/master/
touch /Users/xuweiqiang/Documents/mysql/master/my.cnf
vim /Users/xuweiqiang/Documents/mysql/master/my.cnf
```
```
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql
pid-file=/var/run/mysqld/mysqld.pid
# 以下是自定义的配置
server-id=1
log-bin=mysql-bin
sync-binlog=1
expire-logs-days=7
relay_log=edu-mysql-relay-bin
binlog-ignore-db=information_schema
binlog-ignore-db=performation_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
gtid_mode=on
enforce-gtid-consistency=true
# 自定义配置到这里结束
[client]
socket=/var/run/mysqld/mysqld.sock
!includedir /etc/mysql/conf.d/
```

### 3.启动master服务
```
docker run -d \
--name=mysql-master \
--privileged=true \
-p 3306:3306 \
--network mysql_net \
--network-alias master \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /Users/xuweiqiang/Documents/mysql/master/my.cnf:/etc/mysql/my.cnf \
-v /etc/localtime:/etc/localtime mysql:8.0
```

### 4.查看master状态获取File和Position
```
docker exec -it mysql-master /bin/bash
mysql -uroot -p
show master status;
```
### 5.主库添加访问用户admin
```
# 创建同步用户
CREATE USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 分配用户权限（全部权限）
GRANT ALL PRIVILEGES ON *.* TO 'admin' @'%';
# 刷新权限
FLUSH PRIVILEGES;
# 查询用户权限
SHOW GRANTS FOR 'admin'@'%';
# 删除用户
DROP USER 'admin'@'%';
```

<hr/>


### 1.设置从库的数据库配置
```
mkdir -p /Users/xuweiqiang/Documents/mysql/slave
touch /Users/xuweiqiang/Documents/mysql/slave/my.cnf
vim /Users/xuweiqiang/Documents/mysql/slave/my.cnf
```
```
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql
pid-file=/var/run/mysqld/mysqld.pid
# 以下是自定义的配置
server-id=2
log-bin=mysql-bin-log
gtid_mode=on
enforce-gtid-consistency=true
binlog-format = ROW
sync-binlog=1
expire-logs-days=7
relay_log=mysql-relay-bin
# 忽略同步的数据库
binlog-ignore-db=information_schema
binlog-ignore-db=performation_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
# 自定义配置到这里结束
[client]
socket=/var/run/mysqld/mysqld.sock
!includedir /etc/mysql/conf.d/
```

### 2.启动slave服务
```
docker run -d \
--name=mysql-slave \
--privileged=true \
--network mysql_net \
--network-alias slave \
-p 3308:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /Users/xuweiqiang/Documents/mysql/slave/my.cnf:/etc/mysql/my.cnf \
-v /etc/localtime:/etc/localtime mysql:8.0
```
```
docker exec -it mysql-slave /bin/bash
mysql -uroot -p
```

### 3. 设置从库连接主库
```
CHANGE MASTER TO master_host = 'master',
master_user = 'admin',
master_password = '123456',
master_port = 3306,
master_auto_position = 1;
```
### 4.开启主从复制
```
start slave;
```

### 5.查看从库状态
```
show slave status;
# 成功的标志是
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

### 参考指令

```
# 查看所有用户
use mysql;
select user,HOST from user;
```

### GTID
```
previous_gtids:用于表示上一个binlog最后一个gtid的位置，每个binlog只有一个
```