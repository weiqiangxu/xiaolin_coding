# MVCC(multi-version-concurrent-control)

[MVCC CSDN博客](https://blog.csdn.net/SIESTA030/article/details/123113437)

> 多版本并发控制，实现对数据库的并发访问

[腾讯云cloud博客](https://cloud.tencent.com/developer/article/1450773)

### MVCC是基于乐观锁理论的

> 为了解决不可重复读 innodb 采用了 MVCC

### 具体逻辑
```
MVCC给每条数据后面添加隐藏的两列(创建版本号和删除版本号) 每个事务开始时候都有一个递增的版本号

create_version \ delete_version
```

### MVCC 创建版本和删除版本都在事务提交后产生

### 为啥MVCC能解决不可重复读

```
有了版本号之后

假定当前事务的版本号是 current_version，那么
create_version <=  current_version  <  delete_version
就可以避免查询到其他事务修改的数据
```

### MVCC手段只适用于读已提交（Read committed）和可重复读（Repeatable Read）
> 串行化没有行的版本控制问题 、 读未提交有些数据事务还没提交自然没有创建和删除版本

https://cloud.tencent.com/developer/article/1450773

### 其实你可以认为innodb通过MVCC实现快照读