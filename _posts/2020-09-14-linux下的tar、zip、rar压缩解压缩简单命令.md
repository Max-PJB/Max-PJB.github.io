---
title: linux下的tar、zip、rar压缩解压缩基本命令
categories:
  - linux
tags:
  - tar
  - zip
  - rar
---

# linux下的tar、zip、rar压缩解压缩

## 1. tar 命令
语法： tar [主选项 + 辅选项] 文件或目录

### 示例
```shell
# 压缩文件 file1 和目录 dir2 到 test.tar.gz
tar -zcvf test.tar.gz file1 dir2
# 解压 test.tar.gz（将 c 换成 x 即可）
tar -zxvf test.tar.gz
# 列出压缩文件的内容
tar -ztvf test.tar.gz
```
### 释义：

>-z : 使用 gzip 来压缩和解压文件
>
>-v : --verbose 详细的列出处理的文件
>
>-f : --file=ARCHIVE 使用档案文件或设备，这个选项通常是必选的
>
>-c : --create 创建一个新的归档（压缩包）
>
>-x : 从压缩包中解出文件


## 2. rar 命令
### 示例
```
# 压缩文件
rar a -r test.rar file
# 解压文件
unrar x test.ra
```
### 释义：
>a : 添加到压缩文件
>
>-r : 递归处理
>
>x : 以绝对路径解压文件

## 3. zip 命令
### 示例：
```
# 压缩文件
zip -r test.zip file
# 解压文件
zip test.zip
```

