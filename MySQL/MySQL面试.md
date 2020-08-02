## MySQL面试总结

### mysql的主从复制原理



- MySql主库在事务提交时会把数据变更作为事件记录在二进制日志Binlog中；

- 主库推送二进制日志文件**Binlog**中的事件到从库的中继日志**Relay Log**中，之后从库根据中继日志重做数据变更操作，通过逻辑复制来达到主库和从库的数据一致性；

- MySql通过三个线程来完成主从库间的数据复制，其中Binlog Dump线程跑在主库上，**I/O线程和SQL线程**跑着从库上；

- 当在从库上启动复制时，首先创建**I/O线程**连接主库，主库随后创建Binlog Dump线程读取数据库事件并发送给**I/O线程**，**I/O线程**获取到事件数据后更新到从库的中继日志Relay Log中去，之后从库上的**SQL线程**读取中继日志**Relay Log**中更新的数据库事件并应用，如下图所示。

  

![主从复制.jpg](https://github.com/roundliuyang/image/blob/master/MySQL/%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6.jpg?raw=true)

