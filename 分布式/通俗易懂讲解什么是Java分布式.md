> 作者：戏精程序媛
>
> blog.csdn.net/xiaoxijing/article/details/99865038

对于初学者来说，对分布式开发比较含糊，特别是apache下面的hadoop、hdfs、hbase，这些基本是分布式开发的标配。那么这篇文章就来和大家一起聊聊分布式吧！

## 一、什么是分布式系统？

要理解分布式系统，主要需要明白一下2个方面：

**1、分布式系统一定是由多个节点组成的系统。**

其中，节点指的是计算机服务器，而且这些节点一般不是孤立的，而是互通的。

**2、这些连通的节点上部署了我们的节点，并且相互的操作会有协同。**

分布式系统对于用户而言，他们面对的就是一个服务器，提供用户需要的服务而已。而实际上这些服务是通过背后的众多服务器组成的一个分布式系统。因此分布式系统看起来像是一个超级计算机一样。

例如淘宝，平时大家都会使用，它本身就是一个分布式系统。我们通过浏览器访问淘宝网站时，这个请求的背后就是一个庞大的分布式系统在为我们提供服务，整个系统中有的负责请求处理，有的负责存储，有的负责计算，最终他们相互协调把最后的结果返回并呈现给用户。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqB8I3rmjYl3qRPTdmHfCZVz0Qk4Qbu2Cu491gBicosSgpsKXtibuiaYvibfqBWL6aMl5xgtQ2ZCzsOKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 使用分布式系统主要有特点：

**1、增大系统容量。**我们的业务量越来越大，而要能应对越来越大的业务量，一台机器的性能已经无法满足了，我们需要多台机器才能应对大规模的应用场景。所以，我们需要垂直或是水平拆分业务系统，让其变成一个分布式的架构。

**2、加强系统可用。**我们的业务越来越关键，需要提高整个系统架构的可用性，这就意味着架构中不能存在单点故障。这样，整个系统不会因为一台机器出故障而导致整体不可用。所以，需要通过分布式架构来冗余系统以消除单点故障，从而提高系统的可用性。

3、因为模块化，所以系统模块重用度更高。

4、因为软件服务模块被拆分，开发和发布速度可以并行而变得更快。

5、系统扩展性更高。

6、团队协作流程也会得到改善。

### 分布式系统的类型有三种：

1、分布式处理，但只有一个总数据库，没有局部数据库。

2、分层式处理，每一层都有自己的数据库。

3、充分分散的分布式网络，没有中央控制部分，各节点之间的联系方式又可以有多种，如松散的联接，紧密的联接，动态的联接，广播通知式的联接等。

## 二、什么是Java分布式应用？

一个大型的系统往往被分为几个子系统来做，一个子系统可以部署在一台机器的多个JVM上，也可以部署在多台机器上。但是每一个系统不是独立的，不是完全独立的。需要相互通信，共同实现业务功能。

一句话来说：分布式就是通过计算机网络将后端工作分布到多台主机上，多个主机一起协同完成工作。

## 三、实现分布式主要的方式

分布式应用用到的技术：网络通信，基于消息方式的系统间通信和基于远程调用的系统间通信。

缺点：就是会增加技术的复杂度。基于消息的系统通信方式，主要是利用的网络协议，比如TCP/IP协议。系统间的通信还需要对数据进行处理，比如同步IO和异步IO。

远程调用实现系统间的通信：通过调用本地的Java接口的方法来透明的调用远程Java的实现。具体的细节有框架来实现。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqB8I3rmjYl3qRPTdmHfCZVoSg1WecGrJXC6w5icaG2qibdGp7GJ7rgE2Q5TjUbRY5ampPo7n8Dd6Dw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 基于Java自身技术实现消息方式的系统间通信

基于Java自身包实现消息方式的系统间通信的方式有：

`TCP/IP+BIO`、`TCP/IP+NIO`、`UDP/IP+BIO`以及`UDP/IP+NIO` 4种方式。

TCP/IP+BIO在Java中可基于Socket、ServerSocket来实现TCP/IP+BIO的系统间通信。

Socket主要用于实现建立连接及网络IO的操作，ServerSocket主要用于实现服务器端端口的监听及Socket对象的获取。

多个客户端访问服务器端的情况下，会遇到两个问题：建立多个socket的，占用过多的本地资源，服务器端要承受巨大的来访量；创建过多的socket，占用过多的资源，影响性能。

通常解决这种问题的办法是，使用连接池，既能限制连接的数量，又能避免创建的过程，可以很大的提高性的问题。缺点就是竞争量大的时候造成激烈的竞争和等待。需要注意的是，要设置超时时间，如果不这样的话，会造成无限制的等待。

为了解决这个问题，采用一连接一线程的方式，同时也会带来副作用，内存占用过多。

TCP/IP异步通信：Java NIO通道技术实现。

以上就是对Java分布式的理解了。希望看完这篇文章大家对Java分布式有更深层次的认识。