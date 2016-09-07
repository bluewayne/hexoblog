---
title: 基于SSL 的 Node.js 的Nginx配置
permalink: ji-zhu-biao-ti-2
id: 3
updated: '2016-07-04 17:01:14'
date: 2016-06-23 15:46:11
tags:
categories:
---

**转载请注明链接**

**http://www.brusport.com/ji-zhu-biao-ti-2/**


Nginx是一个非常高效的HTTP服务器，同时也是一个非常优秀的反向代理服务器.不想传统的服务器,ngnix遵循事件驱动的异步框架.因此内存占有量底，但是效率非常高.如果你的web 应用是基于Node.js的可以严肃的考虑让Nginx充当反向代理服务器.Ngnix可以非常高效的处理静态资源.对于其他的网络请求它跟Node.js对话然后发送相应给客户端(一般指浏览器).这篇文章我们将讨论怎么用Nginx配置Node.js.of course我们同时会讨论如何在Ngnix服务器中配置SLL.

==安装Ngnix==

先假设你们电脑里面已经安装好Node.js,接下来我们之间讨论如何安装Ngnix.


(1)在MAC安装

如果你的电脑是MAC你可以很方便的用[Homebrew](http://brew.sh/)安装.步骤如下：

Homebrew 需要你的用户名来使用命令chown，命令如下:
sudo chown -R 'username here' /usr/local
[chown的使用说明](http://www.demopu.com/doc/linux/chown.html)

然后用下面两个命令安装Nginx:
brew link pcre
brew install nginx

一旦安装成功，用下面命令启动Ngnix
sudo nginx

Ngnix的配置文件目录在:
/usr/local/etc/nginx/nginx.conf.

<!--more-->

(2)在Ubuntu安装:
如果你使用的Ubuntu环境可以使用下面的命令安装
sudo apt-get update
sudo apt-get install nginx

一旦Ngnix安装成功，它会默认启动

(3)在Windows安装
对于Windows系统,先到Ngnix官网下载页[下载页](http://nginx.org/en/download.html)下载zip压缩包.在dos命令窗口定位到zip的路径，执行下列命令安装Ngnix.

unzip nginx-1.3.13.zip
cd nginx-1.3.13
start nginx

如你看到的，start ngnix就是启动ngnix的命令

现在Ngnix安装任务已经结束，接下来我们来看看如和简单的配置一个服务器.

==安装一个Node.js服务器==
首先,我们先创建一个简单的node.js服务器.你可以通过这个链接https://github.com/jsprodotcom/source/blob/master/nodejs-nginx-ssl-demo-app.zip ,快速下载基于Express的项目.如果你已经下载好这个zip文件后，通过命令行定位到demoApp目录，然后通过下面命令启动这个Express项目(端口默认是3000)

npm install
node bin/www

基于你所在的系统环境，你可以直接打开nginx.conf配置文件.或者用命令
nano /usr/local/etc/nginx/nginx.conf
查了ngnix的配置文件.在配置文件里面你将看到类似的如下命令:
server {
  listen       8080;
  server_name  localhost;
  ....
}
接下来你可以在 http块中的server块配置node.js服务器配置.我们想让Ngnix处理我们网站的静态资源，其他请求则传递给Node.js处理.所以我们配置如下
server {
  listen       8080;#我的个人博客是放在aws的ubuntu环境里，对外开放的端口是80,所以在我的配置文件里面是写80,所以配置因人而异
  server_name  localhost;#如果你用域名，可以用域名代替，比如www.brusport.com
``` bash
  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
  location /public {
    root /usr/local/var/www;
  }
}
```
如你所看到的，上面的nginx配置监听http://localhost:8080. 同时Location/ 块是用来告诉Ngnix如何处理外来的请求.在location块中我们用proxy_pass来指定Node.js来处理请求(在我们例子中就是指http://localhost:3000)

同时我们还需要一个Location /public块来告诉Ngnix如何处理静态资源的请求.在这个location块中我们设置了目录/usr/local/var/www,又来指定静态资源的目录(当然这个目录根据每个人的实际情况而定).所以如果有如下情况http://localhost:8080/public/somepath/file.html 实际上ngnix将会访问文件 /usr/local/var/www/public/somepath/file.html.

如果上面的关于nginx.conf的配置已经配置完了，保持nginx.conf的修改然后又下面命令重新启动nginx.
Mac:
sudo nginx -s stop && sudo nginx

Ubuntu:
sudo service nginx restart
Or
sudo /etc/init.d/nginx restart

Windows:
nginx -s reload


==Setting Up SSL==

搭建一个正式的网站，大多数你需要配置SSL来保护敏感信息.正常我们一般会去一个认证机构同时得到一个颁发的证书.当然你也可以创建一个自己签名的正书.但是在别人的浏览器打开网站会警告“此证书不能被信赖”.但是如果只是出于本地测试，显示是正常的.下面我们讨论如何创建自己的SSL证书.

一旦你已经拥有SSL证书和一个私钥你就可以在Ngnix配置SSL.修改配置如下:
``` bash
server {
 listen       8080;
 listen       443 ssl;
 server_name  localhost;
 ssl_certificate  /etc/nginx/ssl/server.crt
 ssl_certificate_key /etc/nginx/ssl/server.key
 location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
  location /public {
    root /usr/local/var/www;
  }
}
```

如上所示，就是这么简单。现在你访问https://localhost:8080, SSL设置将正常工作./etc/nginx/ssl/server.crt and /etc/nginx/ssl/server.key 分别是你本地存储证书文件和私钥的位置（ 路径根据具体情况而改变）。

refer to https://www.sitepoint.com/configuring-nginx-ssl-node-js/

thx a lot

                                          Bruce liu
                                          2016.7.2