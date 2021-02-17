+++
author = "Weiran"
title = "解决Jellyfin外挂中文字幕变成方块的问题"
date = "2020-09-20"
summary = "Jellyfin怎么这么难用QAQ。"
math = true
categories = [
    "NAS"
]
tags = [
    "jellyfin"
]

+++

问题相关的github issue：https://github.com/jellyfin/jellyfin-web/issues/934

本文的解决办法也是参考了上述issue。

问题来源：jellyfin的服务端都依赖于一个叫jellyfin-web的框架，其中缺乏对CJK中文字库的支持。外挂的ass特效字幕通常指定了字体，如果服务端没有这个字体就会出现方块。


# 方法1：

重新部署jellyfin的docker镜像，把/share/xxx/fonts（自己指定的存放字体的文件夹）挂载到对应容器的/usr/share/fonts中，其他设置不变。这一步可以用portainer比较方便完成，用自带的docker管理软件比较麻烦。然后在刚刚指定的fonts文件夹里放上外挂字幕所需的字体（通常字幕组在文件下载的时候会一并提供）或者以记事本打开.ass文件看一下需要哪些字体。

最后在客户端的设置/字幕/烧录字幕选项中，选择所有高级特效字幕。然后刷新并选定相应字幕就完成了。

缺点：1. 由于是在服务器端把字符烧录进入视频流，因此所有视频都会被强制转码。切换字幕的时候整个视频流会停止。服务器负载大。

2. 安卓手机客户端中目前没有这个烧录字幕选项，因此安卓手机观看还是方块。ios端正常，tv端没有测试。

 

# 方法2：

修改docker中的jellyfin-web和字库相关的代码。开启nas的ssh连接，用putty进入nas。

首先可以在container station里找到jellyfin的dockerID，我的是e29daf1852d7

把相关的js代码拷出来

docker cp e29daf1852d7:/usr/lib/jellyfin/bin/jellyfin-web/components/htmlvideoplayer/plugin.js /share/nas1(自己可以访问的共享文件夹)

打开这个plugin.js文件，找到renderWithSubtitlesOctopus这个function（大约在1054行，和版本有关），原来的代码是这样的

```javascript
function renderWithSubtitlesOctopus(videoElement, track, item) {
            var attachments = self._currentPlayOptions.mediaSource.MediaAttachments || [];
            var apiClient = connectionManager.getApiClient(item);
            var options = {
                video: videoElement,
                subUrl: getTextTrackUrl(track, item),
                fonts: attachments.map(function (i) {
                    return apiClient.getUrl(i.DeliveryUrl);
                }),
```

把它改成这样：

```javascript
        function renderWithSubtitlesOctopus(videoElement, track, item) {
            var attachments = self._currentPlayOptions.mediaSource.MediaAttachments || [];
            var attachmentsFonts = attachments.map(function (i) {
                return i.DeliveryUrl;
            });
            var options = {
                video: videoElement,
                subUrl: getTextTrackUrl(track, item),
                fonts: attachmentsFonts.concat(appRouter.baseUrl() + "/libraries/NotoSerifCJKsc-Medium.woff2"),
```

主要改动就是指定了fonts字库的路径。这个路径也可以自己修改到方便的位置。NotoSerifCJKsc-Medium.woff2是简体中文的字库，下载位置：

https://github.com/CodePlayer/webfont-noto/blob/master/dist/NotoSerif/NotoSerifCJKsc-hinted/subset/NotoSerifCJKsc-hinted-standard/NotoSerifCJKsc-Medium.woff2

然后把修改过的代码和字库都导入相应的位置：

docker cp /share/nas1/plugin.js e29daf1852d7:/usr/lib/jellyfin/bin/jellyfin-web/components/htmlvideoplayer/

docker cp /share/nas1/NotoSerifCJKsc-Medium.woff2 e29daf1852d7:/usr/lib/jellyfin/bin/jellyfin-web/libraries/

把烧录选项调回自动，刷新客户端可以看到字幕已经正常了。

缺点：不能应用ass文件指定的字体。而且修改的代码只对网页端访问生效。安卓端还是没法用。

 

# 3. 修改安卓客户端

由于jellyfin的各客户端都是以jellyfin-web作为底层框架编写的，所以可以用一样的方法修改安卓客户端。

准备工作：

1. 下载jellyfin安卓客户端：

https://repo.jellyfin.org/releases/client/android/jellyfin-android-release-v1.0.1.apk

2. 下载Java开发环境JDK

步骤：

1. 将下载的apk文件改名为jellyfin.apk，然后打开方式选择winrar。
2. 把之前修改好的js文件放进jellyfin.apk\assets\www\components\htmlvideoplayer替换源文件。
3. 把下载的字库NotoSerifCJKsc-Medium.woff2放进jellyfin.apk\assets\www\libraries文件夹里。
4. 删除META-INF文件夹。修改客户端就完成了。但是现在没法安装，因为修改后的客户端还没有签名。
5. 拷贝下载好的jdk开发环境中bin文件夹中的keytool.exe，jarsigner.exe，放到和apk同文件夹中。
6. 打开cmd/powershell，进入所在文件夹。输入

keytool -genkey -alias key.keystore -keyalg RSA -validity 30000 -keystore key.keystore

然后适当填写一些信息，之后会生成一个key.keystore的秘钥文件。记住刚刚写的秘钥库的密码。

jarsigner -verbose -keystore key.keystore -signedjar jellyfin_signed.apk jellyfin.apk key.keystore

签名完成，把生成的jellyfin_signed.apk导入手机就能安装了。

 

# 附：解决jellyfin扫描文件库过于缓慢的问题。

修改/Container/container-station-data/lib/docker/containers/e29...文件夹中的hosts文件。e29..是jellyfin的docker ID

在hosts文件后增加两行

13.225.103.51        api.themoviedb.org
13.224.157.34        api.thetvdb.com

每次重启docker后失效