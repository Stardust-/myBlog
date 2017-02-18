---
title: Python图片爬虫
date: 2016-07-20 21:07:42
layout:
subtitle: 网络爬虫的实现步骤 
author: Stardust
header-img: header-img.jpg  
tags: 
	- Python
	- 网络爬虫
	- Crawler 
	- Facebook 

---

> 本文为原创文章，转载请注明地址： http://ranjun.me/2016/07/20/PythonCrawler/

## 前言

之前一直对爬虫很好奇，在网上和知乎上看到过许许多多有趣的爬虫，一直想自己写一个来玩玩，但是没有计划的事情总是实施不起来。机缘巧合之下，接触到了Python，发现Python真是一门简洁而强大的语言，然后在暑假实习的诱惑之下，开始了第一个爬虫的之旅。

第一个爬虫是抓取的日本社交网站[mixi](https://mixi.jp/)用户的公开图片，不过这个神奇的网站对爬虫的限制简直到了影响用户体验的程度，所以对这个网站完全没有那种玩爬虫的快感。

因此，今天我还是来写写我的第二个爬虫-----对[facebook](https://facebook.com)用户图片的抓取。

![facebook](facebook.jpg)


## 正文

### 爬行策略

网上貌似有很多爬虫的框架，不过灵活性不太好的样子，爬虫这种东西还是要根据自己的需求来用。因此自己动手，丰衣足食啊。

一般来说，先不要急着动手写，先制定好爬行策略，第一步就是要分析网页，找到自己的想要的资源的位置。利用好浏览器的web开放工具`F12`，将会事半功倍。

根据自己的观察和探索，发现每一个用户都对应一个id，类似于这种`100010625445795`，然后数字这么大，猜测第一个1不代表用户的数目，经过维基的资料，查到facebook的用户2015年的时候大概20亿，那么去掉前面几位，剩下的就比较接近查到的用户数了`10625445795`。经过思考，决定就使用用户id依次爬取。

当然，还可以使用用户的好友圈，来抓取，但是这样必须维护两个集合来保存爬到的用户地址和已经抓取过得用户，在基数变大之后，查询所消耗的时间将会大大增加，因此我就选择了根据用户id来进行爬取。

### 分析页面

跟着策略，进入到用户的主页，发现页面的html结构是动态生成，图片的大图经过ajax加载，因此需要花费一些时间好好研究页面结构然后找到自己需要的资源。

## Coding

##### 模拟登陆

要进入用户的主页并查看相册，需要模拟浏览器的登陆，并获得cookies。

这里，我使用了第三方库 [requests](http://docs.python-requests.org/en/master/) 强烈推荐，简单好用。只需要一行代码，就能绑定cookies,当然不用每次都去请求，将cookies保存在本地以便下次使用：
```python

	import requests

	s = requests.Session()
	s.cookies = LWPCookieJar(path + 'cookies')
	s.cookies.save() #保存cookies

	s.cookies.load() #下次使用直接加载
```

 > path:保存的路径

然后就是模拟登陆了：

```python


login_params = {
    'email':USER_ACCOUNT,
    'pass':USER_PASSWORD
}
login_res = s.post(url = URL_LOGIN,	
    data = login_params, 
    headers = headers,	
    timeout = 10)
```

 > header为模拟浏览器的请求头，在chrome浏览器中按F12，选择network，就能看到每一条请求的具体参数。
 > timeout: 设置超时时间，10秒。

login_res就是登陆后的页面，成功则跳转到你的主页，失败则还是登陆页面，可以将login_res输出到文件，打开查看是否登陆成功：

```python

	file = open('test.html','w')
	file.write(login_res.text)
	file.close()
```

##### 图片抓取

登陆成功之后就按照之前制定的爬取策略开始行动，这里解析html可以使用正则，但是比较麻烦，所以选择了第三方库 [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)。BeautifulSoup会将html解析为一颗文档树，html在最上一层，接下来就是body和head，依次类推，官方有非常详细的文档说明：

 * [中文文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)
 * [英文文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)


```python
	
	from bs4 import BeautifulSoup

	page = s.get(url = url, headers = headers, timeout = 10)	
	soup = BeautifulSoup(page.content)
	divs = soup.findAll('div', attrs = {
				'class' : 'hidden_elem'
			})
```

上面的代码就是一个简单的例子，可以得到类名为`hidden_elem`的所有`div`元素。

之后找到了图片的链接就可以下载保存到本地了：

```python

	data = requests.get(img_url,timeout = 10, headers = headers).content
	fileName = str(int(time.time())) + '.jpg'
	filePath = download_path + '\\' + fileName
	image = open(filePath, 'wb')
	image.write(data)
	image.close()
```

##### 优化效率

爬到图片之后，发现爬的速度真是感人，那么就要想想办法来提升。爬虫的瓶颈主要在于网络，因此我们使用多线程来优化爬取速度：

```python

import thread

def crawl(name,delay):
    global USER_ID
    while USER_ID > 0:		
        time.sleep(delay)
        start()


try:
    thread.start_new_thread(crawl,('thread-1',0.1))
    thread.start_new_thread(crawl,('thread-2',0.2))
    thread.start_new_thread(crawl,('thread-3',0.3))
    thread.start_new_thread(crawl,('thread-4',0.4))
except Exception,e:
    print 'thread error:',e
```

 > crawl: 开始爬取的入口函数


## 效果

![effct](effect1.png)

</br></br>

![effct](effect2.png)

</br></br>

![effct](effect3.png)

## 后记

一个爬虫完成大概就是这些步骤，不过其中还是有很多细节需要自己体会。爬过坑，下次才不会被坑是吧~

如果对我实现的这个爬虫感兴趣，源码在这里 https://github.com/Stardust-/PythonCrawler 如果有什么问题，欢迎留言和提issue，喜欢的话，加颗star吧。

PS: 不要爬太快了哦，小心被限制账号哦，不要问我是怎么知道的 ，一切都是泪啊 = =。

PPS：其实也就限制访问其他用户主页7天而已，其他功能可以继续使用。

> 本文为原创文章，转载请注明地址： http://ranjun.me/2016/07/20/PythonCrawler/