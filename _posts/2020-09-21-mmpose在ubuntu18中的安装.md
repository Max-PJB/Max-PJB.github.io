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

## 0. git mmpose下来

先啥也别说，把这个项目从 [github](https://github.com/open-mmlab/mmpose) 上搞下来啊。

`git clone https://github.com/open-mmlab/mmpose.git` 

## 1.安装 CUDA

这个没啥好说的，就按照官网的一路操作下去了。 [CUDA下载地址](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal) ，选择对应的版本，可以手动下载，也可以根据下面的 bash 命令安装

![image-20200921103502579](/public/img/image-20200921103502579.png)

![image-20200921103557504](/public/img/image-20200921103557504.png)

如果想配置多 cuda 版本切换，网上也有教程。大致思路的就是配置一个 环境变量指向一个 软链接 ， 这个软链接指向哪个 cuda 版本的目录就表示当前环境使用的是哪个 版本。

## 2.安装 Anaconda

这里参考我另一篇文章，介绍使用 清华源 下载安装 Anaconda 然后配置他的源。

## 3.创建环境，安装依赖

接下来基本就是按照  [mmpose官网来进行操作了](https://jackypjb.github.io/posts/ubuntu/ubuntu18%E5%AE%89%E8%A3%85Anaconda%E9%85%8D%E7%BD%AE%E6%B8%85%E5%8D%8E%E6%BA%90) ，不过官网内容较多。我这里就具体记录一下我是的操作。

![image-20200921111912209](/public/img/image-20200921111912209.png)

上面是官方需要的依赖。我们一个一个安装。

### 3.1 创建环境并进入环境

~~~python
conda create -n open-mmlab python=3.7 -y
conda activate open-mmlab
~~~

### 3.2 安装依赖

3.2.1. 安装 opencv  `pip install opencv-python`  我的版本是 opencv-python 4.4

这里会自动安装上 numpy

3.2.2. 安装 pytorch `conda install pytorch torchvision` 我的版本是 pytorch 1.6

3.2.3 安装 [mmcv](https://github.com/open-mmlab/mmcv) ，我们安装 全功能版本  `pip install mmcv-full`

3.2.3 [xtcocotools](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal)  `pip install xtcocotools`

3.2.4 json_tricks  `pip install json_tricks`

### 3.3 安装 mmdet

mmdet 是 [mmdetection](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal) 工具，安装他的需要使用本地手动安装。

它需要 mmcv pytorch cuda 等环境，我们前面已经安装了。

~~~
git clone https://github.com/open-mmlab/mmdetection.git
cd mmdetection
pip install -r requirements/build.txt
pip install -v -e .  # or "python setup.py develop"
~~~

## 4 下载测试数据集

官方支持很多数据集，我这里只使用了 COCO 数据集。按照它的指引取操作即可，唯一需要担心的墙的问题，有些数据是在 Google Driver 上的。

[数据集下载使用](https://mmpose.readthedocs.io/en/latest/data_preparation.html) [还有这里也是使用说明](https://github.com/open-mmlab/mmpose/blob/master/docs/data_preparation.md) 

![image-20200921113451809](/public/img/image-20200921113451809.png)

4.1 下载 COCO 数据集下载下图中的部分， Test images 可以不用下

![image-20200921122450844](/public/img/image-20200921122450844.png)

4.2 除了数据，还需要下载 HRNet 预训练的 目标检测结果，这个是放在 [OneDrive](https://1drv.ms/f/s%21AhIXJn_J-blWzzDXoz5BeFl8sWM-) 上的。

4.3 下载好了后，把他们一一解压，放到他们该在的目录下：

![image-20200921123347749](/public/img/image-20200921123347749.png)

## 5 下载与训练模型

5.1 去 mmpose的 [Pretrained backbones on ImageNet](https://mmpose.readthedocs.io/en/latest/pretrained.html) ，把他的imageNet预训练模型 backbone 统统下载下来，点 右边的ckpt 就能下载 。

模型放置成如下目录格式（models 文件夹没有就自己创建）：

![image-20200921124805877](/public/img/image-20200921124805877.png)

5.2 然后下载 [Bottom UP Models](https://mmpose.readthedocs.io/en/latest/bottom_up_models.html) ，下载模型，TOP DOWN 模型也可以下，都一样哟~

这个需要往右边拖，才能看到 ckpt 下载按钮：

![image-20200921124020801](/public/img/image-20200921124020801.png)

这些放在 checkpoints 目录下（没有就创建）

## 6 模型测试

进入 mmpose 项目，第一次使用还需要安装一些依赖 

~~~bash
cd mmpose # 注意，这里是已经激活了刚才创建的环境 mmlab 的状态哟
pip install -r requirements.txt # 第一次使用安装
python setup.py develop # 第一次使用安装
~~~

然后是测试，官网也提供了各种测试的命令解释，可以去查看 [官方教程](https://mmpose.readthedocs.io/en/latest/bottom_up_models.html)

我这里就测试一下 ==mobilenetv2_coco_512x512== 模型

执行命令就好

`python tools/test.py configs/bottom_up/mobilenet/coco/mobilenetv2_coco_512x512.py checkpoints/mobilenetv2_coco_512x512-4d96e309_20200816.pth  --eval mAP`

我这里执行真鸡儿慢，1.5 t/s ，查看 GPU 只有 3% 使用，应该要去某段代码增加并发。 

执行结果：![image-20200921125900836](/public/img/image-20200921125900836.png)

## 7 模型训练

### 单机器，单GPU

`python tools/train.py configs/bottom_up/mobilenet/coco/mobilenetv2_coco_512x512.py `

## 8 demo 使用



## 9 参考

CSDN： [https://blog.csdn.net/weixin_43013761/article/details/108148047?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.channel_param](https://blog.csdn.net/weixin_43013761/article/details/108148047?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-7.channel_param)

