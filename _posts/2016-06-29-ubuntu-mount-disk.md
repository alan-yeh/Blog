---
layout: post
date: 2016-06-29
title: Ubuntu自动挂载硬盘分区
tags: Linux
---
　　最近公司增配了一个固态硬盘，然后将老硬盘换下，固态作为系统分区。老硬盘2T不能浪费，作为备份盘。但是每次Ubuntu开机，都要手动挂载一下硬盘，比较烦。于是折腾一下，让系统开机之后自动挂载硬盘。

　　查看硬盘挂载信息。

```bash
$ sudo blkid

/dev/sda1: LABEL="Data" UUID="2662a892-8f1d-42cc-b474-a22aa643b9e3" TYPE="ext4" PARTUUID="ca795365-9cc0-4b41-a5f4-7f88256cc3df"
/dev/sdb1: UUID="B138-0525" TYPE="vfat" PARTUUID="051aab1d-d4a1-4649-9982-7b9e75702a4e"
/dev/sdb2: UUID="87e2afb3-33dc-46a7-b407-92eed240d037" TYPE="ext4" PARTUUID="93b5108a-f1df-40ed-97b0-371615ec57b2"
/dev/sdb3: UUID="09b7568a-3d1a-4c22-a9ca-04e75a5542c4" TYPE="swap" PARTUUID="87f8393c-4cd8-49ba-92fd-7773b1808740"
```

　　修改分区挂载配置表。

```bash
$ sudo nano /etc/fstab
```

　　配置文件包含以下几项信息：

- `file system`: 分区定位，可以给磁盘号、UUID或Label，在`blkid`里，我们可以获得它的UUID或LABEL(如果存在)。
- `mount point`: 挂载点。例如可以挂载在`/backup`，那么以后就可以使用`/backup`来访问硬盘。
- `type`: 挂载磁盘类型。在`blkid`中可以找到TYPE。
- `options`: 挂载参数。
- `dump`: 磁盘备份，默认为0，不备份。
- `pass`: 磁盘检查，默认为0，不检查。

　　我现在想要将*/dev/sda1*挂载到*/backup*目录下，那么根据`blkid`给出的信息，那么可以像最后一行这样写：

```bash
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during installation
UUID=87e2afb3-33dc-46a7-b407-92eed240d037 /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda1 during installation
UUID=B138-0525  /boot/efi       vfat    umask=0077      0       1
# swap was on /dev/sda3 during installation
UUID=09b7568a-3d1a-4c22-a9ca-04e75a5542c4 none            swap    sw              0       0

# backup
UUID=2662a892-8f1d-42cc-b474-a22aa643b9e3 /backup ext4 errors=remount-ro 0 0
```
　　检查并挂载新添项。

```bash
$ sudo mount -a
```
　　`mount -a`会将*/etc/fstab*中的项全部挂载，如果有错，则会提示错误。然后根据错误找出原因修改。

　　最后，重启试试效果。