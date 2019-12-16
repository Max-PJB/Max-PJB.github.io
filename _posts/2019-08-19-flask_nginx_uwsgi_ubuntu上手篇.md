---
title: flask 部署 nginx uwsgi ubuntu supervisor，-上手篇
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

## ubuntu 下安装 python 虚拟环境

ubuntu默认是安装了 python2 和 python3 的。但是没有Python3.x环境下的pip3安装工具，所以我们需要手动安装pip3。

```shell
sudo apt-get install python3-pip # 安装pip3
pip3 install --upgrade pip # 升级 pip3
```

升级后报错，pip无法使用

```shell
ubuntu@VM-0-9-ubuntu:~$ pip3 -V
Traceback (most recent call last):
  File "/usr/bin/pip3", line 9, in <module>
    from pip import main
ImportError: cannot import name 'main'
```

找到一个答案，将当前使用的pip或pip3文件按其所写如下修改，位置是 /usr/bin/pip 或 /usr/bin/pip3，此操作需要管理员权限。

`sudo vim /usr/bin/pip3`

前：
```python
from pip import main
if __name__ == '__main__':
    sys.exit(main())
```
后：
```python
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```

我们这里只使用 virtualenv，后面环境多了，可以考虑加入 virtualenvwrapper

virtualenv是虚拟环境，virtualenvwrapper对virtualenv的命令进行了封装，使得其更加友好，对多个环境集中管理。

```shell
sudo pip3 install virtualenv  # 安装virtualenv
#sudo pip3 install virtualenvwrapper
```

> 注意：安装的顺序不能颠倒，virtualenvwrapper必须依赖于virtualenv。

创建、启用、退出、删除 虚拟环境

```shell
# 指定python版本，否则就是使用默认的系统 python版本
virtualenv --no-site-packages --python=3.6.8 my_python # 会在当前创建一个 my_python 目录
# --no-site-packages 表示不继承系统python的包，若想包括系统的site-packages，可以使用 --system-site-packages
source my_python/bin/activate # 进入虚拟环境
python>> deactivate # 退出当前虚拟环境
rm -r my_python # 删除环境直接把目录删了就好了，Linux 万物皆文件嘛
```

TODO
> 关于 virtualenvwrapper 的东西，后面用到了我再加

## 安装 uwsgi

安装依赖包
`sudo apt-get install build-essential python-dev`

进入虚拟环境 `source env/bin/activate`
```shell
pip3 install uwsgi
```

安装uwsgitop，监控数据
`pip install uwsgitop`

配置 uwsgi

以一个简单的 flask 为例 ，/home/ubuntu/test/myflaskapp.py

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "<span style='color:red'>I am app 1</span>"
```

创建一个 uwsgi 的配置文件 /home/ubuntu/test/my_uwsgi.ini

```shell
[uwsgi]
socket = 127.0.0.1:5051
pythonpath = /home/ubuntu/test # 应用目录，即项目代码所在目录
module = myflaskapp # web 应用python主程序名
wsgi-file = /home/ubuntu/test/myflaskapp.py # web 应用python主程序
callable = app # 一般在主运行程序 run_app.py 里指定 app = Flask(__name__)
processes = 1
# threads = 2
# 使用supervisor就不需要daemonize了，否则不能拉起uwsgi程序
# daemonize = /home/ubuntu/test/server.log
# python 虚拟环境目录
home = /home/ubuntu/lemong-server
# 该参数是设置打开http body缓冲, 如果HTTP body的大小超过指定的限制,那么就保存到磁盘上，单位是 Byte，8192就是 8K
post-buffering = 8192
```

```shell
# 启动 uwsgi
uwsgi /home/ubuntu/test/my_uwsgi.ini

# 停止 uwsgi
pkill -f -9 uwsgi

uwsgi --reload uwsgi.pid # 重启
uwsgi --stop uwsgi.pid # 关闭

```


## 安装 nginx
`sudo apt install nginx`

`sudo nginx -V` 查看版本

```
sudo service nginx stop
sudo service nginx start
sudo service nginx restart
sudo nginx -s reload
```

配置nginx

安装完成后默认的nginx的配置文件位于`/etc/nginx/sites-enabled/default`
其实nginx默认读取的文件是`/etc/nginx/nginx.conf`，打开这个文件看看可以看到在其http块中有些

```
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

