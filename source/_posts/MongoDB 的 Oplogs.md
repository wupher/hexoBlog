title: MongoDB - Oplogs
tags: mongodb
date: 2015-03-12
---

# MongoDB - Oplogs

标签（空格分隔）： MongoDB

oplog（即操作日志）是MongoDB Replica set的核心。Master负责将所有的写操作记录在一个单独的Collection中，Slavery/Secondary 从这个Collection读取操作然后同步到自己的库表中。Oplog 仅当 mongodb 处于 Replica 状态中才会被记录（在standalone状态下，MongoDB 不做操作记录）。

## oplogs 的位置
* 在 master / slave 结构下, oplog 位于`local.oplog.$main`
* 在 Replcia set 结构下， oplog 位于`local.oplog.rs`

## 数据格式 
* ts: 操作的时间戳
* h: 操作的唯一id
* op: 操作的写类型, n 代表 no-op, 表示仅一条状态信息
* ns: 操作的 database 和 collection
* o: 操作的document
* v: oplog 版本
* fromMigrate : true / false, 代表是否是 chunk 迁移

例子：

```json
{
    "ts" : {
        "t" : 1354923335000,
        "i" : 2
    },
    "h" : NumberLong("563747339476084113"),
    "op" : "u",
    "ns" : "msg.device",
    "o2" : {
        "_id" : ObjectId("509fa1207386d978864c7833")
    },
    "o" : {
        "$set" : {
            "flag" : "1",
            "pin" : "5126d5b23c303",
            "device" : "ceb27de6b9dd8f045130f046a7662630",
            "modified" : ISODate("2012-12-07T23:36:24.628Z")
        }
    }
}
```


## 查看服务器上oplogs信息

通过"db.printReplicationInfo()"命令可以查看oplog的信息
![](http://images.cnitblog.com/blog/593627/201412/092248239788547.png)

字段说明：
* configured oplog size： oplog文件大小
* log length start to end: oplog日志的启用时间段
* oplog first event time: 第一个事务日志的产生时间
* oplog last event time: 最后一个事务日志的产生时间
* now: 现在的时间

### 查看slave状态
通过"db.printSlaveReplicationInfo()"可以查看slave的同步状态
![](http://images.cnitblog.com/blog/593627/201412/092248262283660.png)


## 扩展应用
可以基于Oplog做一些有趣的扩展应用

### 触发器
通过监视oplogs，发现感兴趣的操作后，立即做某事，可以实现类似触发器的机制

### 不被同步的collection
可将Collection建立在local中，则此collection不会被同步到其它结点去，可以用来实现一些特殊操作

### 崩溃后的数据恢复
服务崩溃后，直接将其它机器上的oplogs，从当前机器上的oplogs上的最后一条往后全部dump过来，然后让其进行Oplogs同步。（正常MongoDB Repl节点启动后不是这么干的？）






## 参考
* [MongoDB oplog 深入解析](http://duoyun.org/topic/5153c2c26151a6502f00f3b4)
* [Replication Internals](http://www.kchodorow.com/blog/2010/10/12/replication-internals/)
* [Getting to Know Your oplog](http://www.kchodorow.com/blog/2010/10/14/getting-to-know-your-oplog/)
* [Bending the Oplog to Your Will](http://www.kchodorow.com/blog/2010/10/27/bending-the-oplog-to-your-will/)
* [MongoDB副本集的工作原理](http://www.cnblogs.com/wilber2013/p/4154406.html)