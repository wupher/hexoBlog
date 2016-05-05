title: Cocoapods on OSX 10.9 Mavericks
date: 2013-11-25
tags: iOS
---

# Cocoapods on OSX 10.9 Mavericks

升级到10.9后，偶然发现我的cocoapods居然不能用了。查了一下，发现Mavericks现在默认就是Ruby2.0了。本来我都是通过gem来安装pods，既然现在用了Xcode5，于是打算直接用[cocoapods-plugin](https://github.com/kattrali/cocoapods-xcode-plugin)来安装了。

没想到安装失败，显示：
>	ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
    
估计是权限问题，于是直接动手改了目录权限：
>	sudo chown -R userName /Library/Ruby/Gems/2.0.0/

结果可气的是后面编译生成pod可执行文件又出错，只好把/usr/bin也折腾一把：
>	sudo chown -R /usr/bin

OK，总算正常了。

---
好吧，千万别这么干！
尤其最后一条，居然把bin目录下的文件全改了，会造成terminal启动之后直接异常退出。还好机器上有fish。修复的话，通过『实用工具』中的『磁盘工具』来进行『验证磁盘权限』和『修复磁盘权限』操作后恢复。

在cocoapods plugin更新之前，我还是继续使用命令行吧。
