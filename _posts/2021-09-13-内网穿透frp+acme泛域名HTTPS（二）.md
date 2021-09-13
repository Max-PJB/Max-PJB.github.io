---
*title*: 内网穿透frp+acme泛域名HTTPS（二）
categories:
  - frp
tags:
  - frp
  - 内网穿透
  - 泛域名
  - HTTPS
---

# 内网穿透frp+acme泛域名HTTPS（二）

> 需求：外网访问，https访问（微信小程序）

碍于各种云服务器厂商的高配置版本都太贵了，想使用一个微服务或者用一个GPU做人工智能处理作为服务器就智能在本地搭建服务平台，然后让本地的服务通过内网穿透来给公网用户使用，主要通过微信小程序https访问。

## （二）https 泛解析

泛域名解析就是我想通过 xx.frp.ailemong.com 访问到我的本地服务器，并且是使用 https 的方式，即 https.xx.frp.ailemong.com。

我采用 acme 的方案进行泛域名解析证书申请。

### 准备工作

1. 一个域名（已备案，以下使用腾讯云，ailemong.com）
2. 一台云服务器（简配的云服务器还是不贵的，使用腾讯云，云服务器一般都会有一个公网IP，假设我这里是12.34.56.78）
3. 一台可以上网的电脑（废话）
4. 下载 [frp](https://github.com/fatedier/frp/releases) 。（[frp](https://github.com/fatedier/frp/releases) 分为客户端和服务器，都在一个压缩包里，不同的系统使用对应的压缩包里的部分)

### 申请 DNSPOD的token

后面需要使用到。

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-07-45.png)

记录下 :

```bash
ID: 258315
Token: 9xxxxxxxxxxxxx945
```

### DNS解析

既然要使用到泛域名，那DNS解析自然是要配置的。

同（一）中添加一条 A 记录。 `*.frp `

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-17-59.png)

### 下载安装 acme

##### 下载

在 `~` 目录下，安装很简单, 一个命令:

```bash
curl  https://get.acme.sh | sh -s email=my@example.com
```

普通用户和 root 用户都可以安装使用.。

安装成功会 新建一个文件夹 `~/.acme.sh/`。

##### 申请证书

1.把前面申请的 秘钥 export 为环境变量：

```bash
export DP_Id="258315"
export DP_Key="9xxxxxxxxxxxxx945"
```

注意：这里仅限腾讯云，其他云服务配置不同，可[参考这里](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)。

2.输入命令，申请证书

```bash
 acme.sh --issue -d frp.ailemong.com -d *.frp.ailemong.com --dns dns_dp --debug 2
 
acme.sh ：表示使用你刚安装好的acme.sh

-–issue ：申请证书

–dns：使用 DNS 的方式来验证你的所有权，你需要在域名上添加一条 txt 解析记录, 验证域名所有权。

-d [域名]：-d表示 domain，后面跟你要申请域名。

--debug 2： 输出详细日志，建议带上，出错方便排查
```

有时候刚安装， `acme.sh` 命令无法使用，可以 `source ~/.bashrc`一下。

成功界面：

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-30-17.png)

如果成功，会出现**出现类似如下信息：** Adding txt value， **（tips:如果日志太多，建议复制全部到本地搜索）**

```shell
 [Sun May  9 23:12:49 CST 2021] Found domain api file: /root/.acme.sh/dnsapi/dns_dp.sh
 [Sun May  9 23:12:49 CST 2021] Adding txt value: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx     for domain:  _acme-challenge.frp.ailemong.com
 [Sun May  9 23:12:49 CST 2021] First detect the root zone
```

3.**Adding txt value:** 在云控制台添加 TXT 解析，记录值为上面日志中的 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 。类型是 **TXT** ！

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-28-05.png)

4.copy 安装证书

