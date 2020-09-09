---
title: 安装opencv-python出现imshow使用缺少gtklib的错误
categories:
  - opencv
tags:
  - opencv
  - python
  - gtklib
---

# 安装opencv-python出现imshow使用缺少gtklib的错误
```shell
OpenCV Error: Unspecified error (The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script) in cvNamedWindow, file /wrk/xbj_vdi/ryanw/opencv_mine/build03/opencv-2.4.5/modules/highgui/src/window.cpp, line 479
terminate called after throwing an instance of 'cv::Exception'
  what():  /wrk/xbj_vdi/ryanw/opencv_mine/build03/opencv-2.4.5/modules/highgui/src/window.cpp:479: error: (-2) The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script in function cvNamedWindow
```

解决办法：

1. 按照提示安装 libgtk2.0-dev 和pkg-config
`sudo apt install libgtk2.0-dev pkg-config`

2. 卸载原来的opencv
`pip3 uninstall opencv-python`

3. 升级 pip3
`pip3 install --upgrade pip`

4. 安装opencv
`pip3 install opencv-python`

