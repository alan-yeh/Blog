---
layout: post
date: 2017-08-23
title: MacOS更新Ruby环境
tags: Other
---
　　MacOS安装后，自带的ruby环境是2.0.0，有点过时了，有的ruby包需要更高版本的ruby环境。在更新ruby环境的时候，踩了一些坑，记录一下。

## 安装方式
　　我是采用源代码编译方式升级ruby的，升级到最新的稳定版2.4.1。直接在[官网](http://www.ruby-lang.org/en/downloads/)下载即可。下载下来是一个压缩包ruby-2.4.1.tar.gz。解压。

## openssl
　　ruby里面很多组件会用到openssl，MacOS后面会有很多坑，甚至要重新安装ruby才行。因此要求MacOS安装openssl。最简单的安装方式是使用brew。来到[brew官网](https://brew.sh)，使用官方提供的脚本进行安装。

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
　　安装完后，使用brew安装openssl

```bash
brew install openssl
```

## 配置ruby
　　打开终端(Terminal)，进入刚才下载解压后的ruby源码目录(ruby-2.4.1)。

```bash
./configure --with-openssl-dir="$(brew --prefix openssl)"
```

## 安装ruby
　　配置完毕后，使用以下命令安装ruby

```bash
sudo make & sudo make install
```

　　查看当前ruby版本，升级成功。

```bash
ruby -v
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-darwin16]
```


