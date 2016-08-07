---
layout: post
date: 2016-04-25
title: Ubuntu安装中文输入法
tags: Linux
---
　　最近准备折腾Gitlab，需要安装一个Linux环境。大学的时候，装过Ubunut，感觉还不错。刚好[Ubuntu 16.04 LTS](http://www.ubuntu.com/download/desktop)发布了，同一时间，[Ubuntu Kylin](http://www.ubuntukylin.com)也同一时间更新了。但是新版的Ubuntu Kylin长相实在太不讨人喜欢了，后面就干脆直接下载安装原版英文版Ubuntu了。

　　安装完后，接着就去安装中文输入法。由于对Ubuntu完全没有经验，完全不懂怎么安装输入法。看网上说Ubuntu自带的ibus输入引擎不合适中国人使用，而且中文输入法大多数是使用fcitx引擎，安装输入法前，需要先安装fcitx。然后就搜索如何安装fcitx，结果折腾了一个下午没搞定。最后发现一个很简单的安装输入法的技巧，特此记录下来。

　　桌面右上角，打开`System Settings`，打开`Language Support`。

![](/assets/blog/ubuntu-setup-input-method/system-setting-language-support.png)

　　接着系统会提示你`The language support is not installed comletely`，选择Install。输入系统管理员密码之后，系统就会开始Applying changes（其实就是下载语言包等）。

![](/assets/blog/ubuntu-setup-input-method/language-support-tips.png)

　　等安装完了之后，不要做任何动作，直接左上角关闭Language Support选项卡。然后重新打开Language Support，就会发现非常神奇的，fcitx已经安装上了！然后把`Keyboard input method system`选上fcitx，右下角，Close。

![](/assets/blog/ubuntu-setup-input-method/language-support-finished.png)

　　重启系统。回到`System Settings`，选择`Text Entry`。

![](/assets/blog/ubuntu-setup-input-method/system-setting-text-entry.png)

　　搜索`Fcitx`，可以看到系统已经为你安装了一系列输入法了，选择其中一个，右下角Add就可以完成中文输入法的安装了。

![](/assets/blog/ubuntu-setup-input-method/text-entry-choose-an-input-source.png)

查看桌面右上角小企鹅，可以在下拉菜单里找到刚才选择的输入法了。

![](/assets/blog/ubuntu-setup-input-method/setup-completed.png)

