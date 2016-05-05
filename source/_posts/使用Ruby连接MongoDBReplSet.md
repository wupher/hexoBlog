tags: ruby, mongodb
title: 使用 Ruby 连接 MongoDB Repl Set
date: 2015-2-14
---

# 使用 Ruby 连接 MongoDB Repl Set

标签（空格分隔）： MongoDB Ruby

---

1. 使用`MongoReplicaSetClient`类来连接 MongoDB Repl Set。
2. Repl Set中的主机名必须要严格与rs.status()中的`name`严格一致，假如配合的主机名不可解析，则需要修改`hosts`文件来实现主机名与IP的映射。

示例代码:

```ruby
client = Mongo::MongoReplicaSetClient.new(
    ['nd0302015418:30008', '192.168.158.34:30008'],
    :read => :secondary,
    :refresh_mode => :sync
    )
```

rs.status() 的显示结果：

```json
{
    "set" : "rep1",
    "date" : ISODate("2015-01-19T10:50:50.000Z"),
    "myState" : 1,
    "members" : [ 
        {
            "_id" : 0,
            "name" : "nd0302015418:30008",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 241286,
            "optime" : Timestamp(1421664502, 40),
            "optimeDate" : ISODate("2015-01-19T10:48:22.000Z"),
            "electionTime" : Timestamp(1421423364, 11),
            "electionDate" : ISODate("2015-01-16T15:49:24.000Z"),
            "self" : true
        }, 
        {
            "_id" : 1,
            "name" : "192.168.158.34:30008",
            "health" : 1,
            "state" : 2,
            "stateStr" : "SECONDARY",
            "uptime" : 241286,
            "optime" : Timestamp(1421664502, 40),
            "optimeDate" : ISODate("2015-01-19T10:48:22.000Z"),
            "lastHeartbeat" : ISODate("2015-01-19T10:50:50.000Z"),
            "lastHeartbeatRecv" : ISODate("2015-01-19T10:50:50.000Z"),
            "pingMs" : 4,
            "syncingTo" : "nd0302015418:30008"
        }
    ],
    "ok" : 1
}
```




