---
permalink: /misc/ubuntu_desktop_app
layout: cnblog_post
title:  'ubuntu桌面程序图标'
date:   2017-10-23 06:34:39
categories: misc
---

以pycharm为例：<br>

```
cd /usr/share/applications
sudo vim pycharm.desktop
```

```
[Desktop Entry]                                                                 
Version=1.0
Type=Application
Name=PyCharm
Icon=/home/dweiw/Documents/PortableApp/pycharm-2017.3/bin/pycharm.png
Exec="/home/dweiw/Documents/PortableApp/pycharm-2017.3/bin/pycharm.sh" %f
Comment=The Drive to Develop
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-pycharm
```