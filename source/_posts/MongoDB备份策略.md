title: MongoDB备份策略
date: 2014-10-20
tags: mongodb,运维
---

# MongoDB 备份策略
标签（空格分隔）： MongoDB

## 对Stand alone MongoDB 服务器进行备份
备份 MongoDB 服务的方法有很多，但是无论采用哪种备份方案，备份操作都会增加系统负担：备份通常需要将你的所有数据都读入内存中。对于 Repl set 来说，最好在副结点上进行备份，对于 stand alone 来说，最好是在空闲或者维护时段进行备份。

### 使用文件系统快照来进行备份
生成文件 snapshot 是最简单的备份方法，使用这种方案要求你的文件系统支持 snapshot，并且你的 mongod 服务在启动打开了 journaling 选项。如果满足以上两点，则可随时按需生成快照。

在恢复数据时，需要确保 Mongod 没有在运行。基本上，按照文件系统的要求，恢复 文件 snapshot 后重启 mongod 进程即可。

### 拷贝数据文件
另外一种方案是复制数据目录中的全部数据文件。当然，在文件拷贝时为了防止数据文件发生变化，你需要在拷贝前将数据文件进行锁定：

```javascript
db.fsyncLock();
```

该命令锁定数据库，禁止任何数据写入，并进行同步操作：将所有的脏页刷新至磁盘，确定数据目录中的所有文件是最新的。

一旦运行该命令，mongod 会将所有之后的写入操作加入至队列中进行等待，并在解锁前不会对这些操作有任何处理。注意，这一操作会停止 __所有__ 数据库的写入操作，而非仅仅是当前连接的 db。

数据复制完成后，需要对数据库进行解锁：
```javascript
db.fsyncUnlock()
```

> 
注意：MongoDB 的 Authorization 机制与`fsyncLock`之间存在一些锁定问题。如果启用了身份验证，则在调用`db.fsyncLock`与`db.fsyncUnlock`之间不能关闭 shell。如果期间关掉了mongo shell，则可能出现无法重新进行连接，需要重启 mongod。而 mongod 总是以非锁定模式启动。

除了使用`fsyncLock`之外，还可以干脆关闭 mongod 进程，再来复制备份文件，然后重启 mongod。关闭 mongod 会将所有的更改立即刷新到磁盘，防止备份期间出现新的写入操作。

同样，在恢复数据时，需要确定 mongod 没有在运行。将待恢复数据目录进行清空并将之前复制备份的文件拷贝回原位置，然后重启 mongod 进程即可。

除了全部数据文件复制以外，也可以根据数据库的名字进行按需备份。尤其是在使用了`--directorydb`选项后，仅需要按需备份相应的子目录下所有内容即可。

> 注意：不要同时使用`fsyncLock` 和 `mongodump`。数据库被锁也许会使 `mongodump`永远处于挂起状态。


### 使用 mongodump
这是一个与 MongoDB 一起捆绑发布的工具，它能对 MongoDB 的数据实时进行备份。通过 MongoDump 导出整个数据库、集合或者某个查询的结果。Mongodump 的缺点是备份和恢复的速度相对最慢，以及在处理 Repl Set 时会有一些问题。但是，当需要备份单独的数据库、collection、甚至 collection 中的子集时，`mongodump`会是一个好的选择。

`mongodump`存在一个问题，它并非对快照备份，也就是说备份进行过程中，系统可能会继续进行写入操作。于是可能出现 ，开始备份时先对 A 数据库进行了转储，随后在进行对 B 数据库进行转储时，删除了数据库 A。然后，由于已经对 A 进行了转储，于是最终转储的结果是一个在原服务器上并不存在的数据快照 。

为了避免，运行 mongodump 时使用了`--replSet`选项，则可使用`mongodump`的`--oplog`选项。这会将转储过程中服务器进行的所有操作记录下来，这样在恢复数据时会重新执行这样操作。这样就可以得到某一时间点的数据快照。

如果给`mongodump`一个repl set 的连接字串（例如：setName/seed1, seed2, seed3），如果备份结点存在的话，它会自动选择一个备份结点进行转储。

>注意：在任何集合中，如果存在`_id`以外的唯一索引（unique index），则应该考虑使用`mongodump`和`mongorestore`以外的备份方式。具体来说，唯一索引要求复制期间数据不发生可能破坏其唯一索引约束的改变。最安全的办法是先想法办法『冻结』数据，再使用前两节中提到的方法进行备份。如果决定 使用 mongodump和 mongorestore 来备份，那么在数据恢复时，可能需要对数据进行一定的预处理。

## 对 Repl set 进行备份
通常应该使用副节点进行数据备份：这会为主结点减轻负担，也可以在不影响应用的情况下锁定备份结点，使用前述三种方法中的任意一种对副本集中的成员进行备份。推荐使用文件系统快照或复制数据文件的方式。

启动 repl set 后，使用 mongodump 备份必须在使用时使用`--oplog`选项，来得到基于某个时间点的快照；否则备份的状态不会和任何其它集群成员的状态相吻合。在恢复时同样必须创建一份 oplog，否则被恢复的成员就不知道该同步到哪里。

要从 mongodump 生成的备份中，对 repl set 成员进行恢复，可将成员作为一个 standalone 服务器进行启动，此时要使用一个空的数据目录。首先，使用`--oplogReplay`选项运行 mongorestore。现在它应该包含了一份完整的数据副本，但是还需要一份 oplog。运行 createCollection 命令来建立 oplog：
```javascript
use local
db.createCollection("oplog.rs", {"capped":true, "size":10000000})
```
以字节单位指定集合大小。

