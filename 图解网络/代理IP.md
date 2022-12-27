# 代理ip

1. HTTP请求一定能获取client的真实ip吗 （可以获取tcp连接的地址remote address）
2. Golang如何实现代理IP (httputils)
3. 代理IP和VPN是什么关系 (vpn是正向代理)
4. golang的http.Get底层是如何拨号socket的，accpet的时候如何获取tcp连接的信息


### X-Forwarded-For | RFC 7239
```
如果一个 HTTP 请求到达服务器之前，经过了三个代理 Proxy1、Proxy2、Proxy3，IP 分别为 IP1、IP2、IP3
用户真实 IP 为 IP0，那么按照 XFF 标准，服务端最终会收到以下信息

X-Forwarded-For: IP0, IP1, IP2
```

> req.Header.Get("X-Forwarded-For") // capitalisation


### Remote Address

```
HTTP 连接基于 TCP 连接，HTTP 协议中没有 IP 的概念
Remote Address 来自 TCP 连接，表示与服务端建立 TCP 连接的设备 IP
Remote Address 无法伪造，因为建立 TCP 连接需要三次握手，如果伪造了源 IP，无法建立 TCP 连接

直接对外提供服务的 Web 应用，在进行与安全有关的操作时，只能通过 Remote Address 获取 IP，不能相信任何请求头

备注: php 获取 Remote Address 是 $_SERVER["REMOTE_ADDR"]
```

### 正向代理和反向代理如何区分(本质上都是转发请求)

1. 正向代理其实是客户端的代理（一个是帮助客户端做事），反向代理则是服务器的代理（帮服务器做事）
2. 正向代理一般是客户端架设的，反向代理一般是服务器架设的

```
VPN 正向代理
nginx 反向代理
```
> httputil.ReverseProxy


### 相关博客

[终于有人把正向代理和反向代理解释的明明白白了 - 总结的比较易懂](https://cloud.tencent.com/developer/article/1418457)
[GO语言讲解正向代理和反向代理](https://cizixs.com/2017/03/21/http-proxy-and-golang-implementation/)
[HTTP 请求头中的 X-Forwarded-For 很详细](https://juejin.cn/post/6998047016668364837)
[HTTP 代理原理及实现](https://juejin.cn/post/6998351770871152653)
[golang从http.Request获取客户端ip的正确方法](https://www.imooc.com/wenda/detail/639125)
