# 集群一主多从搭建k8s

### 搭建步骤
#### docker拉镜像
docker pull mysql:5.7

### 主库master节点
```
mkdir /Users/xuweiqiang/Documents/mysql
mkdir /Users/xuweiqiang/Documents/mysql/conf
```

### 启动服务
```
$ docker run -d \
--name=mysql-master \
--privileged=true \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /Users/xuweiqiang/Documents/mysql/conf:/etc/mysql/conf.d \
-v /etc/localtime:/etc/localtime mysql:8.0
```
```
touch /Users/xuweiqiang/Documents/mysql/conf/my.cnf
```
# 设置conf打开binlog集群模式
```
[mysqld]
# 设置 server_id 唯一
server-id=1
# 开启二进制日志功能 
log-bin=mysql-master-bin-log
# relay_log 配置中继日志
relay_log=edu-mysql-relay-bin
```

# 查看master状态
```
mysql -uroot -p
show master status
```
# 添加slave访问用户
```
docker ps -a
docker exec -it [容器ID] /bin/bash
```
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



### 从库
```
$ docker run --restart=always \
-d --name=mysql --privileged=true -p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-v /Users/xuweiqiang/Documents/mysql/my.cnf:/etc/mysql/my.cnf \
-v /Users/xuweiqiang/Documents/mysql/logs:/var/log/mysql \
-v /Users/xuweiqiang/Documents//mysql/data:/var/lib/mysql \
-v /etc/localtime:/etc/localtime mysql
```

### 从库配置
```
$ cat /usr/local/mysql/my.cnf 
[mysqld]
server-id=2
# 开启二进制日志功能
log-bin=mysql-bin
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
