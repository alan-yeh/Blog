---
layout: post
date: 2016-06-26
title: Mac OS X下Gradle的安装与配置
tags: Java-Web
---

## 下载Gradle
到[Gradle官网](http://gradle.org/download)下载安装包，选择下载`Binary only distribution`。下载完了之后，解压到一个目录，比如`Users/yerl/gradle`。

## 设置环境变量
打开终端，输入以下命令，编辑bash_profile

```bash
$ nano ~/.bash_profile
```
添加以下代码在最后，保存并退出：

```bash
# gradle
export GRADLE_HOME=/Users/yerl/gradle 
export PATH=$PATH:$GRADLE_HOME/bin
```

输入命令使`bash_profile`生效

```bash
$ source ~/.bash_profile
```

## 查看是否安装成功

```bash
$ gradle -version
```
如果输出以下信息，说明gradle安装成功了

```bash
$ gradle -version

------------------------------------------------------------
Gradle 2.13
------------------------------------------------------------

Build time:   2016-04-25 04:10:10 UTC
Build number: none
Revision:     3b427b1481e46232107303c90be7b05079b05b1c

Groovy:       2.4.4
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_45 (Oracle Corporation 25.45-b02)
OS:           Mac OS X 10.11.5 x86_64
```