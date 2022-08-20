# 集群一主多从搭建k8s

### 搭建步骤
#### docker拉镜像
docker pull mysql:5.7

### 主库配置挂载宿主机
```
mkdir -p /Users/xuweiqiang/Documents/mysql/master/conf
touch /Users/xuweiqiang/Documents/mysql/master/conf/my.conf
vim /Users/xuweiqiang/Documents/mysql/master/conf/my.conf
```
```
[mysqld]
# 设置 server_id 唯一
server-id=1
# 开启二进制日志功能 
log-bin=mysql-master-bin-log
# relay_log 配置中继日志
relay_log=edu-mysql-relay-bin
```

### 启动master服务
```
$ docker run -d \
--name=mysql-master \
--privileged=true \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /Users/xuweiqiang/Documents/mysql/master/conf:/etc/mysql/conf.d \
-v /etc/localtime:/etc/localtime mysql:8.0
```

# 查看master状态
```
docker exec -it mysql-master /bin/bash
mysql -uroot -p
show master status
```
# 添加访问用户
```
mysql -uroot -p
```
```
# 创建同步用户
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
# 分配用户权限（全部权限）
GRANT ALL PRIVILEGES ON *.* TO 'slave' @'%';
# 也可也分配增删改查权限和 slave 权限
GRANT SELECT, DELETE,UPDATE,INSERT,REPLICATION SLAVE, REPLICATION CLIENT,REPLICATION_SLAVE_ADMIN ON *.* TO 'slave' @'%';
# 授予复制账号 REPLICATION CLIENT 权限，复制用户可以使用 SHOW MASTER STATUS, SHOW SLAVE STATUS 和 SHOW BINARY LOGS 来确定复制状态。
# 授予复制账号 REPLICATION SLAVE 权限，复制才能真正地工作。
-------------------------------------------------------------------------------------------------------------
# 刷新权限
FLUSH PRIVILEGES;
# 查询用户权限
SHOW GRANTS FOR 'slave'@'%';
-------------------------------------------------------------------------------------------------------------
# 删除用户
DROP USER 'slave'@'%';
```



### 从库配置挂载宿主机
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
```

### 启动slave服务
```
$ docker run -d \
--name=mysql-slave \
--privileged=true \
-p 3308:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /Users/xuweiqiang/Documents/mysql/slave/conf:/etc/mysql/conf.d \
-v /etc/localtime:/etc/localtime mysql:8.0
```
```
docker exec -it mysql-slave /bin/bash
mysql -uroot -p
show slave status;
```

### 服务重启
```
$ docker restart 容器ID
```

### 查询所有用户
```
use mysql
select user,HOST from user;
```

### 参考博客

(基于docker搭建的)[https://www.mobaijun.com/posts/572796845.html]
