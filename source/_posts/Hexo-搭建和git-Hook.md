---
title: 在VPS上搭建Hexo静态博客和实现更新

date: 2016-10-12 10:13:35

tags:
- linux
- git
- node.js
- nginx
- hexo


categories: 
- linux

---

### 为什么选择hexo ###
之前在阿里云用学生优惠租了个一年的服务器，体验不好，WordPress也常常cpu跑满，感觉就跟被DDOS，WordPress相对hexo太臃肿了，最主要的我不想备案...

现在看到的这个网站是搭建在搬瓦工VPS上的，120多一年，单核20G硬盘1G内存2000G流量每月，够用！便宜！

搬瓦工特点：

* 一键shadowsocks服务器搭建
* 无需备案
* 随意更换系统OS
* 便宜

>搬瓦工地址：[https://bwh1.net/](https://bwh1.net/)

### 捋一捋

+ 搭建shadowsocks
+ 搭建hexo博客环境
+ 配置git实现更新博客
<!-- more -->
#### hexo

一套运行在node.js上的程序，生成静态HTML，拥有大量的主题可供选择

#### nginx

HTTP反向代理，在高并发下也拥有强大能力应对，nginx可作为负载均衡器

#### git

分布式版本控制系统，相比SVN，拥有更多高级特性，使得协同更加简单


----------
### 手把手实现

#### 本机环境安装

1. 安装 node  
	>[node官网](https://nodejs.org/en/)
	>安装包已安装npm


2. 安装 git

	>[git官网](https://git-scm.com/downloads)

	打开git Bash
	设置用户名
	<pre><code>git config --global user.email "email@example.com"
	git config --global user.name "username"</code></pre>
	生成ssh秘钥：
	`ssh-keygen -t rsa -C "exsample@google.com"`  
	一路回车 完成后 打开 `C:\Users\<用户名>\.ssh` 文件下会有两个文件  
	`id_rsa`  
	`id_rsa.pub` ←这个文件稍后使用


	
3. 安装 hexo

	>[hexo官网](https://hexo.io)

	选择一个文件夹如： F:\hexo  
	右键文件夹打开git Bash  
	输入  
	`npm install hexo-cli -g`  
	`hexo init blog`  
	`cd blog`
	`npm install`  
	`hexo s -p 5000`  
	↑启动http服务指定端口5000  
	这时打开 [http://localhost:5000/](http://localhost:5000/)就可以看到你的Hexo博客了
#### 服务器环境安装与配置
1. 安装 Nginx：  
	`yum install nginx -y `
2. 安装 Git：  
	`yum install git-core`
3. 未来安全添加一个新的账户
	<pre>
	adduser git
	chmod 740 /etc/sudoers
	vim /etc/sudoers
	</pre>
	找到下面这句 
	<pre><code>## Allow root to run any commands anywhere
	root    ALL=(ALL)     ALL</code></pre>
	在下面添加一句
	<pre><code>git   ALL=(ALL)   ALL</code></pre>
	Esc->:wq ->回车 保存  
	然后设置权限输入  
	<pre>chmod 440 /etc/sudoers</pre>
	设置 git 账户的密码
	<pre>passwd git</pre>
	查看 git 账户信息
	<pre>finger git</pre>
4. 创建远程仓库
	<pre>
	su git
	cd ~
	mkdir .ssh
	cd .ssh
	touch authorized_keys
	vi authorized_keys	
	</pre>
	在vim中粘贴入`id_rsa.pub`文件中从AAA到最后的字符串  
	保存退出
	<pre>
	cd ~
	mkdir hexo.git   #这就是远程仓库名
	cd hexo.git
	git init --bare   #初始化为裸仓库
	</pre>
5. 创建网站根目录：
	选择一个目录作为你的网站根目录 例如： /var/www/hexo
	<pre>
	mkdir /var/www/hexo
	cd /var/www/hexo
	chown git:git -R /var/www/hexo  #指定文件的拥有者改为指定的用户 -R： 目录下所有文件
	</pre>
	
6. 创建 git hook

	当服务器上的远程仓库收到了新的pull时，我们应该将新的静态页面放到 根目录下 也就是 `/var/www/hexo`
	git 有一个特性，也就是 git hooks
	>[git Hooks](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)  
	
	在服务器端钩子中有一个钩子叫 `post-receive`
	我们可以在这个钩子里面搞事情，处理来自客户端的推送操作时，把东西放到正确的地方
	
	<pre>su git
	cd /home/git/hexo.git/hooks
	vim post-reveive
	</pre>
	输入以下内容然后保存
	<pre>#!/bin/bash
	GIT_REPO=/home/git/hexo.git #git仓库目录
	TMP_GIT_CLONE=/tmp/hexo #缓存地址
	PUBLIC_WWW=/var/www/hexo #网站根目录
	rm -rf ${TMP_GIT_CLONE} #删除缓存地址
	git clone $GIT_REPO $TMP_GIT_CLONE #迁出到缓存地址
	rm -rf ${PUBLIC_WWW}/* #清除 网站根目录
	cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW} #从缓存中复制到网站根目录</pre>
	然后给予脚本执行权限
	<pre>chmod +x post-receive</pre>

	>看到这里 加入对于shell 脚本有兴趣可以学习下 ，有用  
	>[shell script教程](http://www.runoob.com/linux/linux-shell.html)

7. 配置Nginx：

	<pre>cd /etc/nginx/conf.d</pre>
	在这里新建一个配置
	<pre>vim hexo.conf</pre>
	输入以下内容后保存
	<pre>server {
    listen         80 ;
    root /var/www/hexo;//网站目录地址 ，与hook里填写的地址
    server_name example.com www.example.com;//指定IP地址或者域名，多个域名之间用空格分 开
    access_log  /var/log/nginx/hexo_access.log;//请求的Log输出地址
    location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ { 
            root /var/www/hexo;
            access_log   off;
            expires      1d;
    }
    location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
        root /var/www/hexo;
        access_log   off;
        expires      10m;
    }
    location / {
        root /var/www/hexo;
        if (-f $request_filename) {
            rewrite ^/(.*)$  /$1 break;
        }
    }
	}</pre>
	然后输入命令：
	<pre>service nginx restart</pre>

	以上是nginx的配置 
	其中 location 是nginx的 URL地址匹配模块  
	上面写的这些算是标准写法，可以不写，默认的也行  
	location URL用来负载均衡、反向代理等等
	支持正则表达式匹配，也支持条件判断匹配  
	例如  
	<pre>location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ { 
            root /var/www/hexo;
            access_log   off;
            expires      1d;
    }</pre>
	其意义是所有扩展名以.gif、.jpg、.jpeg、.png、.bmp、.swf结尾的静态文件都交给nginx处理，存放目录由 root指定

	又例如：  
	<pre>location ~ .*.php$ {
    index index.php;
    proxy_pass http://localhost:8080;
	}</pre>
	location是对此server下动态网页(php)的过滤处理，也就是将所有以.php为后缀的文件都交给本机的8080端口处理。

	>学习 Nginx 就上 [Nginx中文文档](http://www.nginx.cn/doc/)
	
	
#### 实现 git 上传
##### 配置hexo deploy
好了，差不多了  
服务器那边已经搞完了  
打开本地`hexo/blog`下的`_config.yml`  
在最下面找到  

	## Docs: https://hexo.io/docs/deployment.html
	deploy:

如下添加完整：

	deploy:
      type: git
      message: update
      repo: git@bansen.cc:hexo.git
      branch: master
	
将bansen.cc替换成你的 域名或ip  

打开 git bash

cd F:\hexo\blog
hexo g 
hexo d
输入git账户的密码
理论上已经可以通过ip或域名访问到博客了

### 主题安装
### 上传
### done