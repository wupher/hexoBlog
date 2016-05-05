title: 使用 Docker 构建开发、测试环境
date: 2014-08-18
tags: docker
---

# 使用 Docker 构建开发、测试环境

## 目录
* [什么是 Docker](#anchor1)
* [Docker 的构成](#anchor2)
* [Docker 的安装](#anchor3)
* [Docker 的运行与退出](#anchor4)
* [Docker 端口映射](#anchor5)
* [宿主机硬盘挂载](#anchor6)
* [容器间存储共享](#anchor7)
* [镜像的导入与导出](#anchor8)
* [DockerFile](#anchor9)
* [应用场景](#anchor10)
* [现有缺陷](#anchor11)

docker 为开发人员和系统管理员提供了一个可供开发，分发(ship)和运行应用的平台。将Docker化的应用及其依赖环境不需要经过任何修改就可以分发到任何地方--提供给QA，团队成员或者分发到云平台中。这是使用Docker的一个很重要的目的。

## <a name="anchor1">什么是 Docker</a>
Docker的英文本意是码头工人，也就是搬运工，这种搬运工搬运的是集装箱（Container），集装箱里面装的可不是商品货物，而是任意类型的App，Docker把App（叫Payload）装在Container内，通过Linux Container技术的包装将App变成一种标准化的、可移植的、自管理的组件，这种组件可以在你的笔记本上开发、调试、运行，最终非常方便和一致地运行在生产环境下的各种云机房和服务器上。

![Docker 与传输虚拟机的对比](images/CVV.jpg)

Docker的核心底层技术是LXC（Linux Container），Docker在其上面加了薄薄的一层，添加了许多有用的功能。 [这篇stackoverflow](http://stackoverflow.com/questions/17989306/what-does-docker-add-to-just-plain-lxc)上的问题和答案很好地诠释了Docker和LXC的区别，能够让你更好的了解什么是Docker， 简单翻译下就是以下几点：

* Docker提供了一种可移植的配置标准化机制，允许你一致性地在不同的机器上运行同一个Container；而LXC本身可能因为不同机器的不同配置而无法方便地移植运行；
* Docker以App为中心，为应用的部署做了很多优化，而LXC的帮助脚本主要是聚焦于如何机器启动地更快和耗更少的内存；
* Docker为App提供了一种自动化构建机制（Dockerfile），包括打包，基础设施依赖管理和安装等等；
* Docker提供了一种类似git的Container版本化的机制，允许你对你创建过的容器进行版本管理，依靠这种机制，你还可以下载别人创建的Container，甚至像git那样进行合并；
* Docker Container是可重用的，依赖于版本化机制，你很容易重用别人的Container（叫Image），作为基础版本进行扩展；
* Docker Container是可共享的，有点类似github一样，Docker有自己的INDEX，你可以创建自己的Docker用户并上传和下载Docker Image；
* Docker提供了很多的工具链，形成了一个生态系统；这些工具的目标是自动化、个性化和集成化，包括对PAAS平台的支持等；

Docker 有什么用呢？以运维的角度来说，你的应用程序一般都需要特定版本的操作系统、应用服务器、 JDK 、数据库服务器，还可能需要调整配置文件和其他一些依赖关系。应用程序可能需要绑定到指定的端口和一定量的内存。这些运行应用程序所需要的组件和配置就是所说的应用程序操作系统。你当然可以写一个包含下载和安装这些组件的安装脚本。 Docker 简化了这个流程，通过创建一个包含应用程序和基础设施的镜像(image)，当作一个组件进行管理。这些镜像可以创建 Docker 容器(container)，容器运行在 Docker 提供的容器虚拟化平台上。

# <a name="anchor2">Docker 的构成</a>
Docker 有两个主要组件：

* Docker：开源的容器虚拟化平台
* Docker Hub：共享和管理 Docker 镜像的 Saas 平台

Docker 采用 Linux 容器 来提供隔离、沙箱、复制、资源限制、快照和其他的一些优势。详细信息的可以看 excellent piece at InfoQ on Docker Containers 了解。

镜像是 Docker 的“构建组件”，也是应用操作系统的只读模版。容器是从镜像创建出来的运行状态，是 Docker 的“运行组件”。容器是可以运行、启动、停止、移动和删除的。镜像保存的仓库是 Docker 的“分发组件”。

![Docker的镜像与容器](images/docker-arch.png)

Docker 按启动顺序包含两个组件：

* 服务端：运行在宿主机上，负责构建、运行和分发 Docker 容器等重要工作
* 客户端：Docker 二进制程序，接收用户的命令和服务程序进行通信

客户端可以和服务端运行在一台主机上，也可以在不同的主机上。服务端需要用 pull 命令从仓库中拉一个镜像下来。服务端可以从 Docker Hub 或者其他配置的仓库中下载镜像。服务端主机可以从仓库中下载和安装多个镜像。然后客户端就可以用 run命令 来启动容器。客户端与服务端通过socket或者REST API 进行通信。

## <a name="anchor3">Docker 的安装</a>
在 CentOS 中安装 Docker:
```shell
sudo yum -y install docker-io    #安装 docker
sudo service docker start			#启动 docker 服务
sudo chkconfig docker on        #如果需要 docker 服务为自启动
```

在Ubuntu/Debian中安装 Docker:
```shell
sudo apt-get udpate
sudo apt-get install docker.io
sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
sudo sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io #命令自动补全
```

其它操作系统的安装可以查看[官方文档](https://docs.docker.com/installation/)。

## <a name="anchor4">Docker的运行与退出</a>

在了解了Image和Container的概念后，我们可以开始下载一个Image，Docker的好处就是提供了一个类似github的Image仓库管理，你可以非常方便pull别人的Image下来运行，例如，我们可以下载一个CentOS Image：
```shell
sudo docker pull centos:centos6
```
这里 centos6是一个 tag，类似于 Git 的 tag，能过它来确定下载的 CentOS 的版本。下载完成后，执行```docker images```命令来列出你已经下载的 images：
![下载截图](images/docker-images.png)
下载之后，我们通过命令行来运行一个容器，命令很简单，例如我们想执行一个 shell 终端：
``` bash
sudo docker run -i -t centos:centos6 /bin/bash
```

默认情况下，docker 容器是不提供交互shell 的，也不提供标准输入。可以指定-i选项来提供交互，提供-t 选项来分配一个伪终端。
在 Shell 中你可以做你想做的任意操作，安装软件，编写程序，运行命令等。当你操作后想将结果保存，这时可以用 docker commit 命令将 Container 提交成 Image。哦，假如你这里还处在交互 shell 中，记得先使用 ``Ctrl+d`` 或者 ``exit`` 命令退出。
```bash
sudo docker ps -a 
```

首先执行 ps 命令查看容器ID
![容器 ID 截图](images/docker-ps.png)
然后使用 commit 命令将容器进行保存
```bash
sudo docker commit 851d custom/centos-aliyun
```

容器提交后，执行``sudo docker images``就能看到刚才提交的容器。

## <a name="anchor5">docker端口映射</a>
经常要在 Docker 中开启某些网络服务，需要将 docker 虚拟机的网络端口与宿主机端口连接起来。比如将 docker 中的8080端口映射到宿主机的80端口上：
```bash
sudo docker run -p 80:8080 custom/tomcat
```

## <a name="anchor6">宿主机硬盘挂载</a>
这也是常用功能之一，尤其是服务需要记录日志、保存文件等时候。
```bash
sudo docker run -i -t -v /host/dir:/container/path ubuntu /bin/bash
```

以上是把宿主机器的/host/dir 挂载到/container/path 路径上。
## <a name="anchor7">容器间共享存储</a>
主要借助于``-volumes-from``参数实现
```bash
COUCH1=$(sudo docker run -d -v /var/lib/couchdb shykes/couchdb:2013-05-03)
COUCH2=$(sudo docker run -d -volumes-from $COUCH1 shykes/couchdb:2013-05-03)
```

这个特性，让人有许多想像空间，比如，一个容器实例用于 Web 存储，另外两个实例用于 Web 请求，实现读写分离。
## <a name="anchor8">镜像的导入/导出</a>
方法1: 使用 save/load 命令来实现镜像的导入导出
```bash
sudo docker save IMAGENAME | bzip2 -9 -c>img.tar.bz2
#或者你喜欢 tar.gz
sudo docker save IMAGENAME > imageName.tar.gz
```

镜像导入功能使用 load 命令解压导入即可

```bash
sudo docker load < imageName.tar.gz
# 喜欢压缩的同学
bzip2 -d -c <img.tar.bz2 | sudo docker load
```

方法2： push/pull 将 image 文件推送到[Docker Hub]()上去。这种方法类似于 git。你可以在 Docker Hub 上建立自己的公有或者私有库，适用于远程分享。缺点是，有时 image 文件特别大，需要考虑网络带宽问题。
方法3： 搭建自己的私服repository，将 image 提交到私服，适用于企业网络。

## <a name="anchor9">Dockerfile</a>
在 Shell 脚本环境中一步步安装，低效且劳累。Docker 可以通过自定义 Dockerfile 实现自动化构建 docker 镜像的脚本，既方便分享也便于修改与模板化。
```bash
# VERSION 1.0.0
# 默认Centos，可以改成你需要的任意镜像
FROM centos
# 签名
MAINTAINER wupher "wupher@foxmail.com"

RUN echo 'We are running some # of cool things'
RUN yum update 
RUN yum install -y openssh-server
RUN mkdir -p /var/run/sshd
# 设置root ssh远程登录密码
RUN echo "root:123456" | chpasswd

RUN yum install -y mysql-server
RUN yum install -y java-1.7.0-openjdk

# 安装tomcat 等……

# 挂载硬盘，用于保存 log
VOLUME ["/var/log/", "/var/volume2"]

# 容器开放22端口
EXPOSE 22
# 容器开放 8080端口
EXPOSE 8080

# 设置Tomcat初始化运行，SSH终端服务器作为后台运行，这样 docker run image的时候，这些服务就自动启动了
ENTRYPOINT service tomcat start && /usr/sbin/sshd -D
```

完整的 DockerFiler，可以参考[官方文档](https://docs.docker.com/reference/builder/)。
## <a name="anchor10">应用场景</a>
Docker目前有以下应用场景：
* 测试：Docker 很适合用于测试发布，将 Docker 封装后可以直接提供给测试人员进行运行，不再需要测试人员与运维、开发进行配合，进行环境搭建与部署。
* 测试数据分离：在测试中，经常由于测试场景变换，需要修改依赖的数据库数据或者清空变动 memcache、Redis 中的缓存数据。Docker 相较于传统的虚拟机，更轻量与方便。可以很容易的将这些数据分离到不同的镜像中，根据不同需要随时进行切换。
* 开发：开发人员共同使用同一个 Docker 镜像，同时修改的源代码都被挂载到本地磁盘。不再因为环境的不同而造成的不同程序行为而伤透脑筋，同时新人到岗时也能迅速建立开发、编译环境。
* PaaS 云服务：Docker 可以支持命令行封装与编程，通过自动加载与服务自发现，可以很方便的将封装于 Docker 镜像中的服务扩展成云服务。类似像 Doc 转换预览这样的服务封装于镜像中，根据业务请求的情况随时增加和减少容器的运行数量，随需应变。
* 使用 Docker 来做分步式集群模拟

## <a name="anchor11">现有缺陷</a>
* 无法修改 hosts 文件，不能自己做域名解析。一种常用的方法是安装dnsmasq 。
* VM 的系统时间是 UTC 时间，貌似没办法修改。办法倒也不是没有，最常用的办法是将宿主机器的/etc/localtime 映射到镜像的/etc/localtime:ro 上去。但是这只能使镜像与宿主机器保持时区一致，假如希望不同的镜像使用不同时区，只有在每次启动时通过CMD 或者 ENTRPOINT命令来自动调整时区。
* 目前还没找到办法将 Container 的 IP 改成静态 IP，重启容器的时候，IP 可能会发生变化。
* 受限于 Lxc，外围环境必须为 Linux，而且内核版本必须大于2.6.27。
* 现在还不支持内存转储及运行状态导出。

Docker 现在才1.0版本，这些问题相信未来都将会解决。