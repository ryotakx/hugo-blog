+++
author = "Weiran"
title = "配置Hugo博客"
date = "2020-02-28"
summary = "搭建一个简洁风格的Hugo博客并部署在github.io静态博客页面上。"
math = true
categories = [
    "应用"
]
tags = [
    "blog",
    "hugo",
]
+++
>一直想搭建一个简单的博客，用Flask之类的动态后端写工作量太大，而且博客也不需要过多复杂的功能。Hugo提供了丰富的模板库，对不熟悉前端的开发者也很友好。Hugo的文档可以看[这里](https://gohugo.io/getting-started/quick-start/)。本文主要以MacOS为例。
<h2>1. 下载</h2>

```bash
# windows (需要安装chocolatey)
choco install hugo -confirm
# macos (需要安装homebrew)
brew install hugo
# 检查安装
hugo version
```

<h2>2. 配置工程</h2>
我这里选择了[hugo-notepadium](https://github.com/ryotakx/hugo-notepadium/)主题。为了之后进一步的修改，我先fork了一份到我的github。这里用了git的submodule模块来管理子模块依赖，我在另一篇文章里介绍。

```bash
hugo new site hugo-blog
# 添加git以及安装主题
cd hugo-blog
git init
git submodule add https://github.com/ryotakx/hugo-notepadium themes/hugo-notepadium
```

之后，修改hugo-blog/config.toml文件来配置网站。通常不同的主题有很多可以配置的选项，因此需要拷贝一份刚刚下载的themes/hugo-notepadium/exampleSite中的配置文件。也可以把里面的其他文件（实例博客）一起拷贝了方便修改。

<h2>3. 开发</h2>

hugo提供了一个开发服务器：

```bash
hugo server -D
```

默认的本地URL是 http://localhost:1313/ 。写作时向content/post文件夹里添加新的.md文件的时候会自动检测更新。要生成可供部署的静态网页：

```bash
hugo -D
#如果要更改部署的url
hugo -D -b https://ryotakx.github.io/
```

<h2>4. 修改theme</h2>

hugo-notepadium支持基于Disqus的评论系统。但是感觉界面太难看而且需要登录才能评论，我选择了简洁风格的不用登陆的评论系统[Valine](https://valine.js.org/index.html)。Valine需要注册LeanCloud账户，在hugo上配置的话我参考了这篇[文章](https://www.smslit.top/2018/07/08/hugo-valine/)。简单地说，先获取LearnCloud中的AppID和AppKey，然后添加配置选项：

```toml
  # Valine.
  # You can get your appid and appkey from https://leancloud.cn
  # more info please open https://valine.js.org
  [params.valine]
    enable = true
    appId = '你的appId'
    appKey = '你的appKey'
    notify = false  # mail notifier , https://github.com/xCss/Valine/wiki
    verify = false # Verification code
    avatar = 'mm' 
    placeholder = '说点什么吧...'
    visitor = true
```

然后修改相对应的主题模板文件themes/hugo-notepadium/layouts/partials/中的article-comments.html，需要调整一下cdn：

```html
<!-- valine -->
  {{- if .Site.Params.valine.enable -}}
  <!-- id 将作为查询条件 -->
  <span id="{{ .URL | relURL }}" class="leancloud_visitors" data-flag-title="{{ .Title }}">
    <span class="post-meta-item-text">文章阅读量 </span>
    <span class="leancloud-visitors-count">1000000</span>
    <p></p>
  </span>
  <div id="vcomments"></div>
  <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
  <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
  <script type="text/javascript">
    new Valine({
        el: '#vcomments' ,
        appId: '{{ .Site.Params.valine.appId }}',
        appKey: '{{ .Site.Params.valine.appKey }}',
        notify: {{ .Site.Params.valine.notify }}, 
        verify: {{ .Site.Params.valine.verify }}, 
        avatar:'{{ .Site.Params.valine.avatar }}', 
        placeholder: '{{ .Site.Params.valine.placeholder }}',
        visitor: {{ .Site.Params.valine.visitor }}
    });
  </script>
  {{- end }}
```

我这里想把文章阅读量放在文章标题栏的下面，因此我把\<span>放在了article-header.html。

<h2>5. 在github.io上部署</h2>

首先在github上创建一个和自己的用户名重名的仓库，我的就是ryotakx.github.io。然后把刚刚生成的public文件夹的内容上传进github:

```bash
cd public
git init
git remote add origin https://github.com/ryotakx/ryotakx.github.io.git
git add .
git commit -m "update info"
git push -u origin master
# 然后输入Github用户名和密码
```

然后访问 https://ryotakx.github.io 就可以看到部署好的网页了。
