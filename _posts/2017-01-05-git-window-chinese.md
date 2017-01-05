---
layout: post
title:  "git window客户端适用汉化问题"
categories: Git
tags:  git 
author: lianlong
---

* content
{:toc}

在搭建自己的独立博客的时候，安装Jekyll，添加模板，git等都没有问题，但是在适用git windows bash客户端的时候出现了问题，无法输入汉字，并且汉字路径显示为乱码，网上搜索了大量资料都没有解决，最后在一个博客上看到，可以在MINGW64的终端总options修改为中文UTF-8即可。
如图,谁知Locale为中文，Character set为UTF-8 然后重启终端即可

![git中文设置](http://oj9jepyz4.bkt.clouddn.com/git%E4%B8%AD%E6%96%87.png)