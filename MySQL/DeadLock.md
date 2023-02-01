# 死锁

### 而间隙锁与间隙锁之间是兼容的  两个事务都持有了(1006,+∞]的next key lock
### 而间隙锁与间隙锁之间是兼容的
### 而间隙锁与间隙锁之间是兼容的

### 而插入意向锁与间隙锁是冲突的

### 两个事务都想获取 插入意向锁

### 即两个事务可以同时持有包含共同间隙范围的间隙锁，并不存在互斥关系
### 即两个事务可以同时持有包含共同间隙范围的间隙锁，并不存在互斥关系

### 插入意向锁 因为 (插入位置是否已被其他事务加了间隙锁) 导致阻塞 ，此时又有多个事务获取了意向锁，导致互相阻塞 dead lock

# dead lock 的问题的关键

> 间隙锁的意义只在于阻止区间被插入，因此是可以共存的。


### 简单描述 - 加锁的顺序导致的
```
事务A获取行记录1的锁
事务B获取行记录2的锁
事务A此刻想要获取行记录2的锁
事务A此刻进入锁等待
事务B此刻也想要获取行记录1的锁
事务B此刻也进入锁等待
...
```

### Insert 语句是怎么加行级锁的



### 我自己的总结

> 两个事务因为互相持有对方正在等待的锁而导致双方都无法获取到锁，永久阻塞


### 情景描述 

```
user表记录user_name和余额user_money

用户A和用户B各有100块钱
```

### 并发
```
1. 用户A要给B转账50块钱
2. 用户B要给A转账30块钱
```

### MySQL获取互斥锁语句

```
# 事务1
set autocommit=0;
START TRANSACTION;
# 获取A的余额并获取记录A的互斥锁
SELECT user_id,@A_balance:=balance from account where user_id = 'A' for UPDATE;
```


```
# 事务2
set autocommit=0;
START TRANSACTION;
# 获取B的余额并获取记录B的互斥锁
SELECT user_id,@B_balance:=balance from account where user_id = 'B' for UPDATE;
```

此刻事务1和2都正常加锁，继续执行

<hr/>

```
# 事务1
# 获取B的余额并获取B的互斥锁
SELECT user_id,@B_balance:=balance from account where user_id = 'B' for UPDATE;

# 修改A 的余额
UPDATE account set balance = @A_balance - 50 where user_id = 'A';
# 修改B 的余额
UPDATE account set balance = @B_balance + 50 where user_id = 'B';
COMMIT;
```

```
# 事务2
# 获A的余额并获取A的互斥锁
SELECT user_id,@B_balance:=balance from account where user_id = 'A' for UPDATE;

# 修改B 的余额
UPDATE account set balance = @A_balance - 30 where user_id = 'B';
# 修改A 的余额
UPDATE account set balance = @B_balance + 30 where user_id = 'A';
COMMIT;
```

### 死锁的表现
```
此刻在事务2无法获取到行记录A的互斥锁，抛出的信息是

Lock wait timeout exceeded!
```

### 理论上如何解决死锁问题

> 保证锁A和锁B不要被两个事务各持有一部分而是每个事务同时持有2个锁，当两个事务获取锁的次序都是A然后B自然就不会有死锁的情况


### 如何避免
```
事务粒度小一点
低级别事务隔离机制

事务等待锁的超时时间 innodb_lock_wait_timeout 默认是50s
主动死锁检测 innodb_deadlock_detect 检测到了以后主动回滚死锁链条之中的某一个事务
```
