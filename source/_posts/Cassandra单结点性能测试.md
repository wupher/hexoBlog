tags: cassandra,ycsb, 测试
title: Cassandra 单结点性能测试
date: 2015-03-12
---

# Cassandra 单结点性能测试

标签（空格分隔）： cassandra ycsb benchmark 

---

## 环境
* 机型： DELL R720
* CPU ： 逻辑CPU 24
* 内存： 32GB
* 硬盘： 300*8 Raid0+1 非 SSD
* 操作系统: CentOS 6.6
* Kernel版本： Linux version 2.6.32-431.el6.x86_64 
* 测试套件：使用YCSB对Cassandra 进行压力测试
* cassandra: 2.1

## 多读少写场景（1千万次操作，90%读10%写）

* 并发：150线程
* TPS :  41539.899472438

| 平均延迟时间(us) | 最小延迟时间(us) | 最大延迟时间(us) | 95%情况下的延迟(ms) | 99%情况下的延迟(ms)|type|
| --- | --- | --- |---|---| ---- | --- |
| 2396 | 160 | 1095039 | 9 | 30| on write|
| 3126 | 238 | 1547759 | 11 | 37 | on read|

* 并发： 50线程
* TPS ： 42996.328113579104

| 平均延迟时间(us) | 最小延迟时间(us) | 最大延迟时间(us) | 95%情况下的延迟(ms) | 99%情况下的延迟(ms)|type|
| --- | --- | --- |---|---| ---- | --- |
|769 | 170 |398553 | 1 |３|on wirte|
| 1106 | 325 | 405006| 2 | 3 | on read|


## 工作负载

* CPU: CPU Avg Load最高达120，瓶颈主要在CPU上

![CPU负载](http://i.imgur.com/9HPvUZt.jpg)

![CPU Load](http://i.imgur.com/RUEgM8n.png)


* 内存：Cassandra对于内存的占用一般

![内存负载](http://i.imgur.com/3BH2Wor.jpg)

* 磁盘IO负载：因为测试的是多读少写，所以对磁盘IO消耗不大。

![磁盘负载](http://i.imgur.com/y6T7EEH.jpg)


## 结论
结合Cassandra文档，多读少写性能测试可得出以下结论：

1. Cassandra并不支持大连接，文档中也说一应用开一session即可，session本身会处理与cassandra的并发连接池。cassandra连接过大时，性能指标反而会下降。
2. 多读少写场景中，Cassandra影响最大的是CPU与内存。如果增大CPU与内存，TPS 性能数据应还有提高。
3. Cassandra本身对于SSD也做了一些特殊优化，有些设置专门针对SSD，有条件的话，建议以SSD环境再测试一次Cassandra的性能。





