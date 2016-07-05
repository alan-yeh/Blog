---
layout: post
date: 2016-07-05
title: Linux java环境变量
tags: Linux
---
　　下载jdk，解压到*/usr/local/*目录下。

> 我下载的版本是*jdk1.8.0_91*，所以我的路径是*/usr/local/jdk1.8.0_91*

　　打开*Terminal*，编辑*/etc/profile*文件。

```bash
$ nano /etc/profile
```
　　在最后面添加java环境变量

```bash
export JAVA_HOME=/usr/local/jdk1.8.0_91
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

　　按下`Ctrl + X`，保存后退出。然后在*Terminal*中输入以下命令，应用Java环境。

```bash
$ source /etc/profile
```

　　检测Java环境是否安装正确。

```bash
$ java -version

java version "1.8.0_91"Java(TM) SE Runtime Environment (build 1.8.0_91-b14)Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```