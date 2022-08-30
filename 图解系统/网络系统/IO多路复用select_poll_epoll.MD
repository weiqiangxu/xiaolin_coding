# IO多路复用 select/poll/epoll

> 时分多路复用

### 背景

```
实现基于TCP的socket

server.bind && listen 
client.connect && server.accept

client.read && server.write
server.read && client.write

服务端要给多个客户端提供服务 (建立一个socket连接就多了一个文件描述符)

多进程 && 多线程 都是消耗太大
```

### 单个进程单个线程就可以服务多个client

```
如果每个请求耗时 1 毫秒以内，那么 1 秒内可以处理上千个请求，把时间拉长来看，多个请求复用了一个进程

这就是多路复用，这种思想很类似一个 CPU 并发多个进程，所以也叫做时分多路复用

(并发不是并行)
```

### select实现原理

```
将所有 已连接的 Socket 放 1个文件描述符集合（内核）

遍历这个集合

标记可读可写，然后拷贝回用户态

再遍历用户态的可写可读的进行处理

2次遍历
2次拷贝

时间复杂度为 O(n)
并发上来以后时间复杂度指数级别增长
```

### epoll

```
红黑树跟踪待检测文件描述字
```

### epoll 支持2种触发模式，边缘触发（edge-triggered，ET）和水平触发（level-triggered，LT）

```
边缘 - 每2s通知
水平 - 有事件不停通知
```