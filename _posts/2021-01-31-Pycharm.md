---
title: Pycharm code template with reuseable content
categories:
  - pycharm
tags:
  - pycharm
  - idea
  - include
  - code template
---

# Pycharm code template with reuseable content

References:

[https://www.jetbrains.com/help/pycharm/settings-file-and-code-templates.html#toolbar](https://www.jetbrains.com/help/pycharm/settings-file-and-code-templates.html#toolbar)

[https://www.jetbrains.com/help/idea/parse-directive.html](https://www.jetbrains.com/help/idea/parse-directive.html)

1. File -> Setting -> Editor > File and code Templates
2. select includes tab, type our custom common reused header:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
-------------------------------------------------

    @   Author  :       pengjb
    @   date    :       ${DATE}
    @   IDE     :       PyCharm
    @   GitHub  :       https://github.com/JackyPJB
    @   Contact :       pengjianbiao@hotmail.com
-------------------------------------------------
    Description :       
-------------------------------------------------
"""
```

![image-20210131005527986](/public/img/image-20210131005527986.png)

3. Remember our reused template file name ==python_header.py== defined before.  Select Files -> Python Script -> type the include script:  `#parse("python_header.py")`

![image-20210131010134828](/public/img/image-20210131010134828.png)

4. Press OK and Test:

   New a Python File !