现在需要填充 oplog。最简单的方式是用备份中的 oplog.bson 文件来填充 local.oplog.rs 集合：
```bash
mongorestore -d local -c oplog.rs dump/oplog.bson
```

> 注意：这并不是全部 oplog 的转储文件（dump/local/oplog.rs.bson）, 而是进行转储期间发生的操作。一旦 mongorestore 完成，即可将服务器作为 repl set 成员重新启动。

## 对 sharding 集群进行数据备份
无法对正在运行的 sharding 集群进行『完美』备份，因为无法及时取得集群在某一时间点完整状态的快照。随着集群的增大，从备份中恢复整个集群的可能性就越小。因此，在面对 sharding 集群时，更应关注分块的备份，即单独备份配置服务器和副本集。

在对 sharding 集群进行备份和恢复之前，应先关闭平衡器。

### 备份和恢复整个集群
当集群很小或处于开发期时，可能需要转储和恢复整个集群。首先，应关闭平衡器；然后通过`mongos`运行`mongodump`。这会在`mongodump`所运行的机器上建立所有分片的备份。

要恢复此种备份，需要运行`mongorestore`并连接到一个 mongos。

关闭平衡器后，可使用文件系统快照或复制数据目录的方式，备份配置服务器和每个分片。然后，不可避免的是，我们不可能在完全相同的时刻得到这些备份，这可能会造成问题。另外，在打开平衡器时会进行数据合并，在分片中备份的某些数据可能会由此消失。

### 备份和恢复单独的分片
更多的时候，只需要恢复集群中的某个单独分片。不是很挑剔的话，可使用在前面提到过的单独服务器处理的方法进行分片的备份恢复。

有一个问题需要注意：假设在星期一对集群进行了备份。到了周四，磁盘损坏，我们需要恢复备份。然而，这几天里，新的数据块可能移动到了这一分片上。而周一备份的分片数据中并不包含这些新增的数据块。也许我们能够使用配置服务器的备份，找到这些消失的数据块在星期一的位置，但这比只是恢复分片要困难的多。
>在大多数情况下，恢复分片，忽略那些消失的数据块，是更好的选择。

单独为某一 sharding 备份时，可以直接连到 sharding 上进行备份，不需要通过 mongos。

### 使用 mongooplog 进行增量备份
前述备份方式，即使和上一次备份相比，只发生了很小的改变，也必须对所有数据进行一次完整复制。如果数据和写入量有很大关系，可能通过 oplog 来进行增量备份。

与每天或者每周进行一次完整的数据复制不同，只需要进行一次完整的数据备份之后，然后使用 oplog 来备份这之后的操作。这种方案比之前提及的技术都要复杂，因此除非确实需要，否则应尽量选择其它技术。

使用这一技术需要两台运行 mongod 的机器，即机器 A 和机器 B。A 为主机（可能是 repl set 中的备份节点），B 则用来进行备份：
1. 记录下 A 的 oplog 中最近一次的操作时间（optime）：
```javascript
op = db.oplog.rs.find().sort({$natural:-1}).limit(1).next();
start = op['ts']['t']/1000
``` 
2. 对数据进行备份，使用以上提及的任何一种方式，得到一份基于某个时间点的备份，恢复备份至 B 上的数据目录。
3. 定期添加 A 上的操作至 B，从而完成数据的复制。MongoDB 发行版中自带的工具`mongooplog`使用这一操作变得简单。`mongooplog`从一台服务器的`oplog`中复制数据，并将其中的操作应用在另一台服务器的数据集上。在 B 上运行：
```javascript
mongooplog --from A --seconds 1234567
```
其中--seconds 选项后跟的参数应为第一步中计算出的 start 变量和当前时间的差值，再额外加上几秒（重复地重放操作也好过数据丢失）。

从本质上说，这种技术像是手动地同步一个备份节点，所以我们也许只是想在备份节点上使用延时复制以代替增量备份。


## 使用 MongoDB 管理服务（MMS）进行数据备份
MMS可以托管到数据中心，将 MMS 备份代理安装到 MongoDB 集群上，并招待初始同步，在此之后，加密和压缩的 oplog 数据会从备份代理同步到 MMS 上，快照每6小时创建一次。

![MMS 结构图](http://i.imgur.com/MzUEBoi.png)

MMS备份可支持Repl Set时间点恢复和 sharding集群整体快照恢复。过去24小时内的oplog 数据会被存储起来，可以使用这些数据一个 Repl Set 创建定制快照。对于 Sharding 集群，负载均衡会每6小时暂停一次，同时一个无操作令牌会被插入到所有分片、Mongos和配置服务器上。oplog 能够被应用到集群中的所有Repl Set 上，为整个集群提供一个一致的快照。


## 参考
* [Preparing for Your First MongoDB Deployment: Backup and Security](http://www.infoq.com/articles/mongodb-deployment-backup-security)
* [MMS backup](https://mms.mongodb.com/help/backup/)
* [Backup and Restore with FileSystem Snapshots](https://mms.mongodb.com/help/backup/)
* [MongoDB Backup and Recovery](http://docs.mongodb.org/manual/administration/backup/)
* [MongoDB Operations Best Practices](http://info.mongodb.com/rs/mongodb/images/10gen-MongoDB_Operations_Best_Practices.pdf)
* [MongoDB权威指南第二版](http://www.amazon.cn/MongoDB%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97-%E9%9C%8D%E5%A4%9A%E7%BD%97%E5%A4%AB/dp/B00JVLEYYW/ref=pd_sim_kinc_3?ie=UTF8&refRID=1SVWA9JJ6XEPP01MBG29)