所以我就把 `sudo mv /etc/nginx/sites-enabled/default /etc/nginx/conf.d/` 移动过来，作为模板，因为没有 conf 后缀，所以这个模板是不起作用的

然后复制一份自己的 `sudo cp /etc/nginx/conf.d/default /etc/nginx/conf.d/myflaskapp.conf`

然后再 myapp.conf 中写入

```shell
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;

        location / {
            include        uwsgi_params;
            uwsgi_pass     127.0.0.1:5051;
        }
```

这样就把 80 端口的 http 请求转发到 uwsgi 的 5051 端口了

## 安装 supervisor
`sudo apt-get install supervisor`

安装成功后，会在/etc/supervisor目录下，生成supervisord.conf配置文件。

从文件中可以看到  进程配置会读取 /etc/supervisor/conf.d目录下的*.conf配置文件
```
[include]
files = /etc/supervisor/conf.d/*.conf
```
那我们就在conf.d目录下创建一个myflaskapp.conf进程配置文件：
```
[program:myflaskapp]
# 注意这里，user，我之前习惯性的写 root ，结果腾讯云 ubuntu 的默认用户是 ubuntu，把我坑了 3 个小时，操蛋
user=ubuntu
command = /home/ubuntu/lemong-server/bin/uwsgi --ini /home/ubuntu/test/uwsgi.ini


autostart=true
autorestart=true
startretries=3

redirect_stderr=true
stdout_logfile=/var/log/uwsgi/supervisor_myflaskapp.log
stderr_logfile=/var/log/uwsgi/supervisor_myflaskapp_error.log
```
在相应的目录下创建/var/log/uwsgi/supervisor_myflaskapp.log 和 /var/log/uwsgi/supervisor_myflaskapp_error.log,否则会报错

```shell
sudo supervisord -c supervisord.conf  # 加载配置文件，启动 supervisor、
sudo supervisorctl reload # 重新启动配置中的所有程序
sudo supervisorctl status # 查看现在运行的supervisor进程的状态
sudo supervisorctl shutdown # 会关闭supervisor进程和其管理的子进程
```

启动后

```
(lemong-server) ubuntu@VM-0-9-ubuntu:~/test$ ps -aux|grep uwsgi
ubuntu   19201  4.0  2.7  95296 27592 ?        S    15:29   0:00 /home/ubuntu/lemong-server/bin/uwsgi --ini /home/ubuntu/test/uwsgi.ini
ubuntu   19210  0.0  0.1  13772  1036 pts/0    S+   15:29   0:00 grep --color=auto uwsgi
(lemong-server) ubuntu@VM-0-9-ubuntu:~/test$ pkill -f -9 uwsgi
(lemong-server) ubuntu@VM-0-9-ubuntu:~/test$ ps -aux|grep uwsgi
ubuntu   19214  8.0  2.7  95296 27648 ?        S    15:29   0:00 /home/ubuntu/lemong-server/bin/uwsgi --ini /home/ubuntu/test/uwsgi.ini
ubuntu   19223  0.0  0.1  13772  1016 pts/0    S+   15:29   0:00 grep --color=auto uwsgi
```

杀了之后，又自动重启了，变了 pid 看到没，说明成功了。


一些常见的错误
```shell
 1.Error: Another program is already listening on a port that one of our HTTP servers is configured to use. Shut this program down first before starting

# 解决方法：
sudo find / -name supervisor.sock
输出 /xxx/supervisor.sock
sudo unlink /xxx/supervisor.sock # xxx 对应上一个命令的输出

2. gave up: transfer entered FATAL state, too many start retries too quickly

去掉supervisord.conf的directory=/home/ruifenygun/shop_master/shop_scripts配置行就可以了.
原因已经问老外了.

3. FATAL     Exited too quickly (process log may have details)
    这个一般都是配置写错了，我是因为 user=root 写错了，我腾讯云的 root 用户名是 ubuntu！！坑爹的腾讯云 fuck
```
