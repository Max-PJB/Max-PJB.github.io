---
title: ubuntu18安装Anaconda配置清华源
categories:
  - ubuntu
tags:
  - ubuntu
  - anaconda
  - conda
---

# ubuntu18安装Anaconda配置清华源

## 下载安装

可以直接上 tuna 的网站上下载 anaconda， ubuntu 下载  [Anaconda3-5.3.1-Linux-x86_64.sh](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.3.1-Linux-x86_64.sh) 下载最新的版本号就行了。

安装很简单，进入 Dowloads 目录 , `bash Anaconda3-5.3.1-Linux-x86_64.sh` ，然后一路 ==yes== 就好了 

## 配置国内源

[清华源](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/) 这是国内清华的源，打开网址就有教你怎么配。

我这里复制粘贴一下。

ubuntu 用户：

`vim ~/.condarc`

> 如果没有这个文件，就 `conda config --set show_channel_urls yes` 生成一下

然后复制下面的内容替换 `.condarc` 里面的所有内容：

```bash
channels:
  - defaults
show_channel_urls: true
channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

保存退出后， 运行  `conda clean -i` 清除索引缓存，保证用的是镜像站提供的索引。

以上纯复制粘贴！:laughing:

剩下的就是 conda 命令了。

`conda create -n myenv python=3.6`  创建一个 python 3.6 的环境玩玩吧~！

`conda activate myenv` 激活 环境

`conda deactivate` 退出环境

