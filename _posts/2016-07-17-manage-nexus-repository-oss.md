---
layout: post
date: 2016-07-17
title: 管理Nexus Repository OSS
tags: Java-Web
---

# 目录
- [支持域名访问](#support-domain)

## <a id="support-domain"></a>支持域名访问
　　Nexus默认使用8081端口，总是输入IP+端口，略麻烦。因此折腾一下，使用域名来访问Nexus。

### 修改Nexus配置
　　Nexus默认`Context Path`是*/nexus*。如果继续使用这个路径的话，使用域名访问会没办法登录。因此需要先修改Nexus的配置。

*/opt/nexus/nexus-2.13.0-01/conf/nexus.properties*

```
# nexus-webapp-context-path=/nexus
# 将context path修改为/
nexus-webapp-context-path=/
```

　　重启nexus

```bash
$ ./nexus stopPassword: ****************************************WARNING - NOT RECOMMENDED TO RUN AS ROOT****************************************Stopping Nexus OSS...Stopped Nexus OSS.
$ ./nexus startPassword: ****************************************WARNING - NOT RECOMMENDED TO RUN AS ROOT****************************************Starting Nexus OSS...Started Nexus OSS.
```

### 安装Nginx
　　打开Terminal，安装Nginx。

```bash
$ sudo apt-get install nginx
```
　　浏览器打开[http://localhost](http://localhost)，看看是否安装成功。

![](/assets/blog/manage-nexus-repository-oss/setup-nginx.png)

　　修改Nginx配置文件，在最后面添加以下内容。

*/etc/nginx/sites-available/default*

```
server {
	listen 80;
	server_name nexus.postsync.cn; #修改成你的域名

	location / {
		 proxy_pass  http://localhost:8081;
                 proxy_redirect  off; 
                 proxy_set_header Host $host; 
                 proxy_set_header X-Real-IP $remote_addr; 
                 proxy_set_header X-Forwarded-For 
                 $proxy_add_x_forwarded_for; 
	}
}
```

　　重新加载nginx。

```bash
$ sudo nginx -s reload
```

![](/assets/blog/manage-nexus-repository-oss/setup-domain.png)


