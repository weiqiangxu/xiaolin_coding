# MySQL索引背后的数据结构及算法原理

[原文](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

> 原文的数据库索引图示画的很棒

### B-Tree和B+Tree是什么

### MySQL索引实现

1. MyISAM索引结构(主键和辅助索引都是叶子节点data域存储数据记录的地址)，非聚集，索引文件和数据文件是分离的

2. Innodb的表数据文件本身是按B+Tree的一个索引结构，索引就是数据，主键索引里面data域就是完整的数据记录，辅助索引的data域记录的是主键值而不是地址，这也是为什么innodb一定有主键而MyISAM可以没有

### 索引使用策略及优化

1. 主键选择，尽量用自增而非身份证号之类的，因为索引维护是顺序的，自增不断新增数据页就可以，而身份证号则插入容易导致页分裂（当前页的数据需要拷贝到其他页）

2. MySQL的查询优化器会自动调整where子句的条件顺序以使用适合的索引

3. 最左前缀匹配（联合索引）

4. WHERE emp_no - 1='10000'会弃用索引，MySQL还没有智能到自动优化常量表达式的程度

5. 索引选择性与前缀索引，选择性（Selectivity），是指不重复的索引值（也叫基数，Cardinality）与表记录数（#T）的比值

### 联合索引的数据结构图示

[点击](https://blog.csdn.net/cristianoxm/article/details/107818084)

### Join表和子查询哪个更快

```
1. 子查询需要临时表,Join更快
2. Join表在分库分表时候效率低
```

### Join表原理

[图例很不错哦](https://cloud.tencent.com/developer/article/2007170)

```
# 左连接
1. 双层for循环，将左边每一行记录都去右表查询一次，Simple Nested-Loop Join简单嵌套循环连接，左表n条右表m条(这里的行数指的是explain的rows)总次数是n*m
2. 右表使用索引，Index Nested-Loop Join 索引嵌套循环
3. Block Nested-Loop Join缓存循环嵌套查询

# 得出结论
1. 驱动表的选择最好小表（取决于join buffer大小）
2. 被驱动表的关联字段需要索引
3. 增大 join buffer 的大小
4. 联表查询时候如果字段过多限制需要的地段不要直接用*获取所有字段
```

