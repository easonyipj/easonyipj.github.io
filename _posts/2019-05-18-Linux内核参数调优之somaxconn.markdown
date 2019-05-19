---
layout: post
title:  "2019-05-18-记一次Linux内核参数debug"
date:   2019-05-18 16:45:31 +0530
categories: Linux 
author: "easonyi"
---
### 背景
> 在一次dev项目发布中，Python服务一直启动不起来，奇怪的是，发布日志无报错。起初以为是自己的分支有问题，于是采用线上稳定部署的master分支再在dev机器部署一次，Python服务还是无法正常启动。这时检查发布记录，发现dev机器上原来的分支是一个很老的分支，落后了master二十多次提交。这时借助git diff，发现后来的分支中uWSIG启动命令中增加了 -- listen 1024 参数，又登录机器检查uWSIG的输出日志，发现报错 Listen queue size is greater than the system max net.core.somaxconn (128).

### 错误原因
uWSIG启动命令中的 -- listen参数作用是设置socket的监听队列大小（默认：100）。
每一个socket都有一个相关联的队列，请求会被放入其中等待进程来处理。当这个队列满的时候，新来的请求就会被拒绝。
队列大小的最大值依赖于系统内核。启动脚本中uWSIG的启动命令设置的listening参数为1024，而dev机器上设置的somaxconn参数，只有128。

### 监听队列
对于一个TCP连接，Server与Client需要通过三次握手来建立网络连接。
当三次握手成功后，我们可以看到端口的状态由LISTEN转变为ESTABLISHED，接着这条链路上就可以开始传送数据了，
后端的应用程序只从已完成的连接的队列中获取请求。每一个处于监听(Listen)状态的端口，都有自己的监听队列。
监听队列的长度，与如下两方面有关:
- Linux系统内核的somaxconn参数
- 使用该端口的程序中listen函数，也就是uWSIG设置的listen参数

```shell
localhost:~ yipingjian$ netstat
Active Internet connections
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4      24      0  localhost.53489        123.125.132.35.https   CLOSE_WAIT
tcp4      24      0  localhost.53488        123.125.132.35.https   CLOSE_WAIT
tcp4       0      0  localhost.53487        61.135.169.125.https   ESTABLISHED
tcp4       0      0  localhost.53486        61.135.169.125.https   ESTABLISHED
tcp4       0      0  localhost.53485        61.135.169.125.https   ESTABLISHED
tcp4       0      0  localhost.53477        17.252.156.24.5223     ESTABLISHED
```

### somaxconn参数
somaxconn定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为128.限制了每个端口接收新tcp连接侦听队列的大小。对于一个经常处理新连接的高负载 web服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多。 服务进程会自己限制侦听队列的大小(例如 sendmail(8) 或者 Apache)，常常在它们的配置文件中有设置队列大小的选项。大的侦听队列对防止拒绝服务 DoS 攻击也会有所帮助。
而对于 Linux 而言，基本上任意语言实现的通信框架或服务器程序在构造 socket server 时，都提供了 backlog 这个参数，
因为在监听端口时，都会调用系统底层 API: int listen(int sockfd, int backlog)。
listen 函数中 backlog 参数的定义如下：
> Now it specifies the queue length for completely established sockets waiting to be accepted,
instead of the number of incomplete connection requests. 
The maximum length of the queue for incomplete sockets can be set using the tcp_max_syn_backlog sysctl. 
When syncookies are enabled there is no logical maximum length and this sysctl setting is ignored.
If the socket is of type AF_INET, and the backlog argument is greater than the constant SOMAXCONN(128 default), 
it is silently truncated to SOMAXCONN.

所以，当传入的backlog值大于somaxconn值时，便会报错。虽然没有看uWSIG的源码，但是可以猜测，listen参数传入的值被最终当作backlog参数传入socket()函数。设置somaxconn参数相关命令：

```shell
# 显示内核所有参数
sysctl -a

# 修改somaxconn参数：
# vim打开/etc/sysctl.conf
vim /etc/sysctl.conf
# 尾部追加net.core.somaxconn=32768
# 生效：
sysctl -p
```





