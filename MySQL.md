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

explain 返回各列的含义

![image](https://user-images.githubusercontent.com/102338681/185840915-c0d4fda8-e19b-4997-901e-f879561a3d98.png)

## 1、id ：每个select有一个
## 2、select_type : select 对应的查询类型
## 3、table : 表名
## 4、type: 针对单表的访问方法 
   ### 1、const 当我们根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是const
   ### 2、ep_ref在连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问的（如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较）。则对该被驱动表的访问方法就是eq_ref
   ### 3、ref :当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是ref
   `EXPLAIN SELECT * FROM s1 WHERE key1 = 'a';`
   ### 4、range 范围查询
   ` EXPLAIN SELECT * FROM s1 WHERE key1 IN ('a', 'b', 'c');`
   ` EXPLAIN SELECT * FROM s1 WHERE key1 > 'a' AND key1 < 'b';`
   ### 5、index 当我们可以使用索引覆盖，但需要扫描全部的索引记录时，该表的访问方法就是index
   ### 6、all 全表扫描
   
## 5、possible_keys
可能会用到的索引
## 6、key
实际用到的索引
## 7、key_len
实际使用到的索引长度 (即：字节数)
帮你检查是否充分的利用了索引，值越大越好，主要针对于联合索引，有一定的参考意义。
## 8、ref
当使用索引列等值查询时，与索引列进行等值匹配的对象信息
## 9、rows
预计需要读取的记录条数
## 10、extra
 using where
 using temporary
 using filesort

   
