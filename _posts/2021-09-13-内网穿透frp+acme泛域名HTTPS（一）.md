---
title: 内网穿透frp+acme泛域名HTTPS（一）
categories:
  - frp
tags:
  - frp
  - 内网穿透
  - 泛域名
  - HTTPS
---

# 内网穿透frp+acme泛域名HTTPS（一）

> 需求：外网访问，https访问（微信小程序）

碍于各种云服务器厂商的高配置版本都太贵了，想使用一个微服务或者用一个GPU做人工智能处理作为服务器就智能在本地搭建服务平台，然后让本地的服务通过内网穿透来给公网用户使用，主要通过微信小程序https访问。

## （一）内网穿透

> 如果你的局域网能够申请到公网IP，固定自然不必讲，动态公网IP可以试用 DDNS-GO 更方便一些。

我们采用 frp 的方案进行内网穿透。

### 准备工作

1. 一个域名（已备案，以下使用腾讯云，ailemong.com）
2. 一台云服务器（简配的云服务器还是不贵的，使用腾讯云，云服务器一般都会有一个公网IP，假设我这里是12.34.56.78）
3. 一台可以上网的电脑（废话）
4. 下载 [frp](https://github.com/fatedier/frp/releases) 。（[frp](https://github.com/fatedier/frp/releases) 分为客户端和服务器，都在一个压缩包里，不同的系统使用对应的压缩包里的部分)

### 配置 frp

##### 配置服务端

进入解压后的frp文件夹，编辑解压后文件夹内服务端的配置文件 `frps.ini`：

```bash
[common]
bind_port = 7000  #监听frp客户端的端口
vhost_http_port = 8080 #vhost端口，将这个端口来的请求转发给客户端
subdomain_host = frp.ailemong.com # 设置 host，在客户端配置中就可以试用 subdomain 来配置子域名，避免写太多custom_domains

```

启动服务端，命令：`./frps -c frps.ini`

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-19-28-28.png)

##### 配置客户端

在本地的电脑上，同样下载一份frp的二进制源码压缩包，修改客户端配置文件`frpc.ini`:

```bash
[common]
server_addr = 12.34.56.78 # 这里需要填腾讯云服务器的公网IP
server_port = 7000 # 这里需要和服务端的监听端口保持一致

[web]
type = http
local_port = 80 # 本地服务提供的端口
custom_domains = frp.ailemong.com # 注意： 只有通过这个url来的请求会被处理，使用 ip 地址是没有用的！ 

```

启动客户端，命令：`./frpc -c frpc.ini`

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-19-38-47.png)

更多 frp 的文档可以参考 [FRP](https://gofrp.org/docs/concepts/)。

### 在云服务器上配置 nginx

##### 修改配置文件

我这里使用的ubuntu系统，nginx的配置文件在 `/etc/nginx/`下。

可以在 `conf.d`目录下新建一个`frp_ssl.conf`（以 .conf 结尾，名字随意）：

```bash
server {
 listen 80;
 server_name frp.ailemong.com;
 proxy_set_header Host $host; # 需要，否则无法转发
 proxy_set_header X-Forwarder-For $remote_addr;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection "Upgrade";

 location / {
      proxy_pass http://127.0.0.1:8080; #端口号一定要和frps.ini的vhost_http_port一致
    }
}

```

然后重启 nginx，命令：`sudo service nginx restart` 或者重新加载配置也行，命令： `sudo nginx -s reload`，命令成功没有任何输出，有输出就是报错了。

### 腾讯云修改域名解析

前面我将 frp.ailemong.com 来的请求转发来本地服务商，但是 公网上并没有关于 frp.ailemong.com 的 DNS 解析，所以需要去腾讯云控制台上增加一条 解析 A 记录。腾讯云使用的是 DNSPOD 。

进入控制台（注意是 DNSPOD控制台，不是腾讯云的），添加记录，记录类型选择`A`，记录值就是云服务器的公网IP。

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-19-54-04.png)

稍微等待一下。

### 本地服务器开启

作为测试，我在本地也安装了个 nginx，然后在nginx的欢迎界面上稍微修改，以进行验证。

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-19-59-26.png)

## 结果

打开浏览器，访问 frp.ailemong.com。 

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-00-07.png)

## 阶段总结

至此，我们可以通过 frp.ailemong.com 访问本地的 80 端口了。

此时还不支持 https 访问，因为需要ssl证书。（二）中我们使用 acme 进行 https的泛解析证书的安装，并在nginx+frp中进行配置。