[https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

前面证书生成以后, 接下来需要把证书 copy 到真正需要用它的地方.

注意, 默认生成的证书都放在安装目录下: `~/.acme.sh/`, 请不要直接使用此目录下的文件, 例如: 不要直接让 nginx/apache 的配置文件使用这下面的文件. 这里面的文件都是内部使用, 而且目录结构可能会变化.

正确的使用方法是使用 `--install-cert` 命令,并指定目标位置, 然后证书文件会被copy到相应的位置, 例如:

Nginx example:

```bash
acme.sh --install-cert -d *frp.ailemong.com -d frp.ailemong.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"

```

(一个小提醒, 这里用的是 `service nginx force-reload`, 不是 `service nginx reload`, 据测试, `reload` 并不会重新加载证书, 所以用的 `force-reload`)

Nginx 的配置 `ssl_certificate` 使用`/etc/nginx/ssl/fullchain.cer` ，而非 `/etc/nginx/ssl/<domain>.cer`。

### 配置 frp

##### 配置服务端

服务端配置文件不需要修改 `frps.ini`：

```bash
[common]
bind_port = 7000  #监听frp客户端的端口
vhost_http_port = 8080 #vhost端口，将这个端口来的请求转发给客户端
subdomain_host = frp.ailemong.com # 设置 host，在客户端配置中就可以试用 subdomain 来配置子域名，避免写太多custom_domains

```

启动服务端，命令：`./frps -c frps.ini`

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-19-28-28.png)

##### 配置客户端

修改客户端配置文件，添加两个二级域名，**api** 和 **haha** ，`frpc.ini`:

```bash
[common]
server_addr = 12.34.56.78 # 这里需要填腾讯云服务器的公网IP
server_port = 7000 # 这里需要和服务端的监听端口保持一致

[web]
type = http
local_port = 80 # 本地服务提供的端口
custom_domains = frp.ailemong.com # 注意： 只有通过这个url来的请求会被处理，使用 ip 地址是没有用的！ 

[api]
type = http
local_port = 80
subdomain = api

[haha]
type = http
local_port = 80
subdomain = haha
```

启动客户端，命令：`./frpc -c frpc.ini`

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-19-38-47.png)

更多 frp 的文档可以参考 [FRP](https://gofrp.org/docs/concepts/)。

### 在云服务器上配置 nginx

##### 修改配置文件

修改在（一） 中新建的配置文件 `/etc/nginx/conf.d/frp_ssl.conf`：

```bash
server {
 listen 80;
 listen 443 ssl http2;
 ssl_certificate /path/to/keyfile/in/nginx/key.pem;#根据安装证书步骤中的路径相应修改
 ssl_certificate_key /path/to/fullchain/nginx/cert.pem;#根据安装证书步骤中的路径相应修改
 server_name frp.ailemong.com;
 proxy_set_header Host $host;
 proxy_set_header X-Forwarder-For $remote_addr;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection "Upgrade";

 location / {
      proxy_pass http://127.0.0.1:8080; #端口号一定要和frps.ini的vhost_http_port一致
    }
}


server {
 listen 80;
 listen 443 ssl http2;
 ssl_certificate /path/to/keyfile/in/nginx/key.pem; #根据安装证书步骤中的路径相应修改
 ssl_certificate_key /path/to/fullchain/nginx/cert.pem;#根据安装证书步骤中的路径相应修改
 server_name *.frp.ailemong.com;
 proxy_set_header Host $host;
 proxy_set_header X-Forwarder-For $remote_addr;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection "Upgrade";

 location / {
      proxy_pass http://127.0.0.1:8080; #端口号一定要和frps.ini的vhost_http_port一致
    }
}

```

然后重启 nginx，命令：`sudo service nginx restart` 或者重新加载配置也行，命令： `sudo nginx -s reload`，命令成功没有任何输出，有输出就是报错了。

稍微等待一下。

## 结果

打开浏览器，访问 https://api.frp.ailemong.com 、https://haha.frp.ailemong.com 和 https://frp.ailemong.com。 

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-42-32.png)

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-43-19.png)

![](https://cdn.jsdelivr.net/gh/max-pjb/imgs/image-2021-09-13-20-43-50.png)

## 总结

通过使用 frp 进行内网穿透，将本地的服务在服务器上以 vhost 的方式转发。

在 nginx 中需要进行相应的配置，其中包括转发和消息头的修改。

通过 acme.sh 进行泛域名的自动 ssl 证书申请和安装。

需要根据不同的云服务器厂商，进行 DNS 解析，其中包括 acme 中关于token的配置文件，不同厂商 环境变量不同。

