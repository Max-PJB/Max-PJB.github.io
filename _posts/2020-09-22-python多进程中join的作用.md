---
title: python多进程中join的作用
categories:
  - python
tags:
  - python
  - multiprocessing
  - 多进程
  - join
  - start
---

[TOC]

## python多进程中join的作用

学python多进程的时候经常看到 `p=Process(target=func……);p.start();p.join()` 

`start()`  很好理解，就是启动进程嘛，那这个 `join()` 是干嘛的呢？

比如  `p1.join()` 是当前 p1 进程阻塞主进程：主进程需要等待 p1 进程执行结束才继续执行后面的代码：一般用来后面的代码需要收集 p1 进程的执行结果的情况。

### 举例

~~~python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
-------------------------------------------------

    @   Author  :       pengjb
    @   date    :       2020/9/22 下午2:21
    @   IDE     :       PyCharm
    @   GitHub  :       https://github.com/JackyPJB
    @   Contact :       pengjianbiao@hotmail.com
-------------------------------------------------
    Description :       process start join
-------------------------------------------------
"""
import multiprocessing


def count_pp(k, **kwargs):
    r = 0
    for _ in range(k):
        r += 1
    print(multiprocessing.current_process().name,kwargs, r)


if __name__ == '__main__':
    p1 = multiprocessing.Process(target=count_pp, name='1111', args=(10**5,), kwargs={'a': 1})
    p2 = multiprocessing.Process(target=count_pp, name='222', args=(10,), kwargs={'b': 2})
    p3 = multiprocessing.Process(target=count_pp, name='33', args=(10 ** 8,), kwargs={'c': 3})
    p1.start()
    p2.start()
    p3.start()
    p1.join()
    p2.join()
    print('end')
"""
result: p1，p2 使用 join 所以 print('end')需要等待p1 p2 执行结束再执行
222 {'b': 2} 10
1111 {'a': 1} 100000
end
33 {'c': 3} 100000000
"""
~~~

