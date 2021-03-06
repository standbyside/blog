---
title: next添加评论功能过程记录
id: add-comment-function-to-next
date: 2018-12-04 12:16:05
updated: 2018-12-04 12:16:05
categories:
  - 那些古怪又令人忧心的问题
tags:
  - hexo
  - next
  - gitment
---
虽然朋友跟我说，评论功能根本没有用，根本没人会评论（这里写一个"惨"字），但是万一哪个小可爱想夸我（划掉）和我交流技术怎么办 ⊙∀⊙，找不到地方多不爽，还是好想要把评论功能加上啊。

### 选择评论系统

首先进入[next官网](http://theme-next.iissnan.com/third-party-services.html)，发现官方居然提供五种评论系统！五种！随便挑一个闭着眼睛就能搞定啊！

它们分别是：DISQUS，Facebook Comments，HyperComments，网易云跟帖，来必力。

然而一个个试下来，呵呵，这都是什么垃圾玩意儿！（╯‵□′）╯︵┴─┴

首先是DISQUS，不开软件，永远都连接不上，永远 (-ι_- )。

然后是Facebook Comments，看名字就知道原因同上。

HyperComments，连打开它家官网都奇慢无比，尝试了一下好像也没出来（然而今天就开得很快，不知道是不是网络波动原因）。

网易云跟帖，点开直接是一个网络连接不安全。

最后是来必力，勉强算是能用。但是进官网注册，首页明明是中文，怎么其他页面都是韩文捏...最后出来的效果，不仅丑，而且不稳定，默默的又把它关闭了。

<!-- more -->

难受啊难受，一个都不争气！在挣扎了半天之后，突然想去看看朋友的网站用的什么评论系统，发现是gitalk，那next能不能集成gitalk呢？让我来搜索一番！

一搜索，真的有，但是搜索结果里不仅出来了gitalk，还提到了gitment，我回去看了看next的_config.yml，发现其实next已经支持gitment了，只是官网上没有写出来。

### 集成gitment

#### 1. 进入[github](https://github.com/settings/applications/new)新建一个认证application

![图片](http://cdn.standbyside.com/shortcut/create-github-oauth-application.jpg)

创建完后会生成这个application对应的 Client ID 和 Client Secret

#### 2. 在自己的github中创建一个同名的repository

以后每篇文章都会对应这里的一个issue，这篇文章的comments和like都会记录到对应的issue里。

#### 3. 进入主题的_config.yml里修改gitment相关属性

以下列出了必须要修改的参数

```
gitment:
	enable: true # 启用gitment
  	mint: false # 这里一定要设置成false，后面说为什么
  	github_user: # 你的github用户名，比如我就是standbyside
  	github_repo: # 刚才你创建的repository的名字，只要名字，不要全链接
  	client_id: # 你的 Client ID
  	client_secret: # 你的 Client Secret
```

做完以上三步，重新刷新网站，再打开，你的文章底下就能看到评论啦！！

但是到这一步，也就是只能看看呢，不能用的呢，根本登录不进去呢，点登录就报Object ProgressEvent呢 (-ι_- )

为啥呢？因为，写gitment的老哥弃坑了啊！不维护了啊！自己的服务器关掉了啊！在[这里](https://github.com/imsun/gitment/issues/170)能看到详细的讨论。

怎么办呢？

#### 4. 进入 gitment.swig 文件

（在我这个版本这个文件位置是 /next/layout/_third-party/comments/gitment.swig），将

```
<link rel="stylesheet" href="//imsun.github.io/gitment/style/default.css">
<script src="//imsun.github.io/gitment/dist/gitment.browser.js"></script>
```
修改为
```
<link rel="stylesheet" href="https://www.wenjunjiang.win/css/gitment.css">
<script src="https://www.wenjunjiang.win/js/gitment.js"></script>
```
就ok了。

这里就要说下为什么上面mint要改成false了，因为配置文件里写的是

```
{% if theme.gitment.mint %}
    {% set CommentsClass = "Gitmint" %}
    <link rel="stylesheet" href="https://aimingoo.github.io/gitmint/style/default.css">
    <script src="https://aimingoo.github.io/gitmint/dist/gitmint.browser.js"></script>
{% else %}
    {% set CommentsClass = "Gitment" %}
    <link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
    <script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
{% endif %}
```

不改成false就走上面的分支了，这个gitmint和gitment有什么差异我现在还不知道，我猜可能是样式更简洁吧，现在作者也不维护了，能不能搞出来就不一定了，所以也没有尝试。

最后再刷新（可能）就ok了。

### 其他gitment问题

#### Error: Not Found

检查一下 github_user 和 github_repo

#### Error: Comments Not Initialized

用自己的账号登录一下，文章底下会出现一个初始化按钮，点一下就完事，每个文章都要点一下哦～

#### Error: Bad credentials

去application里点击一下Reset client secret，重新生成一个secret。回去清下内存，刷新下页面，如果还是显示 Bad credentials，尝试点击一下评论框的Login登录一遍。

### 总结

现在总结起来仿佛很简单，但是解决问题时其实还是挣扎了挺长时间的。

所以有时候我很愿意去相信，一个复杂问题的背后，可能存在一个简洁的解决方案。找出来。
