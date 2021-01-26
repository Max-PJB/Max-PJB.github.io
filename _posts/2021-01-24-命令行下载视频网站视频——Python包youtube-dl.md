---
title: 命令行下载视频网站视频——Python包youtube-dl
categories:
  - 视频下载
tags:
  - youtube-dl
  - Python
  - ffmpeg
---

# 命令行下载视频网站视频——Python包youtube-dl

我的环境是 Ubutnu20.04

### 一，安装 *ffmpeg* 和 *youtube-dl* 

`sudo apt install ffmpeg` 

`pip3 install youtube-dl`

### 二，使用

输入命令查看可下载的内容但不下载：

`youtube-dl -F 视频页面地址`

例如我准备下载大魔王谭晶的《九儿》，输入

`youtube-dl -F  https://www.youtube.com/watch?v=NJt8tkSY2wo`

![](public/img/image-20210124174915595.png)

然后输入命令开始下载:

`youtube-dl -f 文件编号[+文件编号+文件编号……]  视频地址`

例如我的输入是：

`youtube-dl -f 137+140  https://www.youtube.com/watch?v=NJt8tkSY2wo`

也可以输入:

`youtube-dl -f bestvideo+bestaudio https://www.youtube.com/watch?v=x0uinJvhNxI`

```shell
# check list
# youtube-dl -F https://www.youtube.com/watch?v=x0uinJvhNxI
[youtube] x0uinJvhNxI: Downloading webpage
[info] Available formats for x0uinJvhNxI:
format code  extension  resolution note
249          webm       audio only tiny   63k , opus @ 50k (48000Hz), 123.69MiB
250          webm       audio only tiny   75k , opus @ 70k (48000Hz), 154.79MiB
251          webm       audio only tiny  130k , opus @160k (48000Hz), 275.97MiB
140          m4a        audio only tiny  150k , m4a_dash container, mp4a.40.2@128k (44100Hz), 319.05MiB
160          mp4        256x144    144p  118k , avc1.4d400c, 30fps, video only, 30.87MiB
278          webm       256x144    144p  205k , webm container, vp9, 30fps, video only, 68.27MiB
242          webm       426x240    240p  262k , vp9, 30fps, video only, 92.25MiB
133          mp4        426x240    240p  271k , avc1.4d4015, 30fps, video only, 52.00MiB
243          webm       640x360    360p  432k , vp9, 30fps, video only, 172.66MiB
134          mp4        640x360    360p  680k , avc1.4d401e, 30fps, video only, 106.63MiB
244          webm       854x480    480p  736k , vp9, 30fps, video only, 267.36MiB
135          mp4        854x480    480p 1241k , avc1.4d401f, 30fps, video only, 176.36MiB
247          webm       1280x720   720p 1284k , vp9, 30fps, video only, 497.02MiB
136          mp4        1280x720   720p 1871k , avc1.4d401f, 30fps, video only, 290.41MiB
248          webm       1920x1080  1080p 2332k , vp9, 30fps, video only, 872.66MiB
137          mp4        1920x1080  1080p 3200k , avc1.640028, 30fps, video only, 453.62MiB
22           mp4        1280x720   720p  246k , avc1.64001F, 30fps, mp4a.40.2@192k (44100Hz)
18           mp4        640x360    360p  262k , avc1.42001E, 30fps, mp4a.40.2@ 96k (44100Hz), 647.47MiB (best)

# actual downlaod 
youtube-dl -f bestvideo+bestaudio https://www.youtube.com/watch?v=x0uinJvhNxI
[youtube] x0uinJvhNxI: Downloading webpage
[download] Resuming download at byte 337521534
[download] Destination: Flutter Crash Course for Beginners 2020 - Build a Flutter App with Google's Flutter & Dart-x0uinJvhNxI.f137.mp4
[download] 100% of 453.62MiB in 02:08
[download] Destination: Flutter Crash Course for Beginners 2020 - Build a Flutter App with Google's Flutter & Dart-x0uinJvhNxI.f140.m4a
[download] 100% of 319.05MiB in 05:08
[ffmpeg] Merging formats into "Flutter Crash Course for Beginners 2020 - Build a Flutter App with Google's Flutter & Dart-x0uinJvhNxI.mp4"
Deleting original file Flutter Crash Course for Beginners 2020 - Build a Flutter App with Google's Flutter & Dart-x0uinJvhNxI.f137.mp4 (pass -k to keep)
Deleting original file Flutter Crash Course for Beginners 2020 - Build a Flutter App with Google's Flutter & Dart-x0uinJvhNxI.f140.m4a (pass -k to keep)
```

### 三，遇到报错

1. 遇到 `You have requested multiple formats but ffmpeg or avconv are not installed. The formats won't be merged.`

   ​       这是因为没有安装 *ffmpeg*，安装就好

2.  -[] 

