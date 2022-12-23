# time wait

### server和client都可以主动关闭TCP连接（换句话说第一个FIN报文server和client都可以发）

> 本质上来说，服务端发送了ACK以后，状态变成了TIME_WAIT，会再等2MSL，才会将连接关闭

### 四次挥手 (主动挥手的server)状态流转 

1. ESTABLISH
2. FIN_WAIT1
3. FIN_WAIT2
4. TIME_WAIT


### 在服务端收到客户端第二次的FIN报文并ACK以后，服务端的状态就是TIME_WAIT

### TIME_WAIT状态的Server会收到SYNC（合法 or 非法）或者RST

1. SYNC  如果合法相当于进入3次握手重新连接，如果非法将会再次发送四次挥手的最后一个ACK

> 处于TIME_WAIT重用该四元组连接，跳过 2MSL 而转变为 SYN_RECV 状态，接着就能进行建立连接过程

2. RST  处于TIME_WAIT的server收到RST报文默认会释放连接，但是可以通过设置让server丢弃RST报文

### 2MSL 是什么

### 相关博客

[在 TIME_WAIT 状态的 TCP 连接，收到 SYN 后会发生什么](https://xiaolincoding.com/network/3_tcp/time_wait_recv_syn.html#%E5%85%88%E8%AF%B4%E7%BB%93%E8%AE%BA)

[如果 TIME_WAIT 状态持续时间过短或者没有，会有什么问题？](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247502380&idx=1&sn=7b82818a5fb6f1127d17f0ded550c4bd&scene=21#wechat_redirect)



