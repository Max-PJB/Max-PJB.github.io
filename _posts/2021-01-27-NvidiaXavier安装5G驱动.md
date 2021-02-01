---
title: NvidiaXavier安装5G驱动
categories:
  - xxx
tags:
  - xxx
  - xxx
  - xxx
---

第一次在 arm64 的系统上安装驱动，炒鸡紧张。

## 集成FIBOCOM模块option驱动信息

### 配置Linux内核option串口驱动

串口驱动需要配置Linux内核，配置方法如下：
1. cd kernel进入到内核根目录；

`cd /usr/src/linux-headers-4.9.140-tegra-ubuntu18.04_aarch64/kernel-4.9`

1. 输入“make menuconfig”后回车；

```shell
nvidia@nvidia:/usr/src/linux-headers-4.9.140-tegra-ubuntu18.04_aarch64/kernel-4.9$ make menuconfig
  HOSTCC  scripts/kconfig/mconf.o
<command-line>:0:12: fatal error: curses.h: No such file or directory
compilation terminated.
scripts/Makefile.host:118: recipe for target 'scripts/kconfig/mconf.o' failed
make[1]: *** [scripts/kconfig/mconf.o] Error 1
Makefile:565: recipe for target 'menuconfig' failed
make: *** [menuconfig] Error 2
```

网上一查，需要需要安装 ==libncurses5-dev== 这个软件，但是安装这个软件前呢，需要安装 ==libtinfo-dev== 。找到了这个人写的文章 [https://blog.csdn.net/isco22/article/details/89846696](https://blog.csdn.net/isco22/article/details/89846696) ，他给的链接里面也没得下载。最后是在这个网站找到了资源 [https://pkgs.org/download/libtinfo-dev](https://pkgs.org/download/libtinfo-dev) ，下载我们需要的包，然后通过 `dpkg -i` 离线安装。

![image-20210127204427945](http://cdn.ailemong.com/2021-01-01/27/21-20-44-29.png)

![image-20210127204549778](http://cdn.ailemong.com/2021-01-01/27/21-20-45-50.png)

```shell
nvidia@nvidia:~/Downloads$ wget http://ports.ubuntu.com/pool/main/n/ncurses/libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb
--2021-01-27 20:36:41--  http://ports.ubuntu.com/pool/main/n/ncurses/libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb
Resolving ports.ubuntu.com (ports.ubuntu.com)... 91.189.88.152, 91.189.88.142, 2001:67c:1360:8001::23, ...
Connecting to ports.ubuntu.com (ports.ubuntu.com)|91.189.88.152|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 74232 (72K) [application/x-debian-package]
Saving to: ‘libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb’

libtinfo-dev_6.1-1ubuntu1.18.04_arm64.de 100%[===============================================================================>]  72.49K  57.4KB/s    in 1.3s    

2021-01-27 20:36:43 (57.4 KB/s) - ‘libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb’ saved [74232/74232]

nvidia@nvidia:~/Downloads$ ls
'FM150 Series.rar'   jetson-xavier-opencv   libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb   libtinfo-dev_6.1-1ubuntu1_arm64.deb
nvidia@nvidia:~/Downloads$ sudo dpkg -i libtinfo-dev_6.1-1ubuntu1
libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb
nvidia@nvidia:~/Downloads$ sudo dpkg -i libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb 
(Reading database ... 144330 files and directories currently installed.)
Preparing to unpack libtinfo-dev_6.1-1ubuntu1.18.04_arm64.deb ...
Unpacking libtinfo-dev:arm64 (6.1-1ubuntu1.18.04) over (6.1-1ubuntu1) ...
Setting up libtinfo-dev:arm64 (6.1-1ubuntu1.18.04) ...
nvidia@nvidia:~/Downloads$ 
```



1. 在弹出的界面中依次选择：device drivers -> USB support - > USB serial convertesupport

- [ ] 崩溃了，make menuconfig 总是报错。提示没有 eqos/Kconfig

```

```

