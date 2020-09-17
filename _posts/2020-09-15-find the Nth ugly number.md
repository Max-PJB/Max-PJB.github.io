---
title: find the Nth ugly number
categories:
  - Leetcode
tags:
  - ugly number
  - Leetcode
  - leetcode

---

# find the Nth ugly number

因子中仅仅包含2、3、5的数，称为丑数。

比如 ==1，2，3，4，5，6== 都是丑数，因子只有 <span style="color:red">2，3</span>（其中 把 1 也看做一个丑数）

再比如 ==7== 就不是一个丑数，它有因子 <span style="color:red">7</span>

要求第 n 个丑数。

## 暴力法

思路：从 1 开始往后遍历，判断当前数字是不是丑数，一直找到第 n 个丑数为止。

丑数的判断：可以整除 2 就一直除，同样对 3，5 。最后得到的结果是 1 就是丑数，否则不是。

```python
# 判断一个数是不是丑数
def is_ugly(x):
    while x % 2 == 0:
        x //= 2
    while x % 3 == 0:
        x //= 3
    while x % 5 == 0:
        x //= 5
    if x == 1:
        return True
    else:
        return False


def n_ugly1(k):
    i = 1
    cnt = 1
    while cnt < k:
        i += 1
        while not is_ugly(i):
            i += 1
        cnt += 1
    return i
```

这么做有一个问题就是 当  $n=1500$ 的时候，答案是 854296875。这个遍历下去会花费很多时间。

## 快速法-空间换时间

思路：如果已经知道前 K 个丑数列表 CS，现在求第 K+1 个丑数。这个新的丑数一定是前面某个丑数（定位下标 m）C[m] 与 2/3/5 之间的一个的乘积。

我们可以用 2 * CS 中所有元素，3 * CS，5 * CS 然后把得到的值找一个未出现的最小值，那就是下一个新的丑数了。

```python
cs = [1]
# 所有数和 2 乘
# 过滤已经出现的丑数，不大于当前最大的丑数的就过滤掉,然后取最小值
mul2 = min(filter(lambda x, x > cs[-1] ,list(map(lambda x:x*2,cs))))
mul3 = min(filter(lambda x, x > cs[-1] ,list(map(lambda x:x*3,cs))))
mul5 = min(filter(lambda x, x > cs[-1] ,list(map(lambda x:x*5,cs))))
cs.append(min(mul2,mul3,mul5))
```

上面这个做法会有很多重复的乘法，我们思考如何避免这些多余的计算，直接求出下一个丑数?

> 最新的丑数一定是上一个丑数 乘 2/3/5 得来的
>
> 那我们可以分别用 3 个下标 ==p2,p3,p5== 来记录:
>
> > p2:  CS[p2] 位置的丑数乘2得到一个新的丑数，这个丑数是当前可以通过乘 2 获得的所有丑数中，且未在 CS 中出现过的最小值。
> >
> > 同理 p3，p5
>
> 记过计算 CS[p2] * 2， CS[p3] * 3， CS[p3] * 5，我们取他们的最小值为新的丑数。
>
> 最小值对应的下标 p 往后移动一个位置。p += 1

![image-20200917110352985](/public/img/image-20200917110352985.png)

[图片来源]:https://blog.csdn.net/qq756441663/article/details/102556760

代码：

```python
def n_ugly2(k):
    if k == 1:
        return 1
    uglies = [1]
    p2 = 0
    p3 = 0
    p5 = 0
    for _ in range(k - 1):
        next_ugly = min(uglies[p2] * 2, uglies[p3] * 3, uglies[p5] * 5)
        if next_ugly == uglies[p2] * 2:
            p2 += 1
        if next_ugly == uglies[p3] * 3:
            p3 += 1
        if next_ugly == uglies[p5] * 5:
            p5 += 1
        uglies.append(next_ugly)
    return uglies[-1]
```

这下时间复杂度就基本是 o(n) 了。:+1:

