---
layout: post
date: 2016-07-05
title: Java环境变量
tags: Java-Web
---

# Ubuntu
　　下载jdk，解压到*/opt/java/*目录下。

> 我下载的版本是*jdk1.8.0_91*，所以我的路径是*/opt/java/jdk1.8.0_91*

　　打开*Terminal*，编辑*/etc/profile*文件。

```bash
$ nano /etc/profile
```
　　在最后面添加java环境变量

```bash
export JAVA_HOME=/opt/java/jdk1.8.0_91
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

# Windows
　　安装JDK，我的安装路径为*C:\Program Files\Java\jdk1.8.0_92*

　　`我的电脑`-->`属性`-->`高级系统设置`-->`环境变量`。

系统变量中加入以下内容：

```
JAVA_HOME=C:\Program Files\Java\jdk1.8.0_92
Path=;%JAVA_HOME%\bin
CLASSPATH=.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar; 
```