title:  Faux Pas 的 NDEBUG 警告与处理
date: 2016-04-12 10:30
tags: iOS, Xcode, NDEBUG, 编译器, 代码检查
---

# Faux Pas 的 NDEBUG 警告与处理

前段时间买了一个[Faux Pas](http://fauxpasapp.com/)来做代码风格检查。发现这厮真不友好啊，上来就给我一个大大的红色提示：

![Snip20160412_2](https://o8tbwvlyj.qnssl.com/images/1604/faux2.png)


Faux 会对所有的 Xcode 默认项目生成这条警告：
> Required compiler argument -DNDEBUG is not used. This argument disables the C standard library assertion macro (as defined in assert.h).

添加此选项是为了防止在 Release 环境中 assert() 被执行。Xcode 与 其它 C/C++ IDE 不同，没有自动在 Release 模式时添加 -DNDEBUG。需要手工添加：

在 "Target" -> "Build Setting" -> "Other C Flags" 中的 "Relase" Model中添加 "-DNDEBUG"

![Snip20160412_1](https://o8tbwvlyj.qnssl.com/images/1604/faux1.png)


### 补充

Faux Pas 认为直接加到`other link flags`中也不是好办法，正确的玩法应该是添加到`Preprocessor Macros`，如下图所示：

![Xcode截图](https://o8tbwvlyj.qnssl.com/images/1604/faux3.png)