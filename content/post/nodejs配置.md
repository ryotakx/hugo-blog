+++
author = "Weiran"
title = "配置nodejs"
date = "2020-02-26"
summary = "在Ubuntu服务器上配置部署NodeJS应用。"
math = true
categories = [
    "应用"
]
tags = [
    "nodejs",
    "server",
]
+++
<h2>1. 下载</h2>

```
wget https://nodejs.org/dist/v12.14.1/node-v12.14.1-linux-x64.tar.xz
```

<h2>2. 解压</h2>

```bash
tar -xvf node-v12.14.1-linux-x64.tar.xz
```

改一个简单的名字

```bash
sudo mv node-v12.14.1-linux-x64 node12
```

<h2>3. 配置安装</h2>

修改/home/ubuntu下的.bashrc文件

在最后添加：

```bash
export NODE_HOME=/home/ubuntu/node12
export PATH=$NODE_HOME/bin:$PATH
```

<h2>4. 安装forever</h2>

```bash
//全局安装
npm install forever -g 
//启动       
forever start app.js 
//关闭         
forever stop app.js           
//输出日志和错误
forever start -l forever.log -o out.log -e err.log app.js
//自动监控文件变化，文件修改保存之后自动重启app.js      
forever -w app.js  
//查看帮助           
forever -h 
```
<h2>附：设置文件链接</h2>

```bash
sudo ln /home/ubuntu/node12/bin/node /usr/local/bin/
```

