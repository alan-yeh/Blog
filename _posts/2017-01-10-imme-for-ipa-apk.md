---
layout: post
date: 2017-01-10
title: apk和ipa文件的MIME类型
tags: Java-Web
---

## MIME 类型
　　MIME (Multipurpose Internet Mail Extensions) 是描述消息内容类型的因特网标准。

　　MIME 消息能包含文本、图像、音频、视频以及其他应用程序专用的数据。

> 以上摘抄自[W3School](http://www.w3school.com.cn/media/media_mimeref.asp)

## 问题
　　公司有一个app管理平台，用于管理应用的下载、升级等等。

　　但是公司的E人E本的原生浏览器下载apk总是提示【无法打开文件】，但是到QQ官网下载android QQ又没有这个问题。换一个浏览器，比如UC、QQ浏览器，又可以打开并安装。

　　刚开是怀疑是不是E人E本对APP的来源有限制，只能使用https下软件，后面试了一下，又没有问题。

　　后面去抓包看了一下，原来那些能正常下载安装应用的链接的Http Header的Content-Type是`application/vnd.android.package-archive`，而我们的是`application/x-download`。按着改了一下，果然修复了。

　　记录一下：

- apk: application/vnd.android.package-archive
- ipa: application/iphone

