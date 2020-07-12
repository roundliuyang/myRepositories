
![叶子分割线](https://mmbiz.qpic.cn/mmbiz_png/xrkWfYibics39AbBIZtiaAkrZMFcBtKQ4aRaiakRCbgWmibw8R1qr2hPcUJZsH5SG7mjeEpfibSKGkIiceMOFKZ7wQPrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)



### MySQL如何解决幻读问题？

​    先来说说幻读的概念吧，在MySQL中，如果一个事务A根据某种特定条件的SQL查询出来一些记录record_a，此时另外一个事务插入了一些符合这种特定条件的记录record_b，原先的事务再次根据同样的SQL，查询到了record_a和record_b,这种现象就称之为幻读。

​    注意，下面两种情况不能称之为幻读：

1、如果查出来数据比record_a的记录要少，

2、或者查出来的数据跟record_a记录一样，但是记录发生了变化。

​    幻读强调的是一个事务按照相同的SQL查询了记录之后，后续的结果中出现了之前结果中不存在的值。



在默认RR隔离级别下，当发生了幻读现象之后，MySQL解决这种情况会使用两种方案。

**方案一：读操作利用MVCC解决，写操作利用加锁解决**

MVCC知识可以查看之前的文章：

[《MySQL之MVCC初探(1)](http://mp.weixin.qq.com/s?__biz=MzUyNjkzNjQwMQ==&mid=2247485153&idx=1&sn=29718324572090e64669cbb2639e23a9&chksm=fa0676dfcd71ffc9b27a8df2e400a24ab00d5c365732f74db9eb42cc340c2b928d830b3017f5&scene=21#wechat_redirect)》

MVCC其实是借助于Readview(读视图)的概念，对数据库生成Readview时刻的版本做了一个快照。普通的查询语句只能看到生成Readview之前已经提交的事务，在生成Readview之前未提交的事务或者生成Readview之后才开启的事务是看不到的。MVCC情况下读取的都是记录的历史版本，而写操作都是更新的是记录的最新版本，因此，MVCC情况下，读操作和写操作本身并不冲突。

说的更简单一点就是RR隔离级别下，事务在第一次select的时候只生成一次Readview(类似拍了一张照片)，后续的查询都复用这个Readview（同样的照片），当然，也就不会出现幻读现象了。



**方案二：读写操作都采用加锁的方式**

在银行支付等场景下，不允许读取记录的历史版本，只允许看到记录的最新版本，此时读操作和写操作都需要加锁，其实，要解决幻读问题，只添加记录锁于事无补，因为幻读的记录在第一次读取之前是不存在的，即使你给记录上了记录锁，但是不存在的记录上面如果没有锁，还是会造成幻读。

为了解决这个问题，MySQL引入了间隙锁，间隙锁的引入，阻止了其他会话在指定的间隙插入相关记录，也就解决了幻读的问题。





**两种方案对比：**

如果采用MVCC方式的话，**只能解决一致性非锁定读(也称之为快照读)的幻读问题**，读-写操作彼此并不冲突，并发性能更高；

如果采用加锁方式的话，**可以解决当前读的幻读情况**，读-写操作彼此需要排队执行，影响性能；

一般情况下我们当然愿意采用MVCC来解决读-写操作并发执行的问题，但在银行业务等特殊场景下，还是需要锁来解决的。
