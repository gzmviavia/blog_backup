---
title: 传输控制协议（4）保活定时器
date: 2016-08-13 14:27:53
tags:
- 计算机网络
- 运输层
- TCP
categories:
- 计算机网络
---


## 什么是保活计时器

TCP/IP协议允许一个TCP连接没有数据传输，也就是说我们可以启动一个客户端与服务器建立一个连接，然后过去数小时、数天、数个星期或者数月，该TCP仍然保持连接。只要两端的主机没有被重启，则连接依然保持建立。

<!-- more -->

那么可能出现这种情况，TCP连接的一方（假设是客户端）因为某种情况导致失去了连接，而TCP连接的另一端（假设是服务器端）又一直没有向客户端发送报文，那么服务器端就会一直认为这个连接是正常的，导致服务器端一直维持着这个连接，导致资源白白被浪费了。

所以必须要有一种机制来检查某个空闲的TCP连接是否是正常的，如果这个空闲的TCP连接是异常的，那么就将其关闭。TCP使用保活计时器来实现，默认保活计时器是关闭的，可以使用`setsockopt`函数来设置`socket`选项从而开启该功能。

保活计时器的默认时间是两个小时，可以在客户端开启保活功能，也可以在服务端开启保活功能，下面的说明默认是服务器端开启了保活功能。

如果一个TCP连接在两个小时内没有任何工作，那么服务器端就会向客户端发送一个探查报文，客户端肯定处于以下4个状态之一：
1. 客户端正常运行，并从服务器可达，说明该连接可用，重新设置其保活计时器。

2. 客户端已经崩溃，服务器端收不到对该探查报文的响应，并在75秒后超时，服务器端总共发送10个这样的探查，每个间隔75秒，如果服务器没有收到一个响应，那么就认为客户端已经关闭并终止连接。

3. 客户端崩溃并且重新启动了，客户端成功收到了该探查，但是客服端已经没有了该连接的任何信息，所以客户端认为这是一个异常的连接，所以会发送一个rst报文给服务器，使得服务器终止这个连接。

4. 客户端正常运行，但是从服务器端不可达。 这与状态2相同，因为TCP不能区分状态2和状态4的区别。

## 如何使用保活计时器

在Linux系统中，有三个跟TCP keepalive相关的参数：

```cpp
tcp_keepalive_intvl (integer; default: 75; since Linux 2.4)
       The number of seconds between TCP keep-alive probes.

tcp_keepalive_probes (integer; default: 9; since Linux 2.2)
       The  maximum  number  of  TCP  keep-alive  probes  to send before giving up and killing the connection if no
       response is obtained from the other end.

tcp_keepalive_time (integer; default: 7200; since Linux 2.2)
       The number of seconds a connection needs to be idle before TCP begins sending out keep-alive probes.   Keep-
       alives  are  sent only when the SO_KEEPALIVE socket option is enabled.  The default value is 7200 seconds (2
       hours).  An idle connection is terminated after approximately an additional 11 minutes (9 probes an interval
       of 75 seconds apart) when keep-alive is enabled.
```

<br/>
<br/>
参考：

+ [动手学习TCP：4种定时器](http://www.cnblogs.com/wilber2013/p/4854526.html)
+ [TCP的定时器系列 — 保活定时器](http://blog.csdn.net/zhangskd/article/details/44177475)
+ [TCP协议中的计时器](http://www.tuicool.com/articles/bQfmUv)
