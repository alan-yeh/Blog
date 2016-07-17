---
layout: post
date: 2016-07-17
title: 搭建Nexus Repository OSS
tags: Java-Web
---
　　前面讲到，[如何将Java库发布到中央库](/blog/publish-library-to-maven-center-repository)。[Sonatype](http://central.sonatype.org)提供开源项目仓库托管服务，但是过程比较复杂，而且有的项目作为企业内部项目，不合适开源发布。但是又想使用maven的话，那怎么解决呢？那么，这篇博文可以为你解决这个问题。搭建Nexus Repository OOS私服。

> 搭建环境为Ubuntu 16.04

## 下载Nexus Repository OOS
　　来到[官方下载地址](http://www.sonatype.com/download-oss-sonatype)。现在Nexus Repository Manager OOS有两个版本，一个是3.xx版，一个是2.xx版。这里选择2.xx版的tar.gz。将下载回来的压缩包放在桌面。

## 配置Java环境
　　Nexus Repository OSS需要依赖Java环境。如果你还没有搭建Java环境，可以在[这里](/blog/java-environment)学习。

## 配置Nexus Repository OOS
　　在*/opt*目录下新建文件夹*nexus*，然后在命令行中进入此文件夹。

```bash
$ cd /opt
$ sudo mkdir nexus
$ cd /opt/nexus
```
　　将桌面的压缩包复制并解压到此目录。

```bash
$ sudo cp ~/Desktop/nexus-2.13.0-01-bundle.tar.gz /opt/nexus
$ sudo tar xvfz nexus-2.13.0-01-bundle.tar.gz
```
　　此时的目录结构如下

```
└── nexus
    └── nexus-2.13.0-01
    └── sonatype-work
    └── nexus-2.13.0-01-bundle.tar.gz
```

　　在Terminal中打开*/opt/nexus/nexus-2.13.0-01/bin/jsw/conf/wrapper.conf*文件

```bash
$ sudo gedit /opt/nexus/nexus-2.13.0-01/bin/jsw/conf/wrapper.conf
```
　　修改`wrapper.java.command`的属性如下。

```
wrapper.java.command=/opt/java/jdk1.8.0_91/bin/java
```

> 根据你的Java环境修改

　　修改*/opt/nexus/nexus-2.13.0-01/bin/nexus*

```bash
$ sudo gedit /opt/nexus/nexus-2.13.0-01/bin/nexus
```
　　将`RUN_AS_USER`属性改为root

```
# NOTE - This will set the user which is used to run the Wrapper as well as#  the JVM and is not useful in situations where a privileged resource or#  port needs to be allocated prior to the user being changed.RUN_AS_USER=root
```

## 启动Nexus Repository OOS
　　Terminal来到*/opt/nexus/nexus-2.13.0-01/bin*目录下。

```bashß
$ cd /opt/nexus/nexus-2.13.0-01/bin
$ ./nexus start
```
　　启动成功了

```
****************************************WARNING - NOT RECOMMENDED TO RUN AS ROOT****************************************Starting Nexus OSS...Started Nexus OSS.
```
　　启动成功后，可以打开[http://localhost:8081/nexus](http://localhost:8081/nexus)，查看是否正常运行。如果打开不了，可以查看*/opt/nexus/nexus-2.13.0-01/logs/wrapper.log*文件。

![](../assets/blog/setup-nexus-repository-oss/setup-success.png)