# List

### 用做消息队列需要满足的点
1. 消息保序  （LPUSH & RPOP & 阻塞式读取减少CPU损耗的BRPOP）
2. 重复消费   （手动生成全局唯一ID比如 LPUSH mq "111000102:stock:99"）
3. 消息可靠性  （BRPOPLPUSH 读取后存入备份List之中）

> 缺点是只支持 1 个消费者，没有消费者组的概念

> Stream 数据类型了，Stream 同样能够满足消息队列的三大需求，而且它还支持「消费组」形式的消息读取


# Hash

> Hash 类型的 （key，field， value） 主要用于存储对象

### 数据结构：压缩列表、哈希表、listpack 是什么来的

# BitMap 位图

>  String 类型作为底层数据结构， 统计二值状态比如是否登陆的数据类型

```
SETBIT key offset value
GETBIT key offset

# ID 100 的用户在 2022 年 6 月份 3 号已签到
SETBIT uid:sign:100:202206 2 1

# 检查该用户 6 月 3 日是否签到
GETBIT uid:sign:100:202206 2

# 统计该用户在 6 月份的签到次数
BITCOUNT uid:sign:100:202206
```

# HyperLogLog 网页 UV 计数

# GEO 存储地理位置信息
```
# ID 号为 33 的车辆的当前经纬度位置存入
GEOADD cars:locations 116.034579 39.030452 33

#  5公里内的车辆信息
GEORADIUS cars:locations 116.054579 39.030452 5 km ASC COUNT 10
```

# Stream 专门为消息队列设计的数据类型