---
title: 传输控制协议（1）连接与释放
date: 2016-08-10 15:38:47
tags:
- 运输层
- TCP
categories:
- 计算机网络
mathjax: true
---

## TCP首部字段

![TCP首部字段](TCP_header.png)

+ 16位源端口：用来实现TCP的分用，根据端口的不同，将数据上交给不同的上层应用。

+ 16位目的端口：同上。

+ 32位序号：对数据进行编号，每个字节的数据都要消耗一个的编号,编号的范围是[0, 2^32-1]，编号增加到2^32-1后，编号就会变成0重新开始。首部中序号表示该次传输的数据的第一个字节的编号，比如第一次传输数据传输了100个字节，那么这些数据的编号是[0，99]，所以首部中序号应该填写0；第二次传输数据传输了100个字节，那么这些数据的编号是[100,199]，所以首部中序号应该填写100。

+ 32位确认号：对收到的数据进行确认，也可以表示期望对方下一个报文段的第一个字节数据的序号。比如A第一次传输了100个字节的数据给B，那么B对其确认时，确认号为100，表明序号100以前的数据我都收到了，B期望收到A的下一个数据的序号是100。
> 如果确认号 = N，则表明到序号N - 1的所有数据已经正确收到。

+ 4位首部长度：4位二进制能够表示的最大数是15，而首部长度的单位是4个字节，所以首部长度最大值是60个字节。也就是说选项长度不能超过40个字节。

+ 6位保留：保留字段，现在还没有使用，所以应该置为0。确定有保留字段，扩展性就变强了。

+ 1位URG：

+ 1位ACK：当ACK = 1时，确认号字段才有效，当ACK = 0时，确认号无效。TCP规定，连接建立之后，所有传送的报文都必须把ACK置为1。

+ 1位PSH：

+ 1位RST：

+ 1位SYN：当SYN = 1时，表示该报文是与建立TCP连接有关的报文。

+ 1位FIN：当FIN = 1时，表示该报文是与释放TCP连接有关的报文。

+ 16位窗口大小：窗口字段指出了现在允许对方发送的数据量。窗口值经常在动态变化着。

+ 16位检验和：对伪首部 + TCP首部 + 数据部分进行16位反码求和，然后取反的结果填写在该字段。接收方对收到的数据同样进行伪首部 + TCP首部 + 数据部分的16伪反码求和，如果最后结果的每一位都是1表面数据在传输的过程中没有出现错误。

+ 16位紧急指针：

+ 选项：
## TCP连接的建立

> how are you ?		&nbsp;&nbsp;&nbsp;&nbsp;(SYN)
> fine.And you?		&nbsp;&nbsp;&nbsp;&nbsp;(SYN + ACK)
> Fine.				&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(ACK)

![三次握手流程图](TCP_three_shake_hands.png)

1. 客户发送一个报文给服务器，该报文的SYN = 1，seq = x。表明这是连接请求的报文。虽然不携带数据，但是要消耗一个序号。

2. 服务器收到该报文之后，发现这是一个连接请求的报文，发送一个确认报文给客服，该报文的SYN = 1，ACK = 1，seq = y，ack\_num = x + 1。表明这是一个对连接请求的确认报文。虽然不携带数据，但是要消耗一个序号。

3. 客户收到服务器的确认报文之后，也要发送一个报文给服务器。该报文的的ACK = 1，seq = x + 1，ack\_num = y + 1。也就你认为上面两步已经够了，但是这一步是很有必要的。只有一个ACK字段置为1，所以这是一个纯粹的确认包，TCP规定，如果确认包不携带数据，那么不消耗序号。


<br/>
**关于为什么要三次握手，谢希仁的《计算机网络》说：防止 已失效的连接请求报文段突然又传给server**

> “已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送ack包。（此时因为client没有发起建立连接请求，所以client处于CLOSED状态，接受到任何包都会丢弃，谢希仁举的例子就是这种场景）但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。

## TCP连接的释放

![TCP四次挥手流程](TCP_four_shake_hands.png)

TCP释放连接为什么需要四次挥手呢，因为TCP是全双工的，也即在同一时刻A可以给B传数据，B也可以给A传数据。A传完数据想要关闭连接时，并不代表B也传完数据，也要关闭连接，所以A->B方向的连接和B->A方向的连接需要分别单独关闭。每个方向连接的关闭需要两次挥手，所以加起来就是四次挥手。

