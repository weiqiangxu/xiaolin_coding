# 四次挥手

### 2MSL
```
两倍的MSL

MSL是Maximum Segment Lifetime英文的缩写，中文可以译为“报文最大生存时间”
```

### 主动关闭方状态
1. established
2. fin_wait_1
3. fin_wait_2
4. time_wait
5. close

### 被动的关闭方状态
1. established
2. close_wait
3. last_ack
4. close


### 假设客户端作为主动关闭
```
client 说不再发送数据，但是还能接受数据
server 说不再接收数据，但是数据还没发完
...
server 说数据发送完了，不再发送数据了
client 说收到了进入close_wait状态
server 接收到进入close
```


### 总结就是一句话，关闭之前确认双方都不会再发送数据，围绕这个数据，最后才会做关闭连接的动作


### 一些思考

1. 第一次挥手丢失
2. 第二次挥手丢失
3. 第三次挥手丢失
4. 第四次挥手丢失


### 解答

1. client第一次FIN时候，没收到server的ACK，会怎么样
```
client的状态还是会变得，发送完就变，由established 变成了 fin_wait_1

收不到会

超时重传机制，重发次数tcp_orphan_retries控制

一直重传都收不到

client 会直接断开连接
```

### ACK 报文是不会重传的
### ACK 报文是不会重传的
### ACK 报文是不会重传的

> 所以如果第一次挥手FIN报文丢失和第二次挥手ACK报文丢失，都会重发第一次FIN报文

> FIN_WAIT2 最大持续时长 tcp_fin_timeout 默认值是60s

### 主动关闭的client如果60s都收不到server的FIN报文，就会直接关闭

### 被动关闭的发送第二次挥手ACK之后，进入CLOSE_WAIT状态
```
内核是没有权利替代进程关闭连接,必须由进程主动调用 close 函数来触发服务端发送 FIN 报文

换句话说

CLOSE WAIT 状态的server，如果要主动关闭连接(发送FIN报文) 必须进程主动调用 close 函数

当server发送FIN之后，如果一直收不到client的ACK也会重试，达到一定次数后，服务端会断开连接
```

### 第四次挥手ACK一直丢失也会导致第三次的FIN报文一直重发

### 而第四次挥手的client，则等到2MSL之后关闭连接

> 等待过程中收到server的FIN，那么会重新发送ACK和开始等待2MSL



### 为什么 TIME_WAIT 等待的时间是 2MSL？

> MSL 是 Maximum Segment Lifetime，报文最大生存时间

> 如果被动关闭方没有收到断开连接的最后的 ACK 报文，就会触发超时重发 FIN 报文，另一方接收到 FIN 后，会重发 ACK 给被动关闭方， 一来一去正好 2 个 MSL

### 为什么需要 TIME_WAIT 状态

> 保证「被动关闭连接」的一方，能被正确的关闭

### TIME_WAIT 过多有什么危害？

1. 占用系统资源 （内存、cpu、线程等）
2. 占用端口资源 net.ipv4.ip_local_port_range,一般32768～61000

```
client的time_wait过多，因为端口资源占满无法对[目的IP和端口]一样的发起连接 

server（主动发起关闭连接方）的 TIME_WAIT 状态过多, TCP 连接过多，会占用系统资源，比如文件描述符、内存资源、CPU 资源、线程资源等
```


### 如何优化 TIME_WAIT

1. 设置client调用connect()函数的时候，内核随机找一个time_wait超过1s的连接给新的连接复用
2. 设置time_wait的最大bucket值，超过就将后面的time_wait连接状态重置
3. 设置调用close就直接发送RST给对端(比较暴力)

> 如果服务端要避免过多的 TIME_WAIT 状态的连接，就永远不要主动断开连接，让客户端去断开，由客户端去承受 TIME_WAIT


### 服务器出现大量 TIME_WAIT 状态的原因有哪些

1. http没有使用长连接 Connection: Keep-Alive （默认开启），但是 header中有Connection:close就无法使用长连接

> 大多数 Web 服务的实现，不管哪一方禁用了 HTTP Keep-Alive，都是由服务端主动关闭连接 ; 客户端禁用或者服务端禁用http的keep-alive 最后都是服务端主动关闭TCP连接；


2. HTTP长连接超时，客户端在完成一个http请求后，60s内没有发起新的请求，nginx就会出发回调函数关闭连接 ，此时会有time_wait连接（比如nginx的keepalive_timeout）

3. HTTP 长连接的请求数量达到上限（比如nginx的keepalive_requests），每个 HTTP 长连接最多只能跑 100 次请求超过了就主动断开


### 服务器出现大量 CLOSE_WAIT 状态的原因有哪些

> close wait属于被动关闭方（收到第一次的FIN报文然后ACK后，状态就是close wait，在这个过程之中把没发完的数据发送完成）
的大量close wait的时候说明服务端没有调用close函数关闭连接

### tcp服务端流程-代码层面

1. create socket && bind port && listen
2. server.socket register epoll
3. epoll_wait...
4. while epoll_wat then accept
5. client.socket register epoll
6. client.close
7. server.close (conn.Close)

> server执行close函数之前，代码卡在某一个逻辑，比如发生死锁等等，也就是第7个是最常见的 (conn.Close没执行)

[客户端的端口可以重复使用吗](https://xiaolincoding.com/network/3_tcp/port.html#%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E7%AB%AF%E5%8F%A3%E5%8F%AF%E4%BB%A5%E9%87%8D%E5%A4%8D%E4%BD%BF%E7%94%A8%E5%90%97)


