title: MongoDB tips
date: 2014-11-11
author: 李宁()，范武(fan_wu@qq.com)
tags: mongodb
---
# MongoDB 最佳实践

## 编程篇

### 使用字段缩写
使用字段缩写意即在 document 设计时采用缩写来表示字段。因为 MongoDB 为非结构化数据库，在储存每个 document 时都会存储字段信息，字段越短，磁盘空间与内存占用就越小。
例子：

```json
{
	'birthday': '1999-9-9',
	'name' : 'Jack Wich',
	'sex' : 'male'
}
```

在缩减之后：

```json
{
	'b' : '1999-9-9',
	'n' : 'Jack Wich',
	's' : 'M'
}
```

> 不过，也有建议不要过度缩减，尽量保证数据的可读性。有种方法是，在 MongoDB 的数据库中保存一张元数据表，用于记录缩减前后的映射，用于未来在丢失文档后，也能理解数据库中字段的涵义。     


```json
{
	'b' : 'birthday',
	'n' : 'name',
	's' : 'sex'
}
```

### 批量数据操作时使用`Bulk()`函数
在最新的2.6版本中，批量数据处理，例如"fire-and-forget"类型的数据插入，应使用 `Bulk()`函数处理，尤其是在 Sharding 系统中。
例如：

```javascript
for (var i = 1; i <= 1000000; i++) {
    db.test.insert( { x : i } );
}
```

应该被写成：

```javascript
var bulk = db.test.initializeUnorderedBulkOp();

for (var i = 1; i <= 1000000; i++) {
    bulk.insert( { x : i} );
}

bulk.execute( { w: 1 } );
```

### 批量`update`
2.6版本的新功能，在一条`update`语句中，可以通过内置的`updates`段，同时对某个 collection 的多条数据同时进行 update。
例子：

```javascript
db.runCommand(
   {
      update: "users",
      updates: [
         { q: { status: "P" }, u: { $set: { status: "D" } }, multi: true },
         { q: { _id: 5 }, u: { _id: 5, name: "abc123", status: "A" }, upsert: true }
      ],
      ordered: false,
      writeConcern: { w: "majority", wtimeout: 5000 }
   }
)
```

新的 `update` 请参见[MongoDB Update 指令参考](http://docs.mongodb.org/manual/reference/command/update/#dbcmd.update)。

### 预估文档大小
MongoDB 中内置的 BSON 文档大小被限制在16MB。另外，频繁对文档进行更新，造成文档大小超过阈值时，可能会引发 MongoDB 的 relocate 操作，会大大降低MongoDB 的性能。在设计时，假如 document 存在子 document 时，这一点尤为需要注意。子 document 是否会频繁更新，进而子 document 是否会无限扩展都是尤其需要进行小心设计的地方。

例子：
常见的论坛、回帖模型。论坛中的帖子与回帖应尽量分至两个不同的 Document 中。而不应该以内置子 Document 的形式而存在。当某个大热贴被频繁回帖时，Document 会被频繁更新，加入回帖的子 document，进而引发前述操作。

### 尽量使用 drop 而非 delete
类似于数据库使用 truncat 而非 delete 是一样的道理。

### 谨慎使用唯一索引
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

## 部署篇

### 尽量使用专有用户来运行 MongodB，而非 root
同时数据文件也存放于相应用户的专有目录中，防止黑客窃取到 shell 权限，进而入侵服务器。