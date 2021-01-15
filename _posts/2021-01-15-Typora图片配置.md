---
title: Typora图片配置
categories:
  - ubuntu
tags:
  - ubuntu20
  - linux
  - typora
---

# Typora图片配置

我使用的是 github pages ，文章中引用的图片都放在 `/public/img/` 下。大概目录如图：

![image-20210115140439426](/public/img/image-20210115140439426.png)

那么，我需要在粘贴后生成的图片链接为 `/public/img/xxx.png` 的话，我需要怎么配置呢？

![image-20210115140607705](/public/img/image-20210115140607705.png)

步骤一，创建一个软链接，`/public/img` 链接到 真实的存放图片的路径

```shell
# 在`/`目录创建一个 `/public`目录
sudo mkdir /public
# 创建软链接
sudo ln -s /path/to/your/GitHubPages/public/img /public/img
```

步骤二，配置typora

[文件] -> [偏好设置] -> [图像] -> 插入图片时选择复制到制定路径，然后选择路径的时候选择 `/public/img` 

![image-20210115141727241](/public/img/image-20210115141727241.png)

