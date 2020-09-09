---
title: git commit 后撤销commit 或者修改注释
categories:
  - git
tags:
  - git commit 撤销
  - git commit 注释
---

# git commit 后撤销 commit

我们一般这样：

```python
git add .
git commit -m "注释"
```

执行完 commit 后想要撤回，怎么办？

这样拌： `git reset --soft HEAD^`

> HEAD^ 的意思是上一个版本，也可以写成 `HEAD~1`
> 如果进行了 2 次 commit，都想撤回，可以使用 `HEAD~2`

除了 `--soft` 还有 `--mixed` 和 `--hard` 参数

> `--soft`: 撤销 git commit 不撤销 git add
> `--mixed`: 撤销git commit 和 git add
> `--hard`: 删除工作空间改动代码，撤销commit，撤销 git add . 这个慎用，会把你这次的改动都给删了

## 只想要修改 commit 的注释呢？

`git commit --amend`

进入修改，ctrl+O 保存，回车确定，ctrl+x退出，结束！

`git show` 查看commit效果
