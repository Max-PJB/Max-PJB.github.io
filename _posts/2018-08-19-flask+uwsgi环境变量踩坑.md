---
title: flask+uwsgi环境变量踩坑
categories:
  - flask
  - python
  - Flask 踩坑记录
tags:
  - flask
  - Flask 踩坑记录
  - python
  - uwsgi
  - 踩坑
---
# flask 设置环境变量
在生产环境中设置 export XXX = yyy
在开发中 pycharm 修改 edit （运行）configuration 修改 Environment ，添加 XXX = yyy

## flask 获取环境变量
```
import os
XXX = os.environ.get("XXX")
```

## 根据环境的不同，选择不同的配置文件
```
from flask import Flask
import os
from config import load_config


app = Flask(__name__)

mode = os.environ.get("MODE")
config = load_config(mode)

app.config.from_object(config)
```

### 附上 config.__init__.py 的内容
```
def load_config(mode):
    """Load config."""
    try:
        if mode == 'PRO':
            from .production import ProductionConfig
            return ProductionConfig
        elif mode == 'TEST':
            from .testing import TestingConfig
            return TestingConfig
        else:
            from .development import DevelopmentConfig
            return DevelopmentConfig
    except ImportError:
        from .default import Config
        return Config
```
![1559408519(1).png](https://i.loli.net/2019/06/02/5cf2b036d1f1756386.png)

## 好的下面就是坑了
## flask 通过 uwsgi 部署的时候， `os.environ.get('xxx')` 他喵的无法获取到环境变量
太坑了，太坑了，太坑了，重要的事情说三遍。不管是修改 `~/.bashrc` 还是 `/etc/profile` 都没用，就是获取不到啊

没办法，只能默认是 `PRODUCTION` 模式，然后再开发的时候设置 `developmemt` 吧，这个坑啊，再吐槽一遍

### 附上 config.__init__.py 的内容
```python
import os


def load_config():
    """Load config."""
    # 这也是为了解决 uwsgi 中的 os.environ.get('MODE') 不起作用，物理吐槽这个坑
    mode = os.environ.get('MODE')
    try:
        if mode == 'development':
            from .development import DevelopmentConfig
            return DevelopmentConfig
        else:
            from .production import ProductionConfig
            return ProductionConfig

    except ImportError:
        from .default import Config
        return Config
```
