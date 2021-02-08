---
title: Ubuntu20以添加Service的方式添加开机启动脚本
categories:
  - ubuntu
tags:
  - ubuntu
  - 开机启动
  - service
---

# Ubuntu20以添加Service的方式添加开机启动脚本

不同于以往的版本，***ubuntu18/20*** 默认不带`/etc/rc.local`文件，我们需要通过配置 ==service== 来使开机启动脚本生效。

***ubuntu18*** 和 ***ubutnu20*** 不再使用 ==initd== 管理系统，改用 ==systemd==，包括用 ==systemctl== 命令来替换了 ==service== 和 ==chkconfig== 的功能。

==systemd== 默认读取 `/etc/systemd/system` 下的配置文件，该目录下的文件会链接 `/lib/systemd/system/` 下的文件。

### 一，新建一个 `test.service`

在 `/lib/systemd/system/` 下，新建一个文件 `test.service` 这个配置文件最好自己手输入一下，我直接复制进去会提示出错，估计是编码的问题。

```shell
# 新建 service 文件
sudo vim /lib/systemd/system/test.service
# 填入以下内容：

[Unit]
Description=test
#若需要联网后启动的话，则需要加入该参数
After=network.target 

[Service]
Type=forking
ExecStart=/bin/bash /path/to/test.sh #执行的内容是脚本test.sh中的内容，其中包括它的绝对地址

[Install]
WantedBy=multi-user.target
```

### 二，创建 test.sh

新建一个测试脚本：

```shell
#!/bin/bash

echo `date`,"ok" >>/tmp/test.log
```

赋予脚本执行权限，路径根据实际情况对应修改。

`chmod +x /path/to/test.sh` 

### 三，启动服务，设置开机启动

`systemctl enable test.service`

查看启动状态：

`systemctl status test.service`

## 四，service 文件参数说明

```shell
[Unit]
# Unit 服务的说明
Description=Your service desctuption
# Description 描述服务
After=network.target
# After 在运行此服务前，先执行什么服务
RefuseManualStart=no(默认)
# RefuseManualStart, RefuseManualStop 如果设为yes，那么此单元将拒绝被手动启动(RefuseManualStart=) 或拒绝被手动停止(RefuseManualStop=)。 也就是说，此单元只能作为其他单元的依赖条件而存在，只能因为依赖关系而被间接启动或者间接停止，不能由用户以手动方式直接启动或者直接停止。设置此选项是为了禁止用户意外的启动或者停止某些特定的单元。 
Documentation=man:systemd.special(7)
# 对此详细说明的文档，man手册systemd的第7部分

[Service]
# Service 服务运行参数的设置
PIDFile=/run/YourServiceName.pid
# PIDFile 服务进程文件（以及存放位置）
Type=forking
# 取值范围: simple, exec, forking, oneshot, dbus, notify, idle
# forking是后台运行的形式, oneshot是一次运行，若启动失败为0则失败
ExecStart
# ExecStart 绝对路径 服务的具体运行命令
ExecReload
# ExecReload 绝对路径 重启命令
ExecStop
# ExecStop 绝对路径 停止命令
PrivateTmp=True
# 表示给服务分配独立的临时空间
KillMode=control-group
# KillMode 取值范围: control-group(默认)(主子进程全杀), process, mixed, none
TimeoutSec=5
# 一个同时设置 TimeoutStartSec=(最大启动时间) 与 TimeoutStopSec=(最大停止时间) 的快捷方式。超时则强制关闭
RemainAfterExit=no
# 当该服务的所有进程全部退出之后， 是否依然将此服务视为活动(active)状态。 默认值为 no

[Install]
# 服务安装的相关设置，可设置为多用户
WantedBy=multi-user.target
# WantedBy 执行 systemctl enable 启用命令之后，将会建立一个指向该单元文件的软链接 /etc/systemd/system/multi-user.target.wants/foo.service，表示将 foo.service 包含到 multi-user.target 目标中，这样，当启动 multi-user.target 目标时， 将会自动起动 foo.service 服务。同时，systemctl disable 命令 将会删除这个软连接。
```



参考 ：[https://blog.csdn.net/qq_42346414/article/details/110859280](https://blog.csdn.net/qq_42346414/article/details/110859280)

