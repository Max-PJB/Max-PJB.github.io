---
title: ubunt18安装ros-melodic
categories:
  - ros
tags:
  - ros
  - ubuntu
  - 安装
---

[TOC]

# ubunt18安装ros-melodic

## 1. 添加ros的清华源

清华镜像源ros[使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/ros/) 下面是原文照抄。

新建 `/etc/apt/sources.list.d/ros-latest.list`，内容为：

```bash
deb https://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ bionic main
```

然后再输入如下命令，信任ROS的GPG Key，并更新索引：

```bash
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt update
```

![image-20200925154502724](/public/img/image-20200925154502724.png)

## 2. 安装 ros

[按照官网来](http://wiki.ros.org/melodic/Installation/Ubuntu) ，上面都是添加源，我们上面就是做这个，只是添加的是国内的，速度更快。

我们可以直接跳到安装这一步了：

`sudo apt install ros-melodic-desktop-full` 

这里使用的是我们上面配置的清华源，应该不会有什么问题，速度也飞快。

## 3. Environment setup

把 ros 加入到环境变量

```bash
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

> 官网上有说，如果你装了很多个 ros，想用哪个就 source 哪个 `source /opt/ros/melodic/setup.bash` 

## 4. Dependencies for building packages

这也是官网的，安装一些包，方便管理ros。

```bash
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
```

这一步安装也是飞快。

## 5.  安装 rosdep

```bash
sudo apt install python3-rosdep
# 初始化
sudo rosdep init
rosdep update
```

#### 初始化 rosdep 的时候报错：

1. `DistributionNotFound: The 'rosdep==0.19.0' distribution was not found and is required by the application` 

2. ~~~bash
   ERROR: cannot download default sources list from:
   https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
   Website may be down.
   ~~~

3. `rosdep update` 卡死

#### 解决办法

1. 重新安装 Python3 版本的 rosdep

```
sudo apt install python3-catkin-pkg python3-rosdistro python3-rospkg python3-rosdep-modules python3-rosdep
```

> 此处可能会提示依赖，按提示安装所有依赖即可

2. 想办法翻墙吧 :laughing: 

![image-20200925174033484](/public/img/image-20200925174033484.png)

3. `rosdep update` 我是翻墙了也要尝试多次，卡死了就重来吧。若出现time out 或错误，重试几遍

![image-20200925183027509](/public/img/image-20200925183027509.png)



## 6. 测试

运行小海龟：

1. 打开一个新的Terminal： 执行 `roscore`

如果出现错误：

~~~bash
pengjb@pengjb-ThinkPad-W530:~$ roscore

Command 'roscore' not found, but can be installed with:

sudo apt install python-roslaunch
~~~

解决：

~~~bash
cd /opt/ros/melodic/bin
ls
# 这里缺少很多文件，没有 roscore
sudo apt-get install ros-melodic-desktop
ls
# 有了！
~~~

2. 再打开一个终端

   `rosrun turtlesim turtlesim_node`

3. 再新建一个终端

   `rosrun turtlesim turtle_teleop_key`

   选中第三个终端键盘上下左右控制海龟

   ![img](/public/img/20180620151022899)

   ## 7. ros常用命令

   ~~~bash
   计算图
   
   rqt_graph
   
   查看节点
   
   rosnode list
   
   查看节点具体信息
   
   rosnode info /turylesim
   
   查看话题
   
   rostopic list / turtle/cmd_vel
   
   监听话题
   
   rostopic echo  / turtle/cmd_vel
   
   查看话题具体信息
   
   rostopic info 
   
   发布命令，命令小海龟
   
   rostopic pub  / turtle/cmd_vel
   
   -r 10频率
   
   rostopic pub -r 10 / turtle/cmd_vel
   
   服务列表
   
   rosservice list
   
   新生一只小海龟
   
   rosservice call /spawn "x:5.0 y:5.0 theta:0.0 name:'turtle2'"
   ~~~

   