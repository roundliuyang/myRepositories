### 一、Redis 到底是单线程还是多线程？我要吊打面试官！



最近发布的文章，其中有一道题：

> Redis是多线程还是单线程？（回答单线程的请回吧，为什么请回，请往下看）

好些粉丝在后台问我：**为什么请回，Redis不是单线程吗？**

大家注意审题：**Redis是多线程还是单线程？**

这个问题你要从多个方面回答，如果你仅仅只回答 "单线程" 肯定是说不过去的，为什么呢？

所以今天，栈长利用工作时间紧急把这个问题紧急梳理了下，希望对大家有帮助。

#### 1、Redis 单线程到底指什么？

没错，大家所熟知的 Redis 确实是单线程模型，指的是执行 Redis 命令的核心模块是单线程的，而不是整个 Redis 实例就一个线程，Redis 其他模块还有各自模块的线程的。

下面这个解释比较好：

![img](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpQ5hFss3jOGsnYU6gaOhYXsNmeF1NCoBcpNfLjATuS5jy5Wqu1M1gaUsoL8u9J7aAn9KKpZIkgUibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> Redis基于Reactor模式开发了网络事件处理器，这个处理器被称为文件事件处理器。它的组成结构为4部分：多个套接字、IO多路复用程序、文件事件分派器、事件处理器。
>
> 因为文件事件分派器队列的消费是单线程的，所以Redis才叫单线程模型。
>
> 参考：https://www.jianshu.com/p/6264fa82ac33

#### 2、Redis 不仅仅是单线程

一般来说 Redis 的瓶颈并不在 CPU，而在内存和网络。如果要使用 CPU 多核，可以搭建多个 Redis 实例来解决。

其实，Redis 4.0 开始就有多线程的概念了，比如 Redis 通过多线程方式在后台删除对象、以及通过 Redis 模块实现的阻塞命令等。

来源官方的解释：

![img](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpQ5hFss3jOGsnYU6gaOhYXsw39jwVhlkaNdEPlnic47Z15LnGS17rmFd5mnlafhicibIeQmh3rV2zia8g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果你能说到这里，对 Redis 单/多线程的理解也有你自己更多的认识了。

另外，前些天 [Redis 6](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493693&idx=2&sn=72cad8c4e5b996903a4d131662ff9dc9&scene=21#wechat_redirect) 正式发布了，其中有一个是被说了很久的多线程IO：

![img](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpQ5hFss3jOGsnYU6gaOhYXsx7dT9F7otXQLws9ycsv74ibRldKzsiaGUb1GLTLvO4ZbSOYYW7mqpCjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个 Theaded IO 指的是在网络 IO 处理方面上了多线程，如网络数据的读写和协议解析等，需要注意的是，执行命令的核心模块还是单线程的。

所以，你要是再把 Redis 6.0 网络处理多线程这块回答上了，你也不至于 "请回" 了。

之前有的人在后台和我杠精说：**Redis 6 不是还没发布吗？**

Redis 6 Beta 版本多线程这个说了多久了，作为一个程序员，如果这个还不能 get 到的话，那就有点 OUT 了，如果确实没听说还好，如果听说了，还要和我杠精，我就无言以对了，对于新技术的发展和学习不就是我们和面试官的谈资吗？![img](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaajvl7fD4ZCicMcjhXMp1v6UibtqiaEtqA8BbI6TznttZmhxpib34icdAILNiaCMREqdnmEhiaibuciaMHXCTQ/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 3、为什么网络处理要引入多线程？

之前的段落说了，Redis 的瓶颈并不在 CPU，而在内存和网络。

内存不够的话，可以加内存或者做数据结构优化和其他优化等，但网络的性能优化才是大头，网络 IO 的读写在 Redis 整个执行期间占用了大部分的 CPU 时间，如果把网络处理这部分做成多线程处理方式，那对整个 Redis 的性能会有很大的提升。

网上也有对 Redis 单/多线程情况下的 get/set 操作性能做了对比：

![img](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpQ5hFss3jOGsnYU6gaOhYXsfg2icGKXwPgFUucf527nSsfFYOmJSibxiaibchRU8MMnRQcjnTnwJgUGyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_png/TNUwKhV0JpQ5hFss3jOGsnYU6gaOhYXs5ZWRibzD8icicCoO3fGfeDq4m0NKOU35gfRbQQlLiamOM8uyX9qChDickHQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 参考：blog.csdn.net/weixin_45583158/article/details/100143587

从上面的性能测试图来看，多线程的性能几乎是单线程的两倍了，从该文章来看，这个只是简单的针对多线程性能的验证，并没有做很多严谨的测试，不能作为线上指标参考。

但可以知道的是，Redis 在网络处理方面上了多线程确实会让 Redis 性能上一个新台阶，不过 Redis 6.0 刚发布，不可能有企业马上上生产环境，可能还需要一段时间的优化和验证，我们再期待吧。

最后，目前最新的 6.0 版本中，IO 多线程处理模式默认是不开启的，需要去配置文件中开启并配置线程数，有兴趣的研究下吧。

### 二、为什么Redis是单线程的

**redis 核心就是 如果我的数据全都在内存里，我单线程的去操作 就是效率最高的，为什么呢，因为多线程的本质就是 CPU 模拟出来多个线程的情况，这种模拟出来的情况就有一个代价，就是上下文的切换，对于一个内存的系统来说，它没有上下文的切换就是效率最高的。redis 用 单个CPU 绑定一块内存的数据，然后针对这块内存的数据进行多次读写的时候，都是在一个CPU上完成的，所以它是单线程处理这个事。在内存的情况下，这个方案就是最佳方案 —— 阿里 沈询**

因为一次CPU上下文的切换大概在 1500ns 左右。

从内存中读取 1MB 的连续数据，耗时大约为 250us，假设1MB的数据由多个线程读取了1000次，那么就有1000次时间上下文的切换，

那么就有1500ns * 1000 = 1500us ，我单线程的读完1MB数据才250us ,你光时间上下文的切换就用了1500us了，我还不算你每次读一点数据 的时间，

**那什么时候用多线程的方案呢？**

**答案是：下层的存储等慢速的情况。比如磁盘**

内存是一个 IOPS 非常高的系统，因为我想申请一块内存就申请一块内存，销毁一块内存我就销毁一块内存，内存的申请和销毁是很容易的。而且内存是可以动态的申请大小的。

磁盘的特性是：IPOS很低很低，但吞吐量很高。这就意味着，大量的读写操作都必须攒到一起，再提交到磁盘的时候，性能最高。为什么呢？

如果我有一个事务组的操作（就是几个已经分开了的事务请求，比如写读写读写，这么五个操作在一起），在内存中，因为IOPS非常高，我可以一个一个的完成，但是如果在磁盘中也有这种请求方式的话，

我第一个写操作是这样完成的：我先在硬盘中寻址，大概花费10ms，然后我读一个数据可能花费1ms然后我再运算（忽略不计），再写回硬盘又是10ms ，总共21ms

第二个操作去读花了10ms, 第三个又是写花费了21ms ,然后我再读10ms, 写21ms ，五个请求总共花费83ms，这还是最理想的情况下，这如果在内存中，大概1ms不到。

所以对于磁盘来说，它吞吐量这么大，那最好的方案肯定是我将N个请求一起放在一个buff里，然后一起去提交。

方法就是用异步：将请求和处理的线程不绑定，请求的线程将请求放在一个buff里，然后等buff快满了，处理的线程再去处理这个buff。然后由这个buff 统一的去写入磁盘，或者读磁盘，这样效率就是最高。java里的 IO不就是这么干的么~

对于慢速设备，这种处理方式就是最佳的，慢速设备有磁盘，网络 ，SSD 等等，

多线程 ，异步的方式处理这些问题非常常见，大名鼎鼎的netty 就是这么干的。

终于把 redis 为什么是单线程说清楚了，把什么时候用单线程跟多线程也说清楚了，其实也是些很简单的东西，只是基础不好的时候，就真的尴尬。。。。

补一发大师语录：来说说，为何单核cpu绑定一块内存效率最高

“我们不能任由操作系统负载均衡，因为我们自己更了解自己的程序，所以我们可以手动地为其分配CPU核，而不会过多地占用CPU”，默认情况下单线程在进行系统调用的时候会随机使用CPU内核，为了优化Redis，我们可以使用工具为单线程绑定固定的CPU内核，减少不必要的性能损耗！

redis作为单进程模型的程序，为了充分利用多核CPU，常常在一台server上会启动多个实例。而为了减少切换的开销，有必要为每个实例指定其所运行的CPU。

Linux 上 taskset 可以将某个进程绑定到一个特定的CPU。你比操作系统更了解自己的程序，为了避免调度器愚蠢的调度你的程序，或是为了在多线程程序中避免缓存失效造成的开销。

顺便再提一句：redis 的瓶颈在网络上 。