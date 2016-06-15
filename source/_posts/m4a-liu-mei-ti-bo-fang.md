date: 2016-03-23
title: m4a文件在iOS上的流媒体播放
Tags: m4a, mp4, iOS, Android
url: m4a-streaming-play-on-mobile-phone
---

### 故障

公司项目中有个语音录制与播放的功能，QA反馈有部分 Android 机型录制的音乐文件无法在 iOS 上进行播放。看了一下 iOS 的实现代码，iOS 的实现有考虑当文件较大时先下载再播放引起的延迟，是按流形式播放的。

找到出错的 m4a 文件，发现如果按下载播放形式，不会出现任何问题；按流形式边下边播，会报错：

> on-optimized formats not supported for streaming

### 原因

m4a 文件本质就是一个mp4文件，m4a仅是声明容器文件中仅含有音频内容。mp4和QuickTime®文件一样都是一种容器文件，类似XML那样由若干个"Atom"组成，每个 atom 文件中存放不同的meta、音频数据、视频数据、字幕等内容。而所谓 "optimize for streaming" 意指将文件中的 meta data 提到文件前部，这样播放器按流形式读取时，可以在获取音视频数据前获取具体的音视频编码信息。

mp4 和 QuickTime 可以使用 Appple 公司的 Atom Inspector 工具打开：

![Snip20160322_2.png](https://o8tbwvlyj.qnssl.com/images/1604/atom.png)

左边的001.m4a 文件是播放出错的 m4a 文件，可以看到，用于存入音频数据（AAC格式）的 mdata 牌ftyp 后，而存放 metadata 的 moov 处于文件的最后。

位于右边的文件是使用iTunes转换后的 m4a 文件，moov 段被移动到了文件前部，mdata 被放置于文件的最后。转换后的m4a可以正常按流形式进行播放。

### 解决

最初是想将文件转换放到服务端来进行，让它类似缩略图生成那样成为一种文件扩展服务。但是，考虑 CS 团队未必有时间马上开发这个扩展功能而项目又等着急用，最后我还是直接在 Android 中实现了这个 optimizing of Streaming 功能。

使用的是[mp4parser](https://github.com/sannies/mp4parser) Library。简单的将文件使用 `mp4parser` 复制一下音轨即可，因为项目中为用户录音，实际音轨只有一条，所以将那条音轨取出存入一个新文件即可。代码如下：

```java
        Movie originalMovie = MovieCreator.build(sourceM4a);
        Movie destMovie = new Movie();
        Track audioTrack = originalMovie.getTracks().get(0);
        destMovie.addTrack(audioTrack);
        Mp4Builder builder = new DefaultMp4Builder();
        Container container = builder.build(destMovie);
        container.writeContainer(new FileOutputStream(destM4a).getChannel());
```