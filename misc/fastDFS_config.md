---
permalink: /misc/fastDFS_cnfig
layout: cnblog_post
title:  'fastDFS配置'
date:   2017-12-06 06:34:39
categories: misc
---

```
cd ~/Document/lib
mkdir fastdfs && cd fastdfs
git clone https://github.com/happyfish100/libfastcommon
git clone https://github.com/happyfish100/fastdfs
cd libfastcommon
./make.sh
sudo ./make.sh install

cd ../fastdfs
./make.sh
sudo ./make.sh install

mkdir -p ~/fastdfs/tracker
mkdir -p ~/fastdfs/storage

cd /etc/fdfs
sudo cp tracker.conf.sample tracker.conf
sudo cp tracker.conf.sample

sudo vim tracker.conf
#base_path=/home/mike/fastdfs/tracker

sudo vim storage.conf
#base_path=/home/mike/fastdfs/storage
#store_path0=/home/mik3/fastdfs/storage
#tracker_server=ip地址:22122

#sudo service fdfs_trackerd start
sudo /etc/init.d/fdfs_trackerd start
#sudo service fdfs_storaged start
sudo /etc/init.d/fdfs_storaged start
```