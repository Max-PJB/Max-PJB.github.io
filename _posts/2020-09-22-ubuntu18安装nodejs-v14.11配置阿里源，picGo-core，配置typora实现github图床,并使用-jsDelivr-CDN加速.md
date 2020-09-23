---
title: 2020-09-22-ubuntu18安装nodejs v14.11配置阿里源，picGo-core，配置typora实现github图床,并使用 jsDelivr CDN加速
categories:
  - ubuntu
tags:
  - ubuntu
  - nodejs
  - picgo
  - github
  - 图床
  - typora
  - jsDelivr
  - CDN
---

[TOC]

# ubuntu18安装nodejs v14.11配置阿里源，picGo-core，配置typora实现github图床,并使用 jsDelivr CDN加速

## 安装nodejs

去 [nodejs中文网下载相应的版本](http://nodejs.cn/download/) 。选择 linux 二进制文件。

下载下来是类似一个这样的格式：`node-v14.11.0-linux-x64.tar.xz`  tar.xz的压缩文件。

使用命令 `tar -Jvxf node-v14.11.0-linux-x64.tar.xz` 解压

把得到的目录移到一个合适的位置，以免放在 Downloads 里面搞混了。一般都是放在 `/usr/local/lib/` 下。然后在环境变量中把 node/bin 加进去

~~~bash
# 挪个位置
sudo mv node-v14.11.0-linux-x64 /usr/local/lib/nodejs
# 环境变量
vim ~/.bashrc
# 在文件的末尾添加一行 export PATH=/usr/local/lib/nodejs/bin:$PATH
# 保存，退出，运行下面命令使之生效
source ~/.bashrc 
# 查看一下
node -v
# v14.11.0
npm -v
# 6.14.8
~~~

## 配置阿里源

很简单哟，就一行命令

`npm config set registry https://registry.npm.taobao.org`

查看当前使用的源

`npm config get registry`

OK！

## github仓库配置

### 新建一个仓库

github怎么新建仓库就不用说了，我们这里创建一个仓库【imgs】专门用来存放图片，这样就相当于是我们私人的图床了。

需要注意的是，一定要勾选 ==public== ，把这个仓库设定为公有，不然怎么当图床用呢是吧？

### 获取token

在**主页**（**是主页哦，不是仓库的那个**）依次选择【Settings】-【Developer settings】-【Personal access tokens】-【Generate new token】，填写好描述，勾选【repo】，然后点击【Generate token】生成一个Token，注意这个Token只会显示一次，自己先保存下来，或者等后面配置好PicGo后再关闭此网页。

> ps：忘了也没关系，把原先的删了重新生成一个就好了。

## 安装配置 picgo-core

### 安装

命令行  `npm install picgo -g`  搞定

然后输入 `which picgo` 验证一下。

### 配置

picgo 可以自动配置，也可以手动配置

#### 自动配置

在bash中输入 `picgo set uploader` 可以进入交互式命令行配置，[参考官网](https://picgo.github.io/PicGo-Core-Doc/zh/guide/config.html#%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90) 

~~~bash
$ picgo set uploader
? Choose a(n) uploader (Use arrow keys)
  smms
❯ tcyun
  github
  qiniu
  imgur
  aliyun
  upyun
? Choose a(n) uploader github
? repo: imgs # 之前设置的 【imgs】
? branch: master # 默认回车
? token: xxxxxxxxxxxxxxxxxxxx # 上面 github 申请到的 token
? path: # 图片存放的路径，如果想把图片放在 仓库 imgs/xxx/ 下，可以path设置为 xxx/
? customUrl: https://cdn.jsdelivr.net/gh/JackyPJB/imgs # 就是这里用到了 CDN
[PicGo SUCCESS]: Configure config successfully!
~~~

- **设定仓库名：**按照【用户名 / 图床仓库名】的格式填写
- **设定分支名：**【master】
- **设定Token：**粘贴之前生成的【Token】
- **指定存储路径：**填写想要储存的路径，如【PCS/】，这样就会在仓库下创建一个名为 PCS 的文件夹，图片将会储存在此文件夹中
- **设定自定义域名：**它的作用是，在图片上传后，PicGo会按照【`自定义域名+储存路径+上传的图片名`】的方式生成访问链接，放到粘贴板上，因为我们要使用 jsDelivr 加速访问，所以可以设置为【`https://cdn.jsdelivr.net/gh/用户名/图床仓库名` 】，上传完毕后，我们就可以通过【`https://cdn.jsdelivr.net/gh/用户名/图床仓库名/图片路径`】加速访问我们的图片了，比如：https://cdn.jsdelivr.net/gh/jackypjb/imgs/PCS/01.png

设置成功后会生成一个 ~/.picgo 文件夹，里面 config.json 就是刚才我们一路的配置.

#### 手动配置

新建文件  `~/.picgo/config.json` 我这里放出我的配置，[具体参考官网](https://picgo.github.io/PicGo-Core-Doc/zh/guide/config.html#%E6%89%8B%E5%8A%A8%E7%94%9F%E6%88%90)

~~~bash
{
  "picBed": {
    "uploader": "github",
    "current": "github",
    "github": {
      "repo": "JackyPJB/imgs",
      "branch": "master",
      "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "path": "",
      "customUrl": "https://cdn.jsdelivr.net/gh/JackyPJB/imgs"
    }
  },
  "picgoPlugins": {}
}

~~~



## typora 配置

打开 typora 的偏好设置，可以使用 CTRL + comma（逗号）打开。

图像选项：上传服务选择 ==Custom Command== ，自定义命令填写

`/usr/local/lib/nodejs/bin/node /usr/local/lib/nodejs/bin/picgo u`  最后的 u 表示upload。注意路径要根据自己的情况相应修改

![image-20200923093941261](https://cdn.jsdelivr.net/gh/JackyPJB/imgs/image-20200923093941261.png)

这就ok了，每次把图片粘贴进来，都会有一个上传图片的选项，点击上传，成功后返回 url 自动填入。

大功告成！

## 更多思考

搞完整个流程发现：

1. picgo 的原理就是根据 github 的 rest api 来上传图片。那我们不就可以同样自己通过 python 的 requests 来写一个吗？ 
2. typora 的 Custom Command 是直接 bash 运行指令，那我们改成 python my.py 不就也可以了吗？
3. typora 怎么获取 my.py 上传后获取到的结果 URL ？

直接上代码：

~~~python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
-------------------------------------------------

    @   Author  :       pengjb
    @   date    :       2020/9/22 下午10:54
    @   IDE     :       PyCharm
    @   GitHub  :       https://github.com/JackyPJB
    @   Contact :       pengjianbiao@hotmail.com
-------------------------------------------------
    Description :       
-------------------------------------------------
"""
import requests
import base64
import json

from PyQt5 import QtCore
from PyQt5.QtWidgets import QApplication
from PIL import Image
import datetime


# 将文件转换为base64编码，上传文件必须将文件以base64格式上传
def read_file_base64(file_path):
    # 读取文件
    with open(file_path, 'rb') as f:
        data = f.read()
    data_b64 = base64.b64encode(data).decode('utf-8')
    return data_b64


# 上传文件
def upload_file(image_base64):
    dt_ms = datetime.datetime.now().strftime('%Y-%m-%d-%H-%M-%S-%f')  # 含微秒的日期时间
    image_name = 'image-' + dt_ms + '.png'  # 文件名
    token = "21f3914d5431488509232756ac2cac43eb15ece0"
    url = "https://api.github.com/repos/JackyPJB/imgs/contents/" + image_name  # 用户名、库名、路径
    headers = {"Authorization": "token " + token}
    data = {
        "message": "message",
        "committer": {
            "name": "pengjb",
            "email": "pengjianbiao@hotmail.com"
        },
        "content": image_base64
    }
    data = json.dumps(data)
    req = requests.put(url=url, data=data, headers=headers)
    req.encoding = "utf-8"
    re_data = json.loads(req.text)
    print(re_data)
    print(re_data['content']['sha'])
    print("https://cdn.jsdelivr.net/gh/JackyPJB/imgs/" + image_name)
	# 在国内默认的down_url可能会无法访问，因此使用CDN访问


if __name__ == '__main__':
    # upload_file('test.png')
    app = QApplication([])
    cb = app.clipboard()
    if cb.mimeData().hasImage():
        qt_img = cb.image()
        # print(type(qt_img))
        # Qimage to base64
        # QImage通过ByteArray转化为BASE64字符串
        qb = QtCore.QByteArray()
        buf = QtCore.QBuffer(qb)
        qt_img.save(buf, 'PNG')
        img_b64 = base64.b64encode(qb).decode('utf-8')
        # pil_img = Image.fromqimage(qt_img)  # 转换为PIL图像
        # pil_img.save('./haha.png', "PNG")
        upload_file(img_b64)
~~~