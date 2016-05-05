title: MongoDB的安全机制笔记
author: 范武（fanwu1977@gmail.com）
date: 2014-11-08
tags: mongodb
---

# MongoDB的安全机制笔记

## Authentication
MongoDB支持两种机制的安全认证：基于密码的安全认证，以及x.509公密钥加密的安全认证。LDAP和Kerberos神马的，都是MongoDB Enterprise才支持，同学你还是忘记吧。

默认情况下，MongoDB的RBAC并不启用，你需要在启动命令行（--auth / --keyFile）中打开，或者在设置文件中的security.authorization/security.keyFile中打开。

MongoDB内置了一些权限角色，用于常用的情况下进行使用，默认的角色有：read, readWrite, dbAdmin, root。

## 审计
MongoDB支持审计！但仅限2.6版本，还必须是企业版，所以，忘了它吧。如果忘不掉，就看[这里](http://docs.mongodb.org/manual/core/auditing/)。

## 加密
当然，加密会使你的程序更安全，当然它也肯定会消耗更多的CPU用于加密与解密，期间权衡在你取舍。

### 传输加密
MongoDB支持使用SSL来加密所有的网络数据交换，保证传输的数据仅被可靠的Client所读取。所以，它用来存艳照门图片真是再好不过了。不过，先别急，它还有以下的几个小事项需要注意：

* 你从源或者MongoDB网站上弄下来的发行版是不支持这个功能的，想用SSL，需要本地编译MongoDB，并在编译时启用--SSL选项通知`scons`；或者，掏钱[买个企业版](http://www.mongodb.com/products/mongodb-enterprise?_ga=1.235753025.1784023503.1402322927)呗。
* Windows版本对于SSL的支持，呵呵，不过据说企业版是支持的。
* 至少128位，密钥要求至少为128bit加密。
* mongoDump / mongoExport / mongoFiles / mongoImpot /mongooplog /mongorestore / mongostat/ mongotop这些都支持SSL加密，但是如果你用的是第三方工具，需要自己检查一下了。
* 至于Client，目前，pytmongo / Java / Ruby / node-mongodb-native / .net都支持SSL连接至于go啊，php神马的，呵呵。以上语言的连接示例代码，请看[这里](http://docs.mongodb.org/manual/tutorial/configure-ssl-clients/)。

### 应用级别加密
就是自己加密些密码字段啊，把某个document或者子document进行加密，最常见的就是把密码MD5化。这个玩意儿，应用程序自己折腾就好。话说，愿意掏银子的话，[这里](http://www.vormetric.com/sites/default/files/sb-MongoDB-Letter-2014-0611.pdf)也有另种解决方案。

### 存储加密
加密所有存储于硬盘上的文件，保证文件被人拷贝后，无法被正确读出。这玩意儿同样属于操作系统插件类，目前有以下方案：

* Linux Unified Key Setup: RedHat企业版支持，CentOS神马的请无视，详情[见此](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Encryption.html)
* IBM Guardium Data Encryption : 当然是要花钱的，不然也对不起这么高大上的名字 
* Vormetric Data Security Platform : 前面有链接，呵呵，就是应用级别加密那边。
* Bitlocker Drive Encryption: M$家的，Win Server 2008 ~ 2012，话说，把MongoDB部署在这上面，再次呵呵。

## 安全机制 HOW -TOs
下面是一系列安全列表及实现的HOW-TO链接。安全级别以从低到高排列

### 开启安全认证
开启MongoDB的安全认证机制，开启安全认证后，所有的Client及Server需要经过正确认证后方能进行访问，怎么玩，见下：

* [Enable Client Access Control](http://docs.mongodb.org/manual/tutorial/enable-authentication/)
* [Enable Authorization in a Sharded Cluster](http://docs.mongodb.org/manual/tutorial/enable-authentication-in-sharded-cluster/)

### 设置RBAC访问
开启安全认证后根据自己的需要，在DB（甚至Collection中）创建用户、角色以支持各种安全级别的访问。记得，先创建角色，再创建用户，在创建普通用户前，先创建Admin用户。怎么玩，见下：

*  [Create a Role](http://docs.mongodb.org/manual/tutorial/define-roles/)
*  [Create a User Administrator](http://docs.mongodb.org/manual/tutorial/add-user-administrator/)
*  [Add a User to Database](http://docs.mongodb.org/manual/tutorial/add-user-to-database/)
*  [Collection-Level-Access Control](http://docs.mongodb.org/manual/core/collection-level-access-control/)

### 传输加密
前面提到过，设置MongoDB使用SSL来加密所有的传输加密数据，呵呵

* [Configure mongod and mongos for SSL](http://docs.mongodb.org/manual/tutorial/configure-ssl/)

### 限制网络访问
也就是仅允许特定的IP段进行访问，除了MongoDB的bindIp选项外，还需要通过操作系统防火墙来进行实现。

* [net.bindIp设置说明](http://docs.mongodb.org/manual/reference/configuration-options/#net.bindIp)
* [怎样通过IPTables让网络访问都不通](http://docs.mongodb.org/manual/tutorial/configure-linux-iptables-firewall/)
* [Windows防火墙就是个渣](http://docs.mongodb.org/manual/tutorial/configure-windows-netsh-firewall/)

### 审计系统活动
这个纯属蛋痛，毕竟你就算想审计，还得有企业版才能实现不是？

* [ Configure System Events Auditing](http://docs.mongodb.org/manual/tutorial/configure-auditing/)

### 加密并保护MongoDB数据
自己写代码加密传输中的数据，或者通过操作系统加密硬盘上的文件。怎么折腾怎么来。认证部分中，罗列了一些可以考虑的商业工具实现。

### 使用专用用户来安装并运行MongoDB
简单的说，就是：表用root啦

### 其它几项影响MongoDB安全的选项
* _-noscripting_ :Mongodb允许在Server端运行脚本来搞一些有的没的，比如说`mapreduce`,`group`, `eval`神马的，如果你不搞以上飞机，最好使用`-noscripting`选项来关闭这项功能。
* 在生产环境，建议仅使用`wire`接口来进行数据库访问，禁用掉http与Rest接口会增强MongoDB的安全性。所以[`net.http.enabled`](http://docs.mongodb.org/manual/reference/configuration-options/#net.http.enabled), [`net.http.JSONEnabled`](http://docs.mongodb.org/manual/reference/configuration-options/#net.http.JSONPEnabled),[`net.http.RESTInterfaceEnabled`](http://docs.mongodb.org/manual/reference/configuration-options/#net.http.RESTInterfaceEnabled)还是在设置文件中关掉吧。
* 开启[`wireObjectCheck`](http://docs.mongodb.org/manual/reference/configuration-options/#net.wireObjectCheck)选项，MongoDB会对输入进行检查，确定生成正确的BSON对象。嗯，话说，默认就是开启的。
