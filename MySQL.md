云数据库 DRDS （Distributed Relational Database Service）是云上百度提供的基于中间件的分布式关系型数据库系统服务。云数据库 DRDS 可以基于普通服务器和横向扩展的方式，构建支持海量数据存储和访问的数据库系统，从而实现无限扩容和弹性扩展。相比云数据库 RDS，云数据库 DRDS 提供了更高规格的存储和 QPS，满足用户持续增长的海量数据存储需求以及持续增长的业务请求压力。

主从复制的原理：三个线程

实际上主从同步的原理就是基于 binlog 进行数据同步的。在主从复制过程中，会基于3个线程 来操 作，一个主库线程，两个从库线程。

<img width="906" alt="image" src="https://user-images.githubusercontent.com/102338681/185834654-677bd10f-1ebe-4f80-9173-ab685dc7c6e4.png">

![image](https://user-images.githubusercontent.com/102338681/185834667-a156a3b4-84a4-4504-85e8-0a421dab4b87.png)

二进制日志转储线程 （Binlog dump thread）是一个主库线程。当从库线程连接的时候， 主库可以将二进制日志发送给从库，当主库读取事件（Event）的时候，会在 Binlog 上加锁 ，读取完成之后，再将锁释放掉。

从库 I/O 线程 会连接到主库，向 主库 发送请求 更新 Binlog。这时从库的 I/O 线程就可以读取到主库的 二进制日志转储线程 发送的 Binlog 更新部分，并且拷贝到本地的中继日志（Relay log）。

从库 SQL 线程 会读取 从库中的中继日志，并且执行日志中的事件，将从库中的数据与主库保持同步。

一、半同步复制，解决主库数据可能丢失的问题
<img width="868" alt="image" src="https://user-images.githubusercontent.com/102338681/185834762-cf30a730-1be7-477e-aec2-c7610b4906a5.png">
![image](https://user-images.githubusercontent.com/102338681/185834780-d6fec9bf-41b1-48df-b280-c975913c42bd.png)
