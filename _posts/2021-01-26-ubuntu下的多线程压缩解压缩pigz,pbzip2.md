---
title: ubuntu下的多线程压缩解压缩
categories:
  - ubuntu
tags:
  - pigz
  - pbzip2
  - tar
---

**pigz(parallel implementation of gzip)是一个并行执行的压缩工具，解压缩比gzip快，同时CPU消耗是gzip的好几倍，在对短时间内CPU消耗较高不受影响的场景下，可以使用pigz。**

[http://blog.chinaunix.net/uid-31524109-id-5842017.html](http://blog.chinaunix.net/uid-31524109-id-5842017.html)

**如何压缩文件**

下面几个是常用参数：

- -p n: 压缩时使用的核心数量，默认使用所有核心
- -k: 压缩后保留源文件
- -l: 列出压缩输入的内容。
- -6: 默认的压缩级别
- -9: 压缩率最高，但是速度慢
- -1: 压缩率最低，速度最快

**如何压缩目录**

Pigz没有压缩文件夹的选项，只可以压缩单个文件。pigz可以和tar[命令](https://www.linuxcool.com/)一起使用，来压缩文件夹。

```shell
 tar -cvf - /var/log | pigz -k > logs.tar.gz
```

可以使用-l选项查看压缩后文件的压缩率

**参考实例**

可以结合tar使用, 压缩命令:

```shell
[root@linuxcool ~]# tar -cvf - dir1 dir2 dir3 | pigz -p 8 > output.tgz
```

**如何解压文件**



# [tar 集成pigz和pbzip2](https://www.cnblogs.com/sj9524437/p/4952911.html)

### pigz 压缩

1 tar直接调用pigz

tar -c -I pigz -f pigz.tar.gz file  

缺点:无法设置pigz的参数

2 使用管道

`tar -c -f - 05beea2f-3d45-4cb2-ba8f-ecb707a4b5ed | pigz -p4> test.tar.gz`

### pbzip2 压缩

3 tar直接调用pbzip2

`tar -c -I pbzip2 -f pbzip2.tar.bz2 file  ``

缺点:无法设置pigz的参数

4 使用管道

`tar -c -f - 05beea2f-3d45-4cb2-ba8f-ecb707a4b5ed | pbzip2 -p4 -c> test.tar.bz2`

### pigz 解压

使用pigz解压tar.gz包

`pigz -dc gzip.tar.gz | tar -x ``

### pbzip2 解压

使用pbzip2解压tar.bz2包

`pbzip2 -dc pbzip2.tar.bz2 | tar -x`

解压的时候加参数 -p# 可以设置解压的线程数



安装 pigz

```
sudo apt-get update
sudo apt-get install pigz
```

压缩：pigz -9: 指定压缩级别（9最高,1最低)，-p: 线程数

```
tar -cf - /home/file | pigz -9 -p 12 > file.tgz
```

解压缩：-p: 要分配的线程数，-d: 解压的文件的路径

```
pigz -p 12 -d file.tgz
```

此时，解压完是.tar文件，还需要进一步解压缩：

```
tar –xvf file.tar
```

