# 集群一主多从搭建k8s

### 搭建步骤

### 1.创建容器内网络
```
docker network create mysql_net
docker network ls
```

### 2.配置主库的数据库配置
```
mkdir -p /Users/xuweiqiang/Documents/mysql/master/conf
touch /Users/xuweiqiang/Documents/mysql/master/conf/my.conf
vim /Users/xuweiqiang/Documents/mysql/master/conf/my.conf
```
```
[mysqld]
# 服务器唯一ID
server-id=1

#指定binlog的存储位置，日志格式为二进制
log-bin=/var/lib/mysql/data/binlog/mysql-bin

#配置为1每次执行写入就与硬盘同步
#如果不配置默认会等到缓冲区满了自动刷盘
sync-binlog=1

#需要同步的二进制数据库名如果有多个就多行binlog-do-db
#binlog-do-db=user

#只保留7天的二进制日志防磁盘被日志占满
expire-logs-days=7

# 中继日志
relay_log=edu-mysql-relay-bin

# 忽略同步的数据库
binlog-ignore-db=information_schema
binlog-ignore-db=performation_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
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
-v /Users/xuweiqiang/Documents/mysql/master/conf:/etc/mysql/conf.d \
-v /etc/localtime:/etc/localtime mysql:8.0
```

# 4.查看master状态获取File和Position
```
docker exec -it mysql-master /bin/bash
mysql -uroot -p
show master status;
```
# 5.主库添加访问用户Slave
```
# 创建同步用户
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';

# 分配用户权限（全部权限）
GRANT ALL PRIVILEGES ON *.* TO 'slave' @'%';

# 也可也分配增删改查权限和 slave 权限
# 有些只配置replication slave,replication client权限
GRANT SELECT, DELETE,UPDATE,INSERT,REPLICATION SLAVE, REPLICATION CLIENT,REPLICATION_SLAVE_ADMIN ON *.* TO 'slave' @'%';

# 授予复制账号 REPLICATION CLIENT 权限，复制用户可以使用 SHOW MASTER STATUS, SHOW SLAVE STATUS 和 SHOW BINARY LOGS 来确定复制状态。
# 授予复制账号 REPLICATION SLAVE 权限，复制才能真正地工作。

# 刷新权限
FLUSH PRIVILEGES;
# 查询用户权限
SHOW GRANTS FOR 'slave'@'%';

# 删除用户
DROP USER 'slave'@'%';
```

<hr/>


### 1.设置从库的数据库配置
```
mkdir -p /Users/xuweiqiang/Documents/mysql/slave/conf
touch /Users/xuweiqiang/Documents/mysql/slave/conf/my.conf
vim /Users/xuweiqiang/Documents/mysql/slave/conf/my.conf
```
```
[mysqld]
# 设置 server_id 唯一
server-id=2

# 开启二进制日志功能 
log-bin=mysql-slave-bin-log

#[必须开启]打开中继日志且日志格式为二进制                  
relay_log = /var/lib/mysql/data/binlog/mysql-relay-bin

#如果salve库名称与master库名相同，使用本配置     
#replicate-do-db = user

#如果master库名[artisan]与salve库名[user01]不同，使用以下配置[需要做映射]     
#replicate-rewrite-db = artisan[主库名] -> user01[从库名]

#如果不是要全部同步[默认全部同步]，则指定需要同步的表   
#replicate-wild-do-table=user01.t_order    
#replicate-wild-do-table=user01.t_order_item

#设置只读权限 
read_only = 1  

#使得更新的数据写进二进制日志中       
log_slave_updates = 1 
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
-v /Users/xuweiqiang/Documents/mysql/slave/conf:/etc/mysql/conf.d \
-v /etc/localtime:/etc/localtime mysql:8.0
```
```
docker exec -it mysql-slave /bin/bash
mysql -uroot -p
```

### 3. 设置从库连接主库
```
CHANGE MASTER TO master_host = 'master',
master_user = 'slave',
master_password = '123456',
master_port = 3306,
master_log_file = 'binlog.000002',
master_log_pos = 157,
master_connect_retry = 30;
```
### 4.查看从库状态
```
show slave status;
```

### 5.查看线程状况
```
show processlist;
# 关注
# Slave_IO_Running: Yes //IO线程正常运行
# Slave_SQL_Running: Yes //SQL线程正常运行
```

### 参考所有指令

```
# 查看所有用户
use mysql;
select user,HOST from user;
```

### 参考博客

[基于docker搭建的](https://www.mobaijun.com/posts/572796845.html)
