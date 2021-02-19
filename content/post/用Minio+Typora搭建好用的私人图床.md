+++
author = "Weiran"
title = "用Minio+Typora搭建好用的私人图床"
date = "2021-02-19"
summary = "私人博客感觉还是不够自由啊"
math = true
categories = [
    "应用"
]
tags = [
    "blog"
]

+++

> 既然重新把公开的博客捡了起来，那还是不要用为知笔记这样难用的东西吧？Chevereto和PicGo/uPic也实在称不上好用。换更简单清晰的Minio试试吧。

### 前提

之前一直用的Chevereto和为知笔记在windows端还行，在mac端就已经很难用了。而且为知笔记的备份导出竟然不是原markdown格式的，这样我一时难以接受。Chevereto和PicGo搭配使用，配置也是相当复杂。于是我捡起了当年搭建的Hugo博客，那么唯一需要解决的就是图床的问题了。

### 1. 搭建Minio服务

![Minio](http://pic.weiran97.cn/pics/115419.png)

Minio是类似阿里云OSS的开源对象存储服务。类似私人云盘，可以存储各种格式的数据。在服务器上是以文件夹的形式保存的，意味着很容易备份和迁移。其实Minio更多被用在多用户分布式的存储服务中，而且支持高访问量，单机使用实乃大材小用。

* 下载Minio Server并启动

```bash
# 下载速度很慢，可以试试本机下载后上传，或者换镜像站dl.minio.org.cn
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
./minio server /data #改为自己的想要存储的地方，通常需要支持外网访问
```

* 修改密码

在刚刚启动server的地方可以修改，可以找到.minio/config.json里修改accessKey和secretKey。本机的默认访问地址是localhost:9000。

```bash
./minio server /data & # 后台运行
ps -aux | grep minio # 查找进程号
kill -9 12345 # 杀进程
```

在管理后台可以上传下载文件或定时分享。

* Apache代理访问已上传图片

在/etc/apache2/sites-available/000-default中增加如下字段

```xml
<VirtualHost *:80>
    ServerName your.server.name
    DocumentRoot /your/data/location
    ErrorLog ${APACHE_LOG_DIR}/your.server.name-error.log
    CustomLog ${APACHE_LOG_DIR}/your.server.name-access.log combined
</VirtualHost>
```

如果你想给后台管理页面也代理上，先启动mod_proxy模块

```bash
sudo a2enmod proxy
```

然后在同样位置增加字段

```xml
<VirtualHost *:80>
    ServerName your.server.name

    ErrorLog ${APACHE_LOG_DIR}/your.server.name-error.log
    CustomLog ${APACHE_LOG_DIR}/your.server.name-access.log combined

    ProxyRequests Off
    ProxyVia Block
    ProxyPreserveHost On

    <Proxy *>
         Require all granted
    </Proxy>

    ProxyPass / http://你的服务器ip:9000/
    ProxyPassReverse / http://你的服务器ip:9000/
</VirtualHost>
```

记得在域名解析里把子域名的A记录给加上。然后尝试访问一下？

### 给Typora安排上自动上传的功能

Typora有插入图片自动触发脚本并替换图片地址的方便的功能，当然，用PicGO也是一样的功能。用Python编写如下的代码。记得pip安装requests和minio两个模块。

```python
import os
import time
import uuid
import sys
import requests
from minio import Minio
from minio.error import InvalidResponseError
import warnings

warnings.filterwarnings('ignore')
images = sys.argv[1:]
minioClient = Minio("你的后台管理地址:9000",
                    access_key='你的账号', secret_key='你的密码', secure=False)
result = "Upload Success:\n"
date = time.strftime("%Y%m%d", time.localtime())


def download(image_url):
    local_path = os.getcwd() + "/temp"
    r = requests.get(image_url, verify=False)
    with open(local_path, "wb") as code:
        code.write(r.content)
    return local_path


for image in images:
    if os.path.isfile(image):
        file_type = os.path.splitext(image)[-1]
        new_file_name = str(uuid.uuid1()).replace('-', '') + file_type
    elif image.startswith("https://") or image.startswith("http://"):
        if image.endswith(".png") or image.endswith(".jpg") or image.endswith(".jpeg") or image.endswith(".gif"):
            url = image.split("/")
            if len(url) > 1:
                image = download(image)
                new_file_name = url[-1]
            else:
                result = result + "error:parsing image error!"
                continue
        else:
            result = result + "error:parsing image error!"
            continue
    else:
        result = result + "error:parsing image error!"
        continue
    try:
        minioClient.fput_object(bucket_name='pics', object_name=date+"-"+new_file_name, file_path=image)
        if image.endswith("temp"):
            os.remove(image)
        result = result + "http://你的域名" + "/你的bucket名/" + date+"-"+new_file_name + "\n"
    except InvalidResponseError:
        result = result + "error:" + "\n"

print(result)
```

在后台里的命名方式是日期+uuid。

然后在Typora界面中填上你的Python脚本的运行指令就行了。也可以设置为对网络位置的图片一并转发。这样就大功告成了！

<img src="http://pic.weiran97.cn/pics/20210219-43e8aedc72ca11ebb7d7acde48001122.png" alt="image-20210219235100880" style="zoom:50%;" />



