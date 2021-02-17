+++
author = "Weiran"
title = "为知笔记+Typora搭建的私人博客"
date = "2020-10-28"
summary = "需要写一些私人markdown日志的时候，搭建博客服务器和图床"
math = true
categories = [
    "应用"
]
tags = [
    "blog",
    "为知笔记",
]
+++

### 需求

需要写一些私人markdown日志，但是搭建在线博客之类稍显繁琐。

#### 1. docker搭建为知笔记

官方教程https://www.wiz.cn/zh-cn/docker

可以ssh到服务器直接运行下列代码

```shell
sudo docker run --name wiz --restart=always -it -d -v /share/Container/wiz/date:/wiz/storage -v /etc/localtime:/etc/localtime -p 2367:80 -p 9269:9269/udp -e SEARCH=ture wiznote/wizserver
```

默认用户名admin@wiz.cn，密码123456。最多支持5用户访问。在客户端中可以选择编辑器为Typora

#### 2. 搭建Chevereto图床

主要参考https://juejin.im/post/6844904067236364295

github https://github.com/Chevereto/Chevereto-Free

下载github代码后搭建的是虚拟主机服务，和sql服务器，利用phpmyadmin来管理。

####3. 利用PicGo配合Typora上传图片

本地上传的图片将自动通过PicGo转换成图床的外链

PicGo： https://github.com/Molunerfinn/PicGo

还需要插件支持chevereto：https://github.com/acewfdy/PicGoPlugins/tree/master/picgo-plugin-chevereto

然后在typora设置一下就可以了。

<img src="http://lwrnas1.myqnapcloud.cn:2369/images/2020/10/21/ME97T5VDTCU5YNKP.png" alt="ME97T5VDTCU5YNKP" style="zoom: 80%;" /> 

#### 4. mac端的设置

mac端的PicGo在安装插件上好像有不少问题，因此我们使用uPic来代替

https://github.com/gee1k/uPic/releases

uPic也不原生支持Chevereto，但是可以设置：

-   `API 地址`: 填写上面准备好的 `[上传服务地址]`
-   `请求方式`: `POST`
-   `使用 Base64`: `勾选`
-   `文件字段名`: `source`
-   `URL 路径`: 上传完成后获取图片链接的路径。`['image', 'url']`

扩展配置

Headers

-   `Content-Type`: `multipart/form-data; charset=utf-8;`

Bodys

-   `key`: 填写上面准备好的 `[API Key]`
-   `action`: `upload`



mac端的为知笔记客户端也是残废的状态，不能使用外部编辑器，我们可以用第三方客户端 https://github.com/altairwei/WizNotePlus，并把外部编辑器设置如下：

![Screen Shot 2020-10-22 at 12.12.02 AM](http://lwrnas1.myqnapcloud.cn:2369/images/2020/10/22/Screen-Shot-2020-10-22-at-12.12.02-AM.png)

折腾完成，大功告成。

但是实际上使用为知笔记就可以包含图片一起上传了。但是，使用图床真的是好选择吗？