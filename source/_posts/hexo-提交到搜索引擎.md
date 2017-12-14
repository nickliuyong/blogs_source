---
title: hexo 提交到搜索引擎
date: 2016-10-19 13:25:41
tags:  
    - hexo
---

### 前言
```
本文主要是为了搜索引擎可以更加友好的搜到我们指定的站点内容。
参考文章：http://www.jianshu.com/p/619dab2d3c08
https://codingbubble.github.io/2015/05/08/custom-your-Hexo/
http://azeril.me/blog/Markdown-Syntax.html
```

### 提交到chrome搜索引擎

```
可以通过如下方式来查看chrome是否已经收录我们的站点
site:https://nickliuyong.github.io/blogs/
```


### 提交到搜索引擎  

[Google搜索引擎入口](https://www.google.com/webmasters/tools/home?hl=zh-CN  "访问chrome")

<!--more-->
### 验证网站

![chrome验证](/blogs/images/hexo-chrome-1.png)

> 将上访获取到的meta标签添加到hexo/themes/yillia/layout/_partial/head.js中  
> 验证通过后通过site:https://nickliuyong.github.io/blogs/可以查看到搜索引擎已经添加我们指定的站点

### 站点地图

 > 通过站点地图可以将您网站内容的组织架构告知Google和其他搜索引擎。  
 > Googlebot等搜索引擎网页抓取工具会读取此文件，以便更加智能地抓取您的网站。

我们要先安装一下，打开你的hexo博客根目录，分别用下面两个命令来安装针对谷歌和百度的插件

``` bash
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save

```
> 如果在你的博客根目录的public下面发现生成了sitemap.xml以及baidusitemap.xml就表示成功了。

### 提交sitemap到谷歌站长工具

![chrome站点地图](/blogs/images/hexo-chrome-2.png)

> 可以看到所有页面的索引都添加到了搜索引擎  