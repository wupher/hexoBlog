title: MongoDB Java Driver 3.0 阅读笔记
date: 2015-04-08
tags: mongodb
---

# MongoDB Java Driver 3.0 阅读笔记

1. 连接的建立入口在 Mongo.java 的第642行 createCluster 函数。
可以看到它折腾了一个maxWaitQueueSize这玩意儿用来做重连的吗？

2. Setting 在上一步设置完之后，就来到了 Mongo.java#656 的createCluster函数，它其实是DefaultClusterFactory来创建。