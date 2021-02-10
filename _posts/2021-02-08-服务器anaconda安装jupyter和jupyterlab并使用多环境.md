---
title: 服务器anaconda安装jupyter和jupyterlab并使用多环境
categories:
  - ubuntu
tags:
  - anaconda
  - ubuntu
  - jupyter
  - jupyterlab
  - 虚拟环境
---

# 服务器anaconda安装jupyter和jupyterlab并使用多环境

以下安装均基于`Anaconda`,环境为`Ubuntu`, 默认环境里已经有基本的`jupyter`
将要安装或启用的功能有：

- [x] 远程访问

- [ ] Tensorboard支持

- [x] 多虚拟环境(Kernel)支持

- [x] 后台运行

- [x] jupyter lab

### 一，安装 *jupyterlab*

anaconda 默认是安装了 *jupyter notebook* 的，所以我们这里只需要再安装一个 *jupyterlab*

```shell
# 安装 jupyterlab
codna install jupyterlab
# 生成配置文件 jupyter_notebook_config.py
jupyter notebook --generate-config

```

> `jupyter notebook --generate-config`  这个命令会在生成一个文件： `~/.jupyter/jupyter_notebook_config.py` ,我们也可以手动创建这个文件

### 二，修改配置文件

##### 1) 创建一个访问密码

```shell
jupyter notebook password
Enter password: ****
Verify password: ****
```

##### 2) 获取访问密码

配置文件中不接受明文的密码，需要获取加密后的密码，然后填入配置文件中，在 bash 中输入 ipython 打开 python 终端。

```python
In [1]: from notebook.auth import passwd
In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'
```

##### 3) 修改配置文件

```shell
# 找到相应的行，然后修改
c.NotebookApp.allow_root = True # 允许以root方式运行jupyterlab
c.NotebookApp.ip = '0.0.0.0' # 允许任意ip段访问
c.NotebookApp.allow_remote_access = True # 允许远程访问
c.NotebookApp.notebook_dir = u'/root/JupyterLab' # 设置jupyterlab页面的根目录
c.NotebookApp.open_browser = False # 默认运行时不启动浏览器，因为服务器默认只有终端嘛
c.NotebookApp.password = u'sha1:b92f3fb7d848:a5d40ab2e26aa3b296ae1faa17aa34d3df351704' # 设置之前生产的哈希密码
c.NotebookApp.port = 8888  # 设置访问端口
```

⚠️如果服务器端有两个jupyterlab，则需要两个不同的配置文件，那么在`.jupyter`再创建一个不同的配置文件即可，例如`jupyter_notebook_config_2.py`，只不过在启动jupyterlab时候，需要如下命令：

```shell
jupyter notebook --config jupyter_notebook_config_2.py
```

`jupyter_notebook_config.py`是默认的配置文件，所以其对应的jupyter在启动时候，直接使用如下命令即可

```shell
jupyter notebook
```

##### 4） 启动服务

启动 jupyter notebook

`jupyter notebook # nohup jupyter notebook &` 

启动 jupyterlab

`jupyter lab # nohup jupyter lab &`

##### 5） 访问

可以通过 http://localhost:8888/lab 访问以下，至此，默认环境的 notebook 和 lab 就安装好了。

![image-20210210185615240](/public/img/image-20210210185615240.png)

### 二，多虚拟环境（kernel）支持

##### 1）创建虚拟环境

`conda create --name resnet python=3.8.5`

基本命令：

```shell
# 查看电脑的虚拟环境
conda env list
# 激活环境
conda activate resnet
# 查看当前环境安装的包
conda list
pip list
# 退出当前环境
conda deactivate
```

##### 2） 将环境加入到 notebook 和 lab

默认的jupyter是在哪个环境下运行就使用哪个环境，我需要多个环境就需要开启多个`jupyter`，很不方便。

###### 方法一，安装`ipykernel` 插件

```shell
# 激活进入相应的环境
conda activate resnet
# 安装插件
codna install ipykernel
# 安装完后，运行命令
python -m ipykernel install --name {你的虚拟环境名字，如resnet} --display-name {你想显示的名称，如resnet，可以一样}
# 查看
jupyter kernelspec list
# 删除不需要的内核
jupyter kernelspec remove <kernel_name>
# 其他内核安装：https://github.com/jupyter/jupyter/wiki/Jupyter-kernels
```

###### 方法二，使用 nb_conda 插件

作为偷懒一点的方法，只需要在每个环境中安装`nb_conda_kernels` 即可

```shell
conda install nb_conda_kernels
```

 安装完这个牛逼conda之后，再次启动jupyter notebook，就能看到**所有虚拟环境**都显示出来了。

##### 两个方法本质上是一样的

原理上来讲，都是在 **......../jupyter/kernels/**目录下，创建一个命名为 **{对应名称}** 的文件夹，文件夹下放一个**kernel.json ** 文件。

于是我们得到 **....../jupyter/kernels/{对应名称}/kernel.json**

运行 **python -m ipykernel ....**  那行命令，就是做**创建文件夹**和**创建文件**这两件事情。

在没运行python -m ipykernel ....之前**，**用find找一下这个文件

```shell
find . -name 'kernel.json'
```

可以看到输出为

```shell
./anaconda3/pkgs/ipykernel-5.1.0-py37h39e3cac_0/share/jupyter/kernels/python3/kernel.json
./anaconda3/share/jupyter/kernels/python3/kernel.json
```

这个 `/share/jupyter/kernels/python3/kernel.json `就是关键了。

我们来看看json文件的内容

```shell
{
 "argv": [
  "/home/qq/anaconda3/bin/python",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Python 3",
 "language": "python"
}
```

正是因为argv中设置的地址为 `./anaconda3/bin/python` ，才导致jupyter的默认kernel为anaconda的主环境。

> 显然，如果我们在对应的/kernels/目录下，新建一个文件夹，再新建一个kernel.json，
>
> 把地址设置为'./anaconda3/envs/qq/bin/python'，
>
> 就可以用到虚拟环境qq下的python了。

### 三，jupyter 扩展

```shell
# 查看已经安装的扩展及其状态
jupyter labextension list
# 安装一个扩展jupyterlab_html，支持html预览:
jupyter labextension install @mflevine/jupyterlab_html
# 卸载扩展:
jupyter labextension uninstall @mflevine/jupyterlab_html
# 更新所有扩展:
jupyter labextension update --all
```

