---
layout: post
title:  "如何在这里发布自己的文章"
date:   2016-11-9 15:14:54
categories: cardinfolink
tags: cardinfolink 教程
excerpt: 你可以在这里记录在讯联的点点滴滴，无论技术分享还是生活随笔。
author: ChenFeng
---

### Fork 仓库

访问讯联仓库地址 [https://github.com/CardInfoLink/Cardinfolink.github.io](https://github.com/CardInfoLink/Cardinfolink.github.io) 并 Fork 到自己的某个组织。

![](../../../../image/E0646CE1-7729-445C-88EC-2094EAE45D52.png)

### clone 仓库

```
	$ git clone git@github.com:xxxx/Cardinfolink.github.io.git
	// 替换成正确的地址
```

### 新增文章

- 打开项目

使用文本编辑器打开上一步 clone 下来的项目，并在 `_post` 目录下新增自己的文章，如图：

![](../../../../image/662AE1C7-2B48-4E96-AE6E-2DCB5C696526.png)

- 文件名命名规范：

```
	YYYY-MM-DD-blog-title.markdown //年-月-日-文章名字.markdown
```
如：`2016-11-10-how-add-blog.markdown`

- 文章内容

![](../../../../image/0C4399E3-2BF9-4A57-9212-7C6CEAFF9B11.png)

抬头部分字段解释：

```
	---
	layout: post 			//使用的模板，默认
	title:  "如何在这里发布自己的文章"	//文章题目
	date:   2016-11-9 15:14:54	//发布时间
	categories: cardinfolink 	//分类
	tags: cardinfolink 教程		//标签
	excerpt: 你可以在这里记录在讯联的点点滴滴，无论技术分享还是生活随笔。//概要
	author: ChenFeng		//作者
	---
```

文章默认使用 markdown 格式编写，不熟悉 markdown 的同学请移步 [markdown 的基本语法]()。

### 运行调试

当文章写好时，你可能需要预览下效果，打开终端，进入到项目根目录下，执行下面的命令

```
	$ jekyll s
	// Server address: http://127.0.0.1:4000/
```
然后打开浏览器，访问地址 [http://localhost:4000/](http://localhost:4000/)

### 提交代码

当调试 OK ，预览效果很满意，便可以提交你的代码了。

```
	$ git add .
	$ git commit -m "my new blog"
	$ git push origin master
```

还没搭好 jekyll 环境的同学请移步 [使用 Github Pages + jekyll 搭建博客](https://cardinfolink.github.io/2016/11/09/welcome-to-jekyll/)

### 发送 PR

登录到 github 网站，找到你第一步 Fork 的仓库 `Cardinfolink.github.io` ，点击下面的 `New pull request` ，然后发送 PR 。

![](../../../../image/CFCB3040-C740-400D-AF15-6B93DF8E1D44.png)

等待项目管理员审核通过，便可以在 [https://cardinfolink.github.io/](https://cardinfolink.github.io/) 看到你的文章了！

