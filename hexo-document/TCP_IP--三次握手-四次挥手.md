---
title: TCP/IP--三次握手，四次挥手
date: 2016-06-14 10:23:00
tags: [TCP/IP,三次握手，四次挥手,网络]
categories: 编程 # 配置categories
---

## Question

* TCP为什么需要3次握手与4次挥手?
* 三次握手，四次挥手，为什么要用四次挥手，三次不行吗，当Client收到Server的ack后，Client还能接收来自Server的数据写入吗？

或者说是怎么去解释“三次握手，四次挥手”？

![tcp-head](http://i.imgur.com/Cn6koqL.png)

符号说明：

* seq:"sequance"序列号，Seq序号，占32位，用来标识从TCP源端向目的端发送的字节流，发起方发送数据时对此进行标记。

* ack:"acknowledge"确认号

标志位，共6个，即URG、ACK、PSH、RST、SYN、FIN等，具体含义如下：

* SYN:"synchronize"请求同步标志

* ACK:"acknowledge"确认标志"，Ack序号，占32位，只有ACK标志位为1时，确认序号字段才有效，Ack=Seq+1。

* FIN：finish结束，"Finally"结束标志,释放一个连接。

* RST:reset重置，重置连接。

* URG:urgent紧急，紧急指针（urgent pointer）有效。

* PUS:push传送，接收方应该尽快将这个报文交给应用层。

注意：确认方Ack=发起方Req+1，两端配对。

### 三次握手

![tcp1](http://i.imgur.com/yrgs5LB.png)

首先Client端发送连接请求报文，Server段接受连接后回复ACK报文，并为这次连接分配资源。Client端接收到ACK报文后也向Server段发生ACK报文，并分配资源，这样TCP连接就建立了。

reference:[http://blog.csdn.net/whuslei/article/details/6667471/](http://blog.csdn.net/whuslei/article/details/6667471/ "http://blog.csdn.net/whuslei/article/details/6667471/")

![tcp_draw](http://i.imgur.com/uVngHhd.png)

* 第一次握手：Client将标志位SYN置为1，随机产生一个值seq=J，并将该数据包发送给Server，Client进入SYN_SENT状态，等待Server确认。

* 第二次握手：Server端收到TCP数据包之后，由标志位SYN=1知道Client请求简历TPC链接，Server端将标志位ACK和SYN都置为1（1表示有效），Act=J + 1，并随机产生一个Seq序列号，Seq=k，然后将该数据包发给Client端以确认并且同步此链接请求，Server端进入SYN_REVD状态.

* 第三次握手：CLient端收到确认后，首先检查ACK是否为1，且Ack是否为j+1，如果正确，则将ACK标志置为1，ack=k+1,并将该数据包发送给Server端，Server端检查Ack是否为k+1，ACK是否为1，如果正确，则表示链接建立成功Client和Server进入ESTABLISHED状态，完成三次握手之后，随之C和S就可以互通数据了。

“**三次握手**”的目的是“为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误”。

这是因为：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送ack包。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样server的很多资源就白白浪费掉了。采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。主要目的防止server端一直等待，浪费资源。

### 四次挥手

![tcp](http://i.imgur.com/oDESM4a.png)

服务端在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。而关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。

![tcp2](http://i.imgur.com/g4u6Tko.png)

>假设Client端发起中断连接请求，也就是发送FIN报文。Server端接到FIN报文后，意思是说"我Client端没有数据要发给你了"，但是如果你还有数据没有发送完成，则不必急着关闭Socket，可以继续发送数据。所以你先发送ACK，"告诉Client端，你的请求我收到了，但是我还没准备好，请继续你等我的消息"。这个时候Client端就进入FIN_WAIT状态，继续等待Server端的FIN报文。当Server端确定数据已发送完成，则向Client端发送FIN报文，"告诉Client端，好了，我这边数据发完了，准备好关闭连接了"。Client端收到FIN报文后，"就知道可以关闭连接了，但是他还是不相信网络，怕Server端不知道要关闭，所以发送ACK后进入TIME_WAIT状态，如果Server端没有收到ACK则可以重传。“，Server端收到ACK后，"就知道可以断开连接了"。Client端等待了2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，我Client端也可以关闭连接了。Ok，TCP连接就这样关闭了！


### 整个过程流程图

建立TCP需要三次握手才能建立，而断开连接则需要四次握手。整个过程如下图所示：
![tcp3](http://i.imgur.com/4VD19KB.jpg)

### Question

【注意】 在TIME_WAIT状态中，如果TCP client端最后一次发送的ACK丢失了，它将重新发送。TIME_WAIT状态中所需要的时间是依赖于实现方法的。典型的值为30秒、1分钟和2分钟。等待之后连接正式关闭，并且所有的资源(包括端口号)都被释放。

【问题1】为什么连接的时候是三次握手，关闭的时候却是四次握手？

答：因为TCP链接是全双工模式，因此需要close链接时需要两方都需要单独进行链接释放。当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

【问题2】为什么TIME_WAIT状态需要经过2MSL(最大报文段生存时间)才能返回到CLOSE状态？

答：虽然按道理，四个报文都发送完毕，我们可以直接进入CLOSE状态了，但是我们必须假象网络是不可靠的，有可以最后一个ACK丢失。所以TIME_WAIT状态就是用来重发可能丢失的ACK报文。

【问题3】为什么收到Server端的确认之后，Client还需要进行第三次“握手”呢？

答：在只有两次“握手”的情形下，假设Client想跟Server建立连接，但是却因为中途连接请求的数据报丢失了，故Client端不得不重新发送一遍；这个时候Server端仅收到一个连接请求，因此可以正常的建立连接。但是，有时候Client端重新发送请求不是因为数据报丢失了，而是有可能数据传输过程因为网络并发量很大在某结点被阻塞了，这种情形下Server端将先后收到2次请求，并持续等待两个Client请求向他发送数据...问题就在这里，Cient端实际上只有一次请求，而Server端却有2个响应，极端的情况可能由于Client端多次重新发送请求数据而导致Server端最后建立了N多个响应在等待，因而造成极大的资源浪费！所以，“三次握手”很有必要！

并且，两次握手可能产生死锁。作为例子，考虑计算机server和client之间的通信，假定client给server发送一个连接请求分组，server收到了这个分组，并发送了确认应答分组。按照两次握手的协定，server认为连接已经成功地建立了，可以开始发送数据分组。可是，client在server的应答分组在传输中被丢失的情况下，将不知道server是否已准备好，不知道server建立什么样的序列号，client甚至怀疑server是否收到自己的连接请求分组。在这种情况下，client认为连接还未建立成功，将忽略server发来的任何数据分组，只等待连接确认应答分组。而S在发出的分组超时后，重复发送同样的分组。这样就形成了死锁。


### TCP的具体状态图可参考：

![tcpstatus](http://i.imgur.com/7HzcDvp.png)