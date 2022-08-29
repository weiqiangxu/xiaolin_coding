# Redis常见数据类型

### simple dynamic string 简单动态字符串
```
# 结构
struct{ len, buf[], alloc, flags }

# 为啥不用C语言字符串实现
1.长度获取
2.有len用于杜绝缓冲区溢出
3.减少内存重新分配 (空间预分配、惰性空间释放)
4.C字符串以空字符串判定是否结束，对于一些有空字符串的数据就炸了，C字符串只能保存文本数据
```


### 跳表 ZshipList

