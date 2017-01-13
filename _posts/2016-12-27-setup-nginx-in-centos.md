---
layout: post
date: 2016-12-27
title: CentOS安装Nginx
tags: Linux
---

## 背景
　　Ubuntu安装Nginx是比较简单的，使用`sudo apt-get install nginx`就可以了，但是公司经常使用CentOS，就不是特别会配置了。还是记录一下。


## 安装依赖项

```
yum install -y pcre pcre-devel  
yum install -y zlib zlib-devel  
yum install -y openssl openssl-devel  
```

## 下载Nginx
　　下载Nginx。

```
http://nginx.org/en/download.html
```
　　解压后，终端进到目录里。

```
cd nginx-1.7.3
./configure
sudo make && sudo make install
```
　　默认安装路径为*/usr/local/nginx*，修改`/usr/local/nginx/conf/nginx.conf`文件后，跳转到*/usr/local/nginx/sbin*，就可以用nginx启动或者停止了。
　　


