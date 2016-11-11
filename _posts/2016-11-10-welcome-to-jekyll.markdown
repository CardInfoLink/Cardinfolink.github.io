---
layout: post
title:  "使用 Github Pages + jekyll 搭建博客"
date:   2016-11-9 13:14:54
categories: jekyll
tags: jekyll 博客
excerpt: 本文会教你如何使用 jekyll 和 github pages 搭建属于自己的博客。
author: ChenFeng
---

### 一、什么是 Github Pages

<img src='https://pages.github.com/images/logo.svg' style="background-color:black;"/>

Github Pages 官网的标语是：

> Websites for you and your projects.

更多详情可以查看 Github Pages 官网：[https://pages.github.com/][github-pages]

当你在 Github 上辛辛苦苦写了一个很牛逼的项目，你却无法告诉全世界，这个时候该怎么办，做个项目官方网站？？你需要编码、数据库、文件服务器、打包、部署、CDN 等等，一想到这些，还是算了吧~ ~ UP! UP! UP! 别担心，有 Github Pages，她可以帮助你搭建自己的个人网站，也可以搭建你的项目官网，她不需要服务器，不需要数据库，甚至有些时候都不需要懂 html 。

好了，现在开始吧~

- github 账号

	首先你需要注册一个属于自己的 github 账号，前往 [github](https://github.com/) 注册账号。

- 创建个人网站(https://`your-account`.github.io/)：

	当你注册好一个 github 账号后，你会有一个用户名，如 `cardinfolink` ，然后，创建一个名为 `cardinfolink`.github.io 的仓库，这个仓库便是保存你个人网站的地方，接下来你只需要将自己编写好的网站提交到仓库，然后访问 [https://cardinfolink.github.io/](https://cardinfolink.github.io/) 便可以了。什么！不会写页面，没关系，后文会讲一个工具 jekyll 。

- 创建项目网站(https://`your-account`.github.io/`project-name`/)：
	
	有时候，你需要将自己的某个项目，如 `Smart-POS` ，也要制作一个网站，当然也是可以的。你只需要将这个项目创建一个名为 `gh-pages` 的分支，然后将你的页面提交到这个分支，然后访问 [https://cardinfolink.github.io/Smart-POS/](https://cardinfolink.github.io/Smart-POS/) 便可以了。

### 二、什么是 jekyll

![][jekyll-logo]

jekyll 官网的标语是：

> Transform your plain text into static websites and blogs.

很显然，就是将你的文本转换成静态网站或者博客的一个工具。

更多详情可以查看 jekyll 官网：[https://jekyllrb.com/][jekyll]

### 如何使用

- 安装 jekyll

```
	$ gem install jekyll bundler
```

- 新建项目

```
	$ jekyll new cardinfolink-site
	$ cd cardinfolink-site
```

一个 jekyll 项目的目录结构

![](../../../../image/33DFDF04-693F-4BB0-A857-EAD28195BF27.png)

- 编写文章

jekyll 默认使用 markdown 写文章，不懂如何使用 markdown 的小伙伴请移步 [markdown 的基本语法]()。找到 `_posts` 文件夹，在里面新建一个文件，文件名随便，后缀为 `.md` 或者 `.markdown` 。这里我们新建一个 `2016-11-11-new-blog.markdown` 的文件，编写内容如下：

![](../../../../image/642BAA6D-3099-45BC-8192-A3C951CC579F.png)

字段解释：

```
	---
	layout: post 				//使用的模板
	title:  "我的第一篇博客!"			//标题
	date:   2016-11-11 10:09:09 +0800	//发布时间
	categories: blog 			//文章分类
	---
	我的第一篇博客				//正文
```

- 运行

```
	$ bundle exec jekyll serve
	// => Now browse to http://localhost:4000
```

运行成功之后访问 [http://localhost:4000](http://localhost:4000) 你会看到：

![](../../../../image/47D9B6E7-6EDB-45A1-921B-308E09872686.png)

### 三、主题

想要炫酷的网站效果，却又不想编码，请直接去网上找主题吧，各种各样的等着你。

官网主题：[jekyll-themes][jekyll-themes]


[github-pages]: https://pages.github.com/
[github-pages-logo]: https://pages.github.com/images/logo.svg
[jekyll]:      http://jekyllrb.com
[jekyll-logo]: http://jekyllrb.com/img/logo-2x.png
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
[jekyll-themes]: http://jekyllthemes.org/

