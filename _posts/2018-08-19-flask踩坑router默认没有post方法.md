---
title: flask踩坑route默认没有post方法
categories:
  - flask
  - python
  - Flask 踩坑记录
tags:
  - flask
  - Flask 踩坑记录
  - python
  - 踩坑
---
# flask踩坑route默认没有post方法

route $\color{red}{默认只有 GET 请求}$，和springboot（默认所有请求都可以）不一样

想要添加 POST，需要 ```methods=["POST", "GET"]```

```python
from flask import request, session, Blueprint, jsonify

bp = Blueprint('xcc_auth', __name__, url_prefix="/xcc")


@bp.route("/login", methods=["POST", "GET"])
def get_json():
    data = request.get_json()
    print(data)
    return jsonify({"name": "sb", "age": 2})
```
