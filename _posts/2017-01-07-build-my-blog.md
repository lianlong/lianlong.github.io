---
layout: post
title:  "建立自己博客的一点感想"
categories: Git jeklly
tags:  博客 检索 
author: lianlong
---

* content
{:toc}

从1月5号开始决定搭建自己的博客开始，经过三天，终于从一个连一句html语言不懂的小白开始，逐步了解了一些html+css+javascript，站长工具使用，百度分析和谷歌分析，CDN加速技术等一些列使用并为自己申请了一个独立域名[http://www.lianlong.website](src=http://lianlong.website)




## github page搭建问题

github搭建过程中问题比较少，网上资料比较多，主要是对于自己新建的项目要采用用户名.github.io的形式，然后套用模板，提交即可，由于在window下使用，所以git可能会出现问题，在[博客](http://lianlong.website/2017/01/05/git-window-chinese/)中有详细的解释。

需要注意的是在绑定自己域名时对于新申请的域名，确保设置好了域名解析服务器，并能够对域名进行有效的解析。

## 搜索引擎检索

对于google搜索引擎，设置检索非常简单，提交测试可以发现已经通过了。

对于百度检索，由于github屏蔽了百度爬虫的IP，所以直接用百度爬虫爬网站的时候拒绝访问，但是显示移动端是可以爬虫到的。对于PC端，由于自己博客量不够大，所以采用了手动加入的方式，采用sitemap也显示拒绝访问。

网上说可以采用CDN加速，进行访问，所以采用了百度云CDN服务，但是通过站点分析可以看到由于自己的网页没有再国内备案，所以采用百度云加速的方式基本在国外节点有数据缓存，在国内节点上没有数据缓存。

另外对于百度云CDN加速，其中有一项锁定的设置对于搜索引擎采用回源方式进行抓取，这样的话会导致百度搜索引擎直接访问github，所以依然拒绝访问。

现在改为加速乐进行加速，正在测试结果，域名解析暂时还没有完成，等待结果，网上说的加速乐缓存不自动更新的问题，现在随着加速乐的升级也可以定时回源了，不存在不可以回源的问题了。

## 站长工具

同时注册了百度站长工具和google站长工具，google站长工具使用比较简单，百度的有些复杂，而且百度的检索比较慢，到目前为止，暂时还没有检索到我的网站，google已经检索到了。测试是否检索可以采用site:lianlong.website的方式进行测试。

## 域名备案

自己使用的website域名暂时无法备案！！！！许多国内的服务都需要备案才能够使用。