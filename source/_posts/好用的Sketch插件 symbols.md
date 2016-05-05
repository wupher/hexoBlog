tags: Sketch, 设计
date: 2014-1-6
title: 好用的Sketch插件 symbols
---

# 好用的Sketch插件 symbols
[Sketch symbols](https://github.com/tisho/sketch-plugins/tree/master/Symbols)是[Sketch.app](http://bohemiancoding.com/sketch/)的一个好用插件。在使用Sketch做App或者Web设计的时候，总有某些对象是相互关联的，比如按钮、文字样式、输入框等。通过Symbols插件，你可以将逻辑相关的一组对象通过Symbol关联在一起，当其中的一个对象样式发生改变时，其它相关联的对象会一起更新。

使用起来很简单，为需要关联的Group name添加『:symbol-name』样式的声明：
![Symbol name](https://github-camo.global.ssl.fastly.net/a6e8747403d974cced2d3008918d1914a5d3bcbe/687474703a2f2f746973686f2e6769746875622e696f2f736b657463682d706c7567696e732f696d616765732f73796d626f6c2d6e616d652e706e67)    
如上图，OK-Buntton与cancel-button都是名为button-default样式的按钮，当其实某个按钮的样式发生改变，使用 > CMD+E 其它的按钮样式也会自动更新。

这时候可能会碰上一个麻烦。我们希望按钮的样式能自动同步，但是按钮的文字会不受影响。毕竟，按钮的文字内容本该就是各不相同的，同步过程中，不要自动更新文字内容。这时只要在text layer前面加一个$符号，symbol就会识别，同步的时候样式，字体、大小都会自动同步 ，但是文字的内容不受影响。    
![text layer](https://github-camo.global.ssl.fastly.net/14e6774c9b51e56bbc54a43d63b4cce3bd89e6dc/687474703a2f2f746973686f2e6769746875622e696f2f736b657463682d706c7567696e732f696d616765732f64796e616d69632d6c617965722d6e616d652e706e67)    



