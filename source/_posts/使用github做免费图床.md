---
title: 使用github做免费图床

date: 2016-10-13 14:37:05

tags: linux

categories: 
- linux

---

## 使用免费图床  

这几天刚刚把hexo 搭起来
写了一篇[教程](http://localhost:5000/2016/10/12/Hexo-%E6%90%AD%E5%BB%BA%E5%92%8Cgit-Hook/)
还把之前wordpress里的文章和印象笔记一起导出成html 放在了
>[alanban.github.io](https://alanban.github.io/)


在使用hexo的过程中看到很多人在用七牛图床
我想要免费的空间啊...

然后突然想起了github

那就用github 

<!-- more -->
以下图片都是使用github 仓库来当图床
假如你看不到图片 
应该github被墙啦

### 新建仓库

新建一个叫 img 的仓库

![创建仓库](https://raw.githubusercontent.com/alanban/img/master/%E5%9B%BE%E5%BA%8A1.png)  

里面我已经上传了两张图片  

![图床2](https://raw.githubusercontent.com/alanban/img/master/%E5%9B%BE%E5%BA%8A2.png)

具体如何上传 ↓ 

![上传](https://raw.githubusercontent.com/alanban/img/master/%E4%B8%8A%E4%BC%A0%E5%9B%BE%E5%BA%8A.png)


### 使用
图片地址是这样的：  
	https://raw.githubusercontent.com/github账号名称/仓库名称/master/图片名称.png  
例如：  
	https://raw.githubusercontent.com/alanban/img/master/%E5%9B%BE%E5%BA%8A2.png
