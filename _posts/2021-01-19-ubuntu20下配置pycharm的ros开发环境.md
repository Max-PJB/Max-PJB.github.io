---
title: ubuntu20下配置pycharm的ros开发环境
categories:
  - SWU-Car
tags:
  - swu-car
  - ubuntu
  - pycharm
  - ros
  - rospy
---

# ubuntu20下配置pycharm的ros开发环境

### 一，*roslib* 的导入

直接使用 *pycharm* 打开 *ros* 工程的时候，是会提示找不到包，也没有*API*提示，写的很糟心：

![no ros lib](/public/img/20210119083933231.png)

就其原因，就是没有把 *roslib* 加入到 *pycharm*的环境中：我们在终端中是执行过 `~/.bashrc` 里面的那句 `source /opt/ros/noetic/setup.bash` 的，但是我们是通过快捷方式启动 *pycharm*的，并没有打开终端，所以也就没有*roslib*啦～

#### 解决办法：修改 *pycharm* 的桌面快捷方式

```shell
# 首先打开终端，然后输入：
sudo vim /usr/share/applications/jetbrains-pycharm-ce.desktop 
# 然后修改其中的 Exec 变量，在‘=’后面添加bash -i -c，改完如下：
Exec=bash -i -c "/home/swucar/software/pycharm-community-2020.3.2/bin/pycharm.sh; sleep 5" %f
# 保存并退出。 添加 bash -i -c 是为了在通过快捷方式启动PyCharm的同时加载~/.bashrc中的ROS环境变量。
```

然后重启 *pycharm*。

