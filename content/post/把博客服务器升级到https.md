+++
author = "Weiran"
title = "把服务器升级到https"
date = "2021-02-20"
summary = "利用Let's Encrypt傻瓜式把图床升级到https"
math = true
categories = [
    "应用"
]
tags = [
    "server",
]

+++

> 本文主要参考 [文章](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04) 非常详尽
### 前提

昨天愉快地把Github Page上部署的博客都接入了自己的图床。但是在Chrome上居然无法正常显示图片，因为在HTTPS页面里无法请求HTTP资源。被迫推动HTTPS的进程。本文主要是在Apache2上搭建。

### 步骤

1. 安装Certbot客户端

   ```bash
   sudo add-apt-repository ppa:certbot/certbot
   sudo apt install python-certbot-apache
   ```

2. 检查防火墙状态

   ```bash
   sudo ufw status
   
   # 若需要开启
   sudo ufw allow 'Apache Full'
   sudo ufw delete allow 'Apache'
   ```

3. 开启SSL认证

   ```bash
   sudo certbot --apache -d your_domain -d www.your_domain
   ```

   这里还可以选择是否要把HTTP请求转发到HTTPS。然后竟然就好了？

4. 自动更新SSL认证

   ```bash
   # 每90天自动认证
   sudo systemctl status certbot.timer
   # 测试
   sudo certbot renew --dry-run
   ```

   

   



