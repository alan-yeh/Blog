---
layout: post
date: 2016-06-25
title: Mac OS X下Maven的安装与配置
tags: Java-Web
---

- 下载[Maven](https://maven.apache.org/download.cgi)，选择下载`Binary zip archive`。下载完了之后，解压到一个目录，比如`Users/yerl/maven`

- 设置`Maven classpath`。打开终端，输入以下命令，编辑bash_profile

```bash
$ nano ~/.bash_profile
```
添加以下代码在最后，保存并退出：

```bash
# maven
export M2_HOME=/Users/yerl/maven
export PATH=$PATH:$M2_HOME/bin
```

- 输入命令使`bash_profile`生效

```bash
$ source ~/.bash_profile
```

- 查看`Maven`是否安装成功

```bash
$ mvn -v
```
如果输出以下信息，说明maven安装成功了

```bash
$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /Users/yerl/maven
Java version: 1.8.0_45, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.11.5", arch: "x86_64", family: "mac"
```