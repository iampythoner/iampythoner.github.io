---
layout: cnblog_post
title: macOS-Big-Sur
permalink: '/misc/macOS-Big-Sur'
date: 2021-12-19 14:21:39
categories: misc
---


```
diskutil list
sudo mount -o nobrowse -t apfs /dev/disk1s6 ${HOME}/mount
sudo kmutil install --volume-root ${HOME}/mount --update-all
sudo bless --folder ${HOME}/mount/System/Library/CoreServices --bootefi --create-snapshot
```