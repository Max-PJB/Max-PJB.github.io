---
title: 命令行下载Google Drive大文件
categories:
  - ubutnu
tags:
  - ubuntu
  - 下载
  - Google Drive
---

# 命令行下载Google Drive大文件

目前查看的有 2 种方式：

1.  `curl` 或者 `wget` 下载
2.  *python* 脚本



### 一，查询 文件 ID

很简单，打开 *google drive* 找到想要下载的大文件，右键，点击 ==获取链接==  

![image-20210126191254664](http://cdn.ailemong.com/2021-01-01/26/21-19-12-56.png)

然后等会弹出一个分享链接。大概长这个样子：

`https://drive.google.com/file/d/0B7IzDz-4yH_HMFdiSE44R1lselE/view?usp=sharing`

`0B7IzDz-4yH_HMFdiSE44R1lselE` 就是我们要的 ==fileId== 

记住他，后面要用。

### 二，命令下载

#### 1.1 `wget`

下载小文件：

`wget --no-check-certificate ‘https://docs.google.com/uc?export=download&id=FILEID’ -O FILENAME`

替换对应的 ==FILEID==（第一步获取的） 和 ==FILENAME==（自定义） 。

下载大文件：

`wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=FILEID' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=FILEID" -O FILENAME && rm -rf /tmp/cookies.txt`

同样替换其中的 ==FILEID== 和 ==FILENAME== 即可。注意 *FILEID* 有两处。

#### 1.2 `curl`

```shell
fileId=0B7IzDz-4yH_HMFdiSE44R1lselE  #（自行替换）
fileName=想要保存的名字  # （这个可以自定义）
curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${fileId}" >/dev/null

code="$(awk '/_warning_/ {print $NF}' /tmp/cookie)"

curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${code}&id=${fileId}" -o ${fileName}
```

#### 2. Python

```python
import requests

def download_file_from_google_drive(id, destination):
    URL = "https://docs.google.com/uc?export=download"

    session = requests.Session()

    response = session.get(URL, params = { 'id' : id }, stream = True)
    token = get_confirm_token(response)

    if token:
        params = { 'id' : id, 'confirm' : token }
        response = session.get(URL, params = params, stream = True)

    save_response_content(response, destination)    

def get_confirm_token(response):
    for key, value in response.cookies.items():
        if key.startswith('download_warning'):
            return value

    return None

def save_response_content(response, destination):
    CHUNK_SIZE = 32768

    with open(destination, "wb") as f:
        for chunk in response.iter_content(CHUNK_SIZE):
            if chunk: # filter out keep-alive new chunks
                f.write(chunk)

if __name__ == "__main__":
    file_id = '0B7IzDz-4yH_HMFdiSE44R1lselE' # 自行更换
    destination = 'haha.zip' # 自定义
    download_file_from_google_drive(file_id, destination)
```

