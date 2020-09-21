---
title: 2020-09-21-mmpose在ubuntu18中的安装
categories:
  - human pose
tags:
  - ubuntu
  - human pose
  - mmpose
  - anaconda
---

# mmpose在ubuntu18中的安装

mmpose 里面打包好了很多工具包，其中我想用的就是它的开箱即用的 HrNet 

## 安装 CUDA

这个没啥好说的，就按照官网的一路操作下去了。 [CUDA下载地址](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal) ，选择对应的版本，可以手动下载，也可以根据下面的 bash 命令安装

![image-20200921103502579](/public/img/image-20200921103502579.png)

![image-20200921103557504](/public/img/image-20200921103557504.png)

如果想配置多 cuda 版本切换，网上也有教程。大致思路的就是配置一个 环境变量指向一个 软链接 ， 这个软链接指向哪个 cuda 版本的目录就表示当前环境使用的是哪个 版本。

## 安装 Anaconda

这里参考我另一篇文章，介绍使用 清华源 下载安装 Anaconda 然后配置他的源。



