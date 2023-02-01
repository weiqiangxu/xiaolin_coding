# MySQL 一行记录是怎么存储的

1. MySQL 的 NULL 值会占用空间吗
2. MySQL 怎么知道 varchar(n) 实际占用数据的大小
3. varchar(n) 中 n 最大取值为多少
4. 行溢出后，MySQL 是怎么处理的

### MySQL 的数据存放在哪个文件

1. 数据文件存放在哪个目录呢
```
SHOW VARIABLES LIKE 'datadir'; // 假设是 /var/lib/mysql


# 创建一个 database（数据库） 都会在 /var/lib/mysql/ 目录里面创建一个以 database 为名的目录
```

```
假设有一个数据库my_test里面只有一张表t_order

# 具体的目录结构

[root@xiaolin ~]#ls /var/lib/mysql/my_test
db.opt         // 当前数据库的默认字符集和字符校验规则
t_order.frm    // t_order 的表结构
t_order.ibd    // t_order 的表数据  MySQL 5.6.6 版本开始每张表的数据都存放在一个独立的 .ibd 文件



表名字.ibd - 独占表空间文件
```

2. 表空间文件的结构是怎么样的

```
段（segment）、区（extent）、页（page）、行（row）


数据表之中的记录按“行”为单位存放的

Innodb的数据是按“页”为单位读写的，默认每个页的大小为 16KB （最多能保证 16KB 的连续存储空间）
```

### 区是干嘛的
```
表中数据很大的时候

按区为单位分配每个区1MB（64个16kb的页划分为一个区）

让链表（底层数据结构是B+）相邻的页物理存储位置也相邻（顺序IO）
```

### 段（segment）又是什么

1. 索引段：存放 B + 树的非叶子节点的区的集合
2. 数据段：存放 B + 树的叶子节点的区的集合
3. 回滚段：存放的是回滚数据的区的集合（MVCC 事务隔离相关）



# Innodb行格式有哪些

1. Redundant ( MySQL 5.0 版本之前用的 )
2. Compact (重点介绍)
3. Dynamic
4. Compressed

### COMPACT 行格式

1. 额外信息 (变长字段长度列表（字节数 比如 03 01 02）、null值列表(1表示null 0表示非null)、记录头信息) - 列表都是逆序的
2. 真实数据（row_id \ trx_id \ roll_ptr \ 列1、2等值）


### 记录头信息
1. delete_mask
2. next_record
3. record_type （0表示普通记录，1表示B+树非叶子节点记录，2表示最小记录，3表示最大记录）

### 当数据表的字段都定义成 NOT NULL 是没有null值列表的 (NULL 值列表占用字节可以扩展)

### row_id 建表的时候指定了主键或者唯一约束列，那么就没有 row_id 隐藏字段了

### trx_id 事务id，表示这个数据是由哪个事务生成的

### roll_pointer 记录上一个版本的指针


### varchar(n) 中 n 最大取值为多少
```
MySQL 规定 列（不包括隐藏列和记录头信息） 占用的字节长度加起来不能超过 65535 个字节

看数据库表的字符集，将字节数换成字符数就是n的取值
```

### 数据库表只有一个 varchar(n) 字段，字符集是 ascii 的情况下
```
varchar(n) 中 n 最大值 = 65535 - 2 - 1 = 65532
```


### 行溢出后，MySQL 是怎么处理的

```
MySQL中磁盘和内存交互的基本单位是页，一个页的大小一般是 16KB (16384字节)，一个 varchar(n) 类型的列（一个数据表字段的存储值）最多可以存储 65532字节


一个页可能就存不了一条记录 (TEXT类型数据)

这就叫行溢出，多的数据就会存到另外的「溢出页」中
```


