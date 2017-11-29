---
layout: post
date: 2017-01-13
title: 申请并使用HTTPS
tags: Java-Web
---

## Https证书申请
　　阿里云提供免费的证书申请服务，过程比较简单，记录一下。

　　打开[阿里云首页](https://wanwang.aliyun.com)。首先注册并登录。

　　打开`产品`->`安全`->`CA证书服务`。
![2017-01-13-ali-index](/assets/blog/2017-01-13-ali-index.png)

　　点击`立即购买`。
![2017-01-13-shop-index](/assets/blog/2017-01-13-shop-index.png)

　　选择`免费型DV SSL`，然后`立即购买`。
![2017-01-13-pay](/assets/blog/2017-01-13-pay.png)

　　支付成功后，进入`证书控制台`
![2017-01-13-pay-success](/assets/blog/2017-01-13-pay-success.png)

　　打开红框标注的`证书服务`
![7F0EA76B-6BCF-42CF-B975-BDD3EB28EEE4](/assets/blog/7F0EA76B-6BCF-42CF-B975-BDD3EB28EEE4.png)

　　点击`补全`
![2017-01-13-cert-list](/assets/blog/2017-01-13-cert-list.png)

　　在框框中填写你要申请的域名地址，然后下一步。
![2017-01-13-input-domain](/assets/blog/2017-01-13-input-domain.png)

　　填写一些信息，下一步
　　
![2017-01-13-input-infomation](/assets/blog/2017-01-13-input-infomation.png)

　　选择`系统生成CSR`，点击`创建`。创建成功之后，提交审核。

![2017-01-13-gen-cs](/assets/blog/2017-01-13-gen-csr.png)

　　之后注意查收邮件，会有进度提醒。

![2017-01-13-emai](/assets/blog/2017-01-13-email.png)
　　
　　需要去域名管理中心进行域名认证。去域名解析中心添加一个主机记录。按邮件上面提供的主机记录填写即可。
　　
![2017-01-13-CNAME](/assets/blog/2017-01-13-CNAME.png)

　　稍等数分钟，刷新证书列表，等待证书状态变更。签发证书成功。

![2017-01-13-success](/assets/blog/2017-01-13-success.png)

　　点击下载，就可以下载对应的服务器所需的证书。

![2017-01-13-download](/assets/blog/2017-01-13-download.png)

## 使用Https证书
　　点击`下载证书for Nginx`，得到一个压缩包。解压后如下：

![2017-01-13-nginx-certs](/assets/blog/2017-01-13-nginx-certs.png)

　　将这两个证书文件拷贝到nginx目录下的`conf`目录下。然后编辑`nginx.conf`

```
    # 强制使用https
    server{        listen 80;        server_name ssl.yerl.cn;
        rewrite ^(.*)$  https://$host$1 permanent;    }
    # SSL配置
    server{        listen 443;        server_name ssl.yerl.cn;        ssl on;        ssl_certificate 213998963030095.pem;        ssl_certificate_key 213998963030095.key;        location /{
            # 转发到本地的Tomcat服务器            proxy_pass http://127.0.0.1:8080;            proxy_redirect default;            client_max_body_size 100m;            #root html;            #index oa.html;            proxy_set_header Host $http_host;            proxy_set_header X-Real-IP $remote_addr;            proxy_set_header X-Scheme $scheme;        }    }
```
　　启动nginx之后，就可以使用`https://ssl.yerl.cn`访问服务器了。



