---
layout: cnblog_post
title:  "install_ubuntu"
permalink: '/misc/install_ubuntu'
date:   2018-01-21 07:34:39
categories: misc
---


```
0.BIOS设置：
Serial ATA Controller 设置为ACHI，否则固态不可见
BOOT的Mode设置为legacy, 否则U盘启动盘不可见

1.先安装windows
工具 ：PE盘
把固态分割成两个盘，一个NTFS(用于安装windows)，一个为未分配(用于安装Ubuntu)
设置windows盘为活动分区，否则写入安装文件后，重启安装会遇到下面情况：
(如果没有活动分区，windows安装进不去)
no bootable device --insert boot disk
An operating system wasn't found.Try disconnecting any drives that don't ……press Ctrl+Alt+Del to restart

2.安装windwos之后，安装Ubuntu	http://v.youku.com/v_show/id_XNjIxMTQ5Njgw.html?spm=a2hzp.8253876.0.0&f=22015116
http://v.youku.com/v_show/id_XNjIxMTQ5Njgw.html?spm=a2hzp.8253876.0.0&f=22015116制作的U盘启动盘
/ 15G
/boot 1G
/home 30G
swap 10G
/var 5G
安装启动引导器的设备 /boot那个分区，这样安装的ubuntu是进不去的，需要在windows安装easyBCD添加引导

3.windows easyBCD添加引导
类型：GRUB(Lagecy)
名称
驱动器：/boot在的盘
```

