---
layout: post
title:  "Github page同时部署到github和code.net远程仓库中"
categories: Git 远程仓库 githubpages
tags:  博客 远程仓库 
author: lianlong
---

* content
{:toc}

在搭建自己的独立博客是，遇到问题，github pages无法被百度检索，所以采用了github 和code.net同时部署的方法，code net博客配置与github的基本相同，可以参考相关文献




## github上下载完整的代码

```javascript
	git clone git@github.com:lianlong/lianlong.github.io.git
```

下载好代码后，在.git/configure中加入code上托管的仓库链接 <br>

```javascript
	[remote "origin"]
		url = git@github.com:lianlong/lianlong.github.io.git
		url = git@git.coding.net:anlongli/anlongli.git
```

然后正常修改，在push时，由于code仓库中没有更新，所采用强制push的模式

```javascript
	git push -f origin master
```

