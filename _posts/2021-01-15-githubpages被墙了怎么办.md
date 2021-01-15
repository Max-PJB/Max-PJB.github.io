---
title: githubpages被墙了怎么办
categories:
  - github
tags:
  - ubuntu
  - linux
  - github
---

# githubpages被墙了怎么办

我的*github pages*无法访问，网上一查是 域名 被干了，无法解析。

#### 步骤一，查询自己 *githubpages* 的解析地址

这个可以去站长工具去查， [http://tool.chinaz.com/](http://tool.chinaz.com/) 

比如我的地址是 *jackypjb.github.io* 

![image-20210115150359056](/public/img/image-20210115150359056.png)

#### 步骤二，修改 *hosts* 文件

这个时候需要修改 hosts：

```shell
# 修改 hosts 文件
vim /etc/hosts
# 添加一行 185.199.111.153 jackypjb.github.io
# ip 是上面查询出来的，需要根据自己的情况相应修改
```

![image-20210115150526980](/public/img/image-20210115150526980.png)

#### 步骤三，修改 DNS

这一步可做可不做，但是大家都是建议修改 DNS 为免费的共用 DNS的，这个 *baidu* 一搜很多，我用的是 *114.114.114.114*  