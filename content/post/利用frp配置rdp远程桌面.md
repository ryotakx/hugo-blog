+++
author = "Weiran"
title = "利用frp配置rdp远程桌面"
date = "2020-02-28"
summary = "利用frp反向代理配置一个外网也可以访问的Windows远程桌面。"
math = true
categories = [
    "应用"
]
tags = [
    "frp",
    "rdp",
    "远程桌面"
]

+++

>在外面用笔记本单机跑实验时遇到诸多不便，在服务器端debug费时费力。因此希望能远程到自己宿舍中的台式机来运行一些轻度实验。其他的选项有Teamviewer和TNC Viewer，但是他们用的都是远程投屏的方式，没有声音且感觉不是很安全。因此我选择配置frp服务器的方法来反向代理远程桌面，再笔记本上连接服务器访问。

<h2>1. 准备</h2>
frp的仓库在[这里](https://github.com/fatedier/frp)。
考虑到延迟问题，首先需要准备一个地点和带宽合适的服务器。
在服务器端下载frp并解压：

```bash
wget https://github.com/fatedier/frp/releases/download/v0.31.2/frp_0.31.2_linux_amd64.tar.gz
tar -zxvf frp_0.31.2_linux_amd64.tar.gz
```

配置`frps.ini`文件：

```
[common]
  bind_addr = 0.0.0.0
  bind_port = 13752 #服务器远程开放的访问端口

  dashboard_port = 13999
  dashboard_user = username #登录dashboard的用户名密码
  dashboard_pwd = password
```

之后在本地Windows下载对应版本的frp，修改`frpc.ini`文件：

```
[common]
server_addr = 11.22.33.44 #远程服务器地址
server_port = 13752 #服务器远程开放的访问端口

[rdp]
type = tcp
local_ip = 127.0.0.1
local_port = 3389 #3389是微软远程桌面默认端口
remote_port = 13701 #访问rdp服务的端口
```

<h2>2. 运行</h2>

在服务器端运行：

```bash
./frps -c frps.ini
# nohup ./frps -c frps.ini 常驻后台
```
然后可以访问ip:13999来访问一下dashboard。

在window端运行：

```bash
.\frpc
```

没有报错即可。

<h2>3. windows开机自启以常驻后台</h2>

利用winsw把frpc注册为系统服务。首先在这里下载[winsw](https://github.com/kohsuke/winsw/releases)，解压至frp文件夹内。为了方便可以重命名为winsw.exe。新建一个winsw.xml文件，写入：

```xml
<service>
    <id>frp</id>
    <name>frp-remote-desktop</name>
    <description>allow frp service</description>
    <executable>frpc</executable>
    <arguments>-c C:\Users\Li\Desktop\frp\frpc.ini</arguments>
    <onfailure action="restart" delay="60 sec"/>
    <onfailure action="restart" delay="120 sec"/>
    <logmode>reset</logmode>
</service>
```

然后运行命令：

```bash
# 开始
.\winsw install
.\winsw start

# 关闭
.\winsw stop
.\winsw uninstall
```

可以在`winsw.out.log`文件里看到正确的log输出。

在笔记本上，远程连接的地址为ip:13701。

