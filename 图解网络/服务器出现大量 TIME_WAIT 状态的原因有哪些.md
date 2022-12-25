# 服务器出现大量 TIME_WAIT 状态的原因有哪些

1. 第一个场景：HTTP 没有使用长连接 (Connection: Keep-Alive or Connection:close)
2. 第二个场景：HTTP 长连接超时（注意不是请求响应超时，这里指的是tcp连接很久不再使用，比如nginx.keepalive_timeout 60s都没有新的请求就会关闭连接）
3. HTTP 长连接的请求数量达到上限(nginx.keepalive_requests) (比如请求次数超过100次就会关闭该tcp连接)


### 仅仅对于connection:close
```
根据大多数 Web 服务的实现，不管哪一方禁用了 HTTP Keep-Alive，都是由服务端主动关闭连接


注意：这并不意味着http之中主动关闭连接的都是server
注意：这并不意味着http之中主动关闭连接的都是server
注意：这并不意味着http之中主动关闭连接的都是server

仅仅是说在http设置connection:close的时候，关闭连接的主动方都是server
仅仅是说在http设置connection:close的时候，关闭连接的主动方都是server
仅仅是说在http设置connection:close的时候，关闭连接的主动方都是server
```

[实战抓包分析](https://xiaolincoding.com/network/3_tcp/tcp_tcpdump.html#%E8%A7%A3%E5%AF%86-tcp-%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B%E5%92%8C%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8B)

> 从抓包分析之中就可以看出，主动关闭连接的是client，第一次发出FIN报文的是client