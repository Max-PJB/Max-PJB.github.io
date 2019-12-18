---
title: flask 部署 nginx uwsgi ubuntu supervisor，-上线篇
categories:
  - flask
  - python
  - nginx
  - uwsgi
  - supervisor
  - 部署
tags:
  - flask
  - python
  - nginx
  - uwsgi
  - supervisor
  - 部署
---

## ubuntu 下安装 python virtualenv 虚拟环境 （略，见上手篇）
安装virtualenv `sudo pip3 install virtualenv`

创建、启用、退出、删除 虚拟环境
```shell
# 指定python版本，否则就是使用默认的系统 python版本
virtualenv --no-site-packages --python=3.6.8 my_python # 会在当前创建一个 my_python 目录
# --no-site-packages 表示不继承系统python的包，若想包括系统的site-packages，可以使用 --system-site-packages
source my_python/bin/activate # 进入虚拟环境
python>> deactivate # 退出当前虚拟环境
rm -r my_python # 删除环境直接把目录删了就好了，Linux 万物皆文件嘛
```

## 安装 uwsgi
`source /path/to/env/bin activate`进入虚拟环境，`pip install uwsgi`

## 安装 nginx
`sudo apt install nginx`

## 安装 supervisor
`sudo apt-get install supervisor`

## 配置
### flaks 项目目录
> 我这里是以 create_app() 来写的 flask 应用，废话不多说，看图

`/app/init.py`
![](/public/img/2019-12-14-1.png)

`run.py`
![](/public/img/2019-12-14-2.png)

整个项目从github上拉取下来，放在目录 `/home/ubuntu/workspace/shop_pc_server`
### uwsgi 配置
在 `/home/ubuntu/workspace/shop_pc_server` 新建 `uwsgi.ini`配置文件

> 有蛮多坑的，比如不加 pythonpath，导致错误 `from app import create_app ModuleNotFoundError: No module named 'app'`

```shell
[uwsgi]
# 使用nginx连接时使用socket通信
# socket = 127.0.0.1:5055
# 直接使用自带web server 使用http通信 for test
http = 0.0.0.0:5055
# pythonpath 这一行是必须的，不然会报错 from app import create_app ModuleNotFoundError: No module named 'app'
# 原因就是找不到app模块，因为没有把这个模块加载python环境变量中嘛
pythonpath = /home/ubuntu/workspace/shop_pc_server/
# 项目根目录
chidir = /home/ubuntu/workspace/shop_pc_server
# 指定加载的WSGI文件
wsgi-file = /home/ubuntu/workspace/shop_pc_server/run.py
# 指定uWSGI加载的模块中哪个变量将被调用
callable = app
# 设置工作进程的数量
# 设置每个工作进程的线程数
processes = 1
# threads = 2

# daemonize = /home/ubuntu/test/server.log
# python 虚拟环境目录
home = /home/ubuntu/python-envs/shop_pc_server
# 该参数是设置打开http body缓冲, 如果HTTP body的大小超过指定的限制,那么就保存到磁盘上.
post-buffering = 8192
```

可以 `uwsgi --ini uwsgi.ini` 运行查看结果

### supervisor 配置
> supervisor 配置文件的地址默认是在 `/etc/supervisor/conf.d`

新建文件 `shop_pc_server.conf `

user 处是因为我的用户名是 ubuntu，万恶的腾讯云

```shell

[program:shop_pc_server]
user=ubuntu
command = /home/ubuntu/python-envs/shop_pc_server/bin/uwsgi --ini /home/ubuntu/workspace/shop_pc_server/uwsgi.ini

autostart=true
autorestart=true
startretries=3

redirect_stderr=true
stdout_logfile=/var/log/uwsgi/supervisor_shop_pc_server.log
stderr_logfile=/var/log/uwsgi/supervisor_shop_pc_server_error.log
```

在相应的目录下创建`/var/log/uwsgi/supervisor_shop_pc_server.log` 和 `/var/log/uwsgi/supervisor_shop_pc_server_error.log`,否则启动会报错

supervisor 部分命令
```shell
sudo supervisord -c shop_pc_server.conf  # 加载配置文件，启动 supervisor、
sudo supervisorctl reload # 重新启动配置中的所有程序
sudo supervisorctl status # 查看现在运行的supervisor进程的状态
sudo supervisorctl shutdown # 会关闭supervisor进程和其管理的子进程
```

### nginx 配置
> nginx 配置文件的地址默认是在 `/etc/nginx/conf.d`

随便新建个以 $\color{red}{.conf}$ 结尾的文件， `xxx.conf`

```shell
server {
     listen 80;
     # 这里配置了一个虚拟主机，当不同二级域名 xx.ailemong.com 访问的时候，就可以做不同的映射
     server_name api.ailemong.com;
     location / {
     #    include       uwsgi_params;
     #    uwsgi_pass    http://127.0.0.1:5055;
         proxy_pass      http://127.0.0.1:5055;
     }
}
```

> 注意了，uwsgi中如果用的是 socket，nginx中就要 include 和 uwsgi_pass,注释proxy_pass
>
> 原因就是 socket 和 http 使用的协议不一样， uwsgi_pass 和 proxy_pass 就是分别对应的 socket和http协议

nginx 部分命令
```shell
sudo service nginx stop
sudo service nginx start
sudo service nginx restart
sudo nginx -s reload
```
