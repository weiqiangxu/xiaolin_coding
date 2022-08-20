# 集群双主架构

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
log-bin=mysql-bin-log
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
# 查看节点状态
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
skip-grant-tables
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

### 3.创建用户admin给master访问
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

### 4.设置从库连接主库
```
CHANGE MASTER TO master_host = 'master',
master_user = 'admin',
master_password = '123456',
master_port = 3306,
master_log_file = 'mysql-bin-log.000003',
master_log_pos = 826,
master_connect_retry = 30;
```

### 5.开启主从复制
```
start slave;
```

### 6.查看从库状态
```
show slave status;
# 成功的标志是
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

<hr/>

### 1.主节点设置跟踪slave节点
```
CHANGE MASTER TO master_host = 'slave',
master_user = 'admin',
master_password = '123456',
master_port = 3306,
master_log_file = 'mysql-bin-log.000003',
master_log_pos = 1221,
master_connect_retry = 30;
```

### 2.查看主节点是否以开启slave角色
```
show slave status;
```

### 参考指令

```
# 查看所有用户
use mysql;
select user,HOST from user;

# mysql的数据data和日志log存放的地方
/var/lib/mysql/

# change master指令的 master_log_file 和 master_log_pos 需要登陆目标host机器查看
show master status;

# binlog格式行
binlog-format = ROW
# 混合模式
binlog_format=MIXED

```
