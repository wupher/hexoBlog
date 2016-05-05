title: 使用 daemontools 守护 mongodb 服务
date: 2015-1-15
author: 范武[fanwu1977@gmail.com]
tags: mongodb
---

<link rel="stylesheet" href="http://yandex.st/highlightjs/6.1/styles/default.min.css">
<script src="http://yandex.st/highlightjs/6.1/highlight.min.js"></script>
<script>
hljs.tabReplace = '    ';
hljs.initHighlightingOnLoad();
</script>

# 使用 daemontools 守护 MongoDB 服务

## daemontools 安装脚本

daemontools 默认不在 CentOS 源内，使用 DaemonTools 需要进行安装，CentOS 6.5的安装脚本见 `install-daemontools.sh`。

## 使用 daemontools 守护 mongod 进程

> 注意，被管理的进程不能以 daemon 形式运行，所以 mongod 运行时，不要以 fork 模型启动。

### 1. 创建 services 目录

```bash
# mkdir -p /opt/svc/mongodb
```

### 2. 在service目录中创建 run 脚本（名字必须为 run 且权限为755）

```bash
# touch /opt/svc/mongodb/run && chmod 755 /opt/svc/mongodb/run
# cat /opt/svc/mongodb/run
#!/bin/bash
exec mongod -f mongo.conf
```

### 3. 创建 symbol link，创建后即被自动启动

```bash
# ln -s /opt/svc/mongodb/ /services/
``` 

链接创建后，使用`pstree`查看，即可以发现 Mongodb 已开始运行。


## daemontools 常用命令

### 1. 手工启动被管理的进程
```bash
# svc -u /service/mongodb
```
### 2. 手要关闭被管理的进程
```bash
# svc -d /service/mongodb
```
### 3. 查看 service 状态
```bash
# svstat /service/mongodb
```
### 4. 移除 service
```bash
# rm -f /service/mongodb
# svc -dx /opt/svc/mongodb
```