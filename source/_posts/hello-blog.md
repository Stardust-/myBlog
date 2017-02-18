---
title: Hello Blog 
date: 2016-07-19 22:56:55 
layout: 
subtitle: 我的建站经历 
author: Stardust 
header-img: header-img.jpg 
tags: 
	- blog 
	- 经历 

---

## 前言

很早之前就想自己搭一个博客了，不过因为之前没有前端的经验，也没有那么大的渴望，一直都没有行动起来（还是因为自己懒，囧）。

大学时间过得真的好快，如果还不行动起来，说不定大学完了还没有搭好博客呢。因此在这次大二暑假实习的间隙（其实是leader出差去了，耶~）花了两天时间终于搭好了自己的博客，希望可以激励自己将以后学习的东西以文字的形式记录下来，以后回头来看说不定还可以别有一番回味。

废话说完，还是将自己遇到的坑记录下来。

## 正文

我使用的是博客框架[hexo](https://hexo.io/)搭建的静态博客，主题使用的[hux](https://github.com/Huxpro)的主题。

嘛，使用别人的东西，万变不离其宗，先配置环境。

### 配置环境

 * hexo基于[git](https://git-scm.com/)和[node.js](https://nodejs.org/en/)，git最近在国内下载超慢，还经常断掉，也许是墙更高了，如果无法忍受官网的速度，可以在我的网盘里面下载[点击下载](https://yunpan.cn/cBp9RLWgkPtja)  访问密码 cbb9
 * 然后就是安装hexo了，安装好node之后，直接使用npm（node的包管理器）安装：
    
	```bash
			$ npm install -g hexo-cli
	```

### 搭建

安装完成后使用如下命令初始化一个hexo博客

```bash
	$ hexo init folder
```
 > folder 是你在本地的路径，也就是blog的根目录，如： d:\mybog


接下来...其实已经搭建好了，恩，就是这样 ==。


使用如下命令，在本地 http://localhost:4000 就可以看到你的blog:

```bash
	hexo g 
	hexo s
```
 > g： generate，按照你的配置生成html和css
 > s： server，在本地启动服务器
 > clean：清除生成的html和css

到此为止你也就有了自己的博客雏形了，当然hexo很重要的一个步骤就是要修改配置文件_config.yml，就在init的根目录下面，官网对于每一个属性有非常详细的解释，我就不多说了。
当然也可以使用我的配置作为[参考](https://github.com/Stardust-)。

 * [英文文档](https://hexo.io/docs/)
 * [中文文档](https://hexo.io/zh-cn/docs/)

接下来最重要的就是选一个自己喜欢的主题啦，官方提供了很多不错的[主题](https://hexo.io/themes/)。你需要将主题下载下来，放到根目录的theme的目录下，然后在_config.yml文件中的theme中配置:

```bash
	theme: huxblog
```

 > huxblog 为theme的名字

之后如果想要修改的话，就要自己慢慢鼓捣了。可以根据自己的喜好添加插件或是细节调整。

如果想要在页面显示访客数的话，可以使用[不蒜子](http://service.ibruce.info/)，参考这个[博客](http://ibruce.info/2015/04/04/busuanzi/)。

当然，因为是建立的静态博客，如果要添加评论分享等功能的话，就要使用插件了，hexo默认可以使用Disqus,不过国内Disqus很慢（你懂得 ==），因此还是使用duoshuo吧，到[官网](http://duoshuo.com/)去注册一个账户，然后将username添加到配置文件中就ok啦：

```bash
	duoshuo_username: xxxxxx
```

最后当然就是发表文章：

```bash
	hexo new <layout> <post_name>
```

 > layout就是根目录下面scaffolds文件夹下的创建文章的布局文件（我觉得更像配置文件，囧）
 > post_name 就是新建文章的标题

layout的格式，看一下就懂啦，当然你也可以自定义自己常用的：
```
	---
	title: Hello Blog 
	date: 2016-07-19 22:56:55 
	layout: 
	subtitle: 我的建站经历 
	author: ranjun 
	header-img: hello-blog/header-img.jpg 
	tags: 
		- blog 
		- 经历 	
	---
```
## 后记

大致的步骤就是这些，不过期间肯定会碰到很多坑，也会走些弯路，不过碰到问题google和baidu一下也就很快解决啦，如果有问题也可以留言，我们一起爬坑~。（坑点都在细节上，主要是要定制自己喜欢的样式是个很费时间和精力的事情）

如果喜欢这个主题的话可以fork我的[项目](https://github.com/Stardust-)，有问题尽管提issue，客官不要客气，还是给颗star噻（泪汪汪）。

好吧，第一次写博客，好啰嗦啊 ==。

> 本文为原创文章，转载请注明地址： http://ranjun.me/2016/07/19/hello-blog/