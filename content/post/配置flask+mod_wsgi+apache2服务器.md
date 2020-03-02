+++
author = "Weiran"
title = "配置Flask Web服务器"
date = "2020-02-27"
summary = "在Ubuntu18.04 上配置完整的Flask + wsgi + Apache2服务器。"
math = true
categories = [
    "应用"
]
tags = [
    "flask",
    "server",
]
+++
> Flask+mod_wsgi+apache2安装在ubuntu18.04 Google Cloud服务器上


<h2>1. 安装必要环境</h2>

```bash
# 更新安装Ubuntu系统依赖
sudo apt-get update
sudo apt-get upgrade
# 安装wsgi和apache2
sudo apt-get install libapache2-mod-wsgi-py3
sudo apt-get install apache2
# 允许wsgi mod
sudo a2enmod wsgi
# 下载miniconda, 速度缓慢的话可以换国内源
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# 安装miniconda至/home/liweiran97/
bash Miniconda3-latest-Linux-x86_64.sh
# 重新加载.bashrc
source ~/.bashrc
```

<h2>2. 搭建flask环境</h2>

```bash
# 新建虚拟环境
conda create -n flaskapp python=3
conda activate flaskapp
# 这时已经进入flaskapp虚拟环境了
conda install flask
# 搭建flask
cd /var/www/
mkdir flaskapp
cd flaskapp
mkdir flaskapp
cd flaskapp
mkdir static
mkdir templates
# 写入init文件
nano __init__.py
```

`__init__.py`文件的内容是这样的：

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def homepage():
    return "Hello?在吗"


if __name__ == "__main__":
    app.run()
```

<h2>3. 新建apache2的配置文件和wsgi配置文件</h2>

```bash
# apache2 配置文件
nano /etc/apache2/sites-available/FlaskApp.conf
```

写入内容：

```
<VirtualHost *:80>
                ServerName weiran97.com
                ServerAdmin liweiran97@gmail.com
                WSGIScriptAlias / /var/www/flaskapp/flaskapp.wsgi
				WSGIDaemonProcess flaskapp python-path=/var/www/flaskapp:/home/liweiran97/miniconda3/envs/flaskapp/lib/python3.8/site-packages
				WSGIProcessGroup flaskapp
                <Directory /var/www/flaskapp/flaskapp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/flaskapp/flaskapp/static
                <Directory /var/www/flaskapp/flaskapp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```bash
# wsgi配置文件
nano /var/www/flaskapp/flaskapp.wsgi
```

写入内容：

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/flaskapp/")

from flaskapp import app as application
application.secret_key = '123456'
```

<h2>4. 启动服务</h2>

```bash
sudo a2ensite flaskapp
sudo service apache2 reload
# 每次更改内容后重启服务器
sudo service apache2 restart
```

<h2>5. 配置301跳转</h2>

想要www域名跳转至非www域名，可以更改默认的配置文件

```bash
nano /etc/apache2/sites-available/000-default.conf
```

写入一行：

```
redirect 301 / http://weiran97.com/
```

<h2>6. 解决winscp没有写入权限的问题</h2>

更改站点配置，高级选项-SFTP中，将SFTP服务器的值改为sudo /usr/lib/openssh/sftp-server

