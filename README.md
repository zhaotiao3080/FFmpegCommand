
![FFmpegCommand](images/ffmpeg-command.png)


## 前景提要
在我们的开发中，经常会用到音视频相关内容，一般我们都会选择[FFmpeg](https://www.ffmpeg.org/)，但是其交叉编译对于我们来说是一件很麻烦的事情．所以这里方便日后使用就编写了这个`FFmpegCommand`，`FFmpegCommand`是由`FFmpeg`核心库，并且集成了`lame`、`libx264`和`fdk-aac`主流音视频处理程序构成的Android程序

**注意：当前库只适用于Android**

## 主要功能
[ ![Download](https://api.bintray.com/packages/sourfeng/repositories/ffmpeg/images/download.svg) ](https://bintray.com/sourfeng/repositories/ffmpeg/_latestVersion)[![License](https://img.shields.io/badge/license-Apache%202-success.svg)](https://www.apache.org/licenses/LICENSE-2.0)[ ![FFmpeg](https://img.shields.io/badge/FFmpeg-4.2.1-orange.svg)](https://ffmpeg.org/releases/ffmpeg-4.2.1.tar.bz2)[ ![X264](https://img.shields.io/badge/X264-20191217.2245-yellow.svg)](http://download.videolan.org/pub/videolan/x264/snapshots/x264-snapshot-20191217-2245-stable.tar.bz2)[ ![mp3lame](https://img.shields.io/badge/mp3lame-3.100-critical.svg)](https://sourceforge.net/projects/lame/files/latest/download)[ ![fdk-aac](https://img.shields.io/badge/fdkaac-2.0.1-ff69b4.svg)](https://downloads.sourceforge.net/opencore-amr/fdk-aac-2.0.1.tar.gz)

* **支持所有FFmpeg命令**
* **支持视频格式转换 mp4->flv**
* **支持音频编解码 mp3->pcm pcm->mp3 pcm->aac**
* **支持视频编解码 mp4->yuv yuv->h264**
* **支持音视频的剪切、拼接**
* **支持视频转图片 mp4->png mp4->gif**
* **支持音频声音大小控制以及混音（比如朗读的声音加上背景音乐）**
* **支持部分滤镜 音频淡入、淡出效果、视频亮度和对比度以及添加水印**
* **支持获取媒体文件信息**

|执行FFmpeg|获取媒体信息|
|---------| ----------------------------------| 
|<img src="images/1.gif" alt="图-1：命令行展示" width="260px" />|<img src="images/2.gif" alt="图-2：命令行执行" width="260px"/>|


## 引入

下面两种引入只选择一种即可,并根据最新版本替换下面的`latestVersion`，当前最新版本[ ![Download](https://api.bintray.com/packages/sourfeng/repositories/ffmpeg/images/download.svg) ](https://bintray.com/sourfeng/repositories/ffmpeg/_latestVersion)

```groovy
// 全部编解码-体积较大
implementation 'com.coder.command:ffmpeg:${latestVersion}'
// 部分常用编解码-体积较小,比上面引入减少大约6M
implementation 'com.coder.command:ffmpeg-mini:${latestVersion}'
```

**如果没有特别的编解码需求,强烈推荐建议使用`ffmpeg-mini`**

## 使用

下面只展示部分使用，其他可以参考 **[【WIKI】](https://github.com/AnJoiner/FFmpegCommand/wiki)**

### FFmpegCommand方法

|方法 |功能 |
|----|----|
|FFmpegCommand->setDebug(boolean debug)|是否Dubug模式，可打印日志，默认true|
|FFmpegCommand->runSync(final String[] cmd)|同步执行ffmpeg命令，外部需添加延时线程|
|FFmpegCommand->runSync(final String[] cmd, OnFFmpegCommandListener listener)|同步执行ffmpeg命令，并回调 完成，取消，进度|
|FFmpegCommand->runAsync(final String[] cmd, IFFmpegCallBack callBack)|异步执行，外部无需添加延时线程，并回调 开始，完成，取消，进度|
|FFmpegCommand->getInfoSync(String path,@Attribute int type)|获取媒体信息，type值必须为`@Attribute`中注解参数|
|FFmpegCommand->FFmpegCommand.exit()| 退出当前ffmpeg执行|

### 使用runAsync
以`runAsync`调用`FFmpeg`为异步方式，不需要单独开启子线程。强烈建议使用此方法进行音视频处理!!!   
直接调用`FFmpegCommand.runAsync(String[] cmd, IFFmpegCallBack callback)`方法，其中第一个参数由`FFmpegUtils`工具类提供，也可以自己添加

```java

final long startTime = System.currentTimeMillis();

FFmpegCommand.runAsync(FFmpegUtils.cutAudio(input, "00:00:30", "00:00:40", output),
    new CommonCallBack() {
         @Override
         public void onComplete() {
         Log.d("FFmpegTest", "run: 耗时：" + (System.currentTimeMillis() - startTime));

         @Override
         public void onCancel() {
             Log.d("FFmpegTest", "Cancel");
         }

         @Override
         public void onProgress(int progress) {
             Log.d("FFmpegTest",progress+"");
         }
    }
});

```
### 自定义FFmpeg命令

这里只是演示了音频剪切，很多如上述功能请自行查阅[FFmpegUtils](https://github.com/AnJoiner/FFmpegCommand/blob/master/ffmpeg/src/main/java/com/coder/ffmpeg/utils/FFmpegUtils.java)
如果其中不满足需求，可添加自己的FFmpeg命令．例如：

```java
String cmd = "ffmpeg -y -i %s -vn -acodec copy -ss %s -t %s %s";
String result = String.format(cmd, input, "00:00:30", "00:00:40", output);

FFmpegCommand.runAsync(result.split(" "), new CommonCallBack() {
     @Override
     public void onComplete() {
         Log.d("FFmpegTest", "run: 耗时：" + (System.currentTimeMillis() - startTime));
     }

     @Override
     public void onCancel() {
         Log.d("FFmpegTest", "Cancel");
     }

     @Override
     public void onProgress(int progress) {
         Log.d("FFmpegTest",progress+"");
     }
})
```

### 取消执行
执行下面方法后将会回调 `CommonCallBack->onCancel()` 或 `OnFFmpegCommandListener->onCancel()` 方法

```java
FFmpegCommand.exit();
```

**[【其他方法】](https://github.com/AnJoiner/FFmpegCommand/wiki/%E4%BD%BF%E7%94%A8)**

**[【功能详解】](https://github.com/AnJoiner/FFmpegCommand/wiki/%E8%AF%A6%E7%BB%86%E5%8A%9F%E8%83%BD)**

**[【常见问题】](https://github.com/AnJoiner/FFmpegCommand/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)**

**[【版本更新】](UPDATE.md)**

## 参考

* Java 使用请参考 [FFmpegCommandActivity](app/src/main/java/com/coder/ffmpegtest/ui/FFmpegCommandActivity.java)
* Kotlin使用请参考 [KFFmpegCommandActivity](app/src/main/java/com/coder/ffmpegtest/ui/KFFmpegCommandActivity.kt)

## 自定义编码器

因为引入了`LAME`，我们其实可以使用它自定义音频编码器，将其编译成so文件提供使用，具体参考[【WIKI-自定义MP3编码器】](https://github.com/AnJoiner/FFmpegCommand/wiki/%E8%87%AA%E5%AE%9A%E4%B9%89MP3%E7%BC%96%E7%A0%81%E5%99%A8)

## 体验交流

| 扫码下载｜[点击下载](http://fir.readdown.com/nfyz)  | 交流|微信赞赏|
| :--------:   |:--------:   |:--------:   |
| <img src="images/qr-code.png" alt="图-4 Demo下载" width="260px" />| <img src="images/ffmpeg-qq.jpg" alt="图-4 Demo下载" width="260px" />  | <img src="images/zan.jpg" alt="图-5 赞赏" width="260px" />|

## Start

撸码不易，如果觉得对您有所帮助，给个Start支持一下吧！


## License
```
Copyright 2019 AnJoiner

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
