---
layout: cnblog_post
title:  "CentOS Ubuntu env"
date:   2017-11-26 06:34:39
categories: Linux
---

## CentOS

```sh
----打开网卡
# 不同的系统版本或者电脑这个会不同，不一定都是ens33
vi /etc/sysconfig/network-scripts/ifcfg-ens33
# 将ONBOOT更改为yes
ONBOOT=yes

----查看ip,直接使用SSH进行剩下操作
ip address

----时间校准
# 安装ntp
sudo yum install ntp
# 设置时区，编辑下面文件
vi /etc/sysconfig/clock
ZONE="Asia/Shanghai"
UTC=false
ARC=false
# 放到时区信息到系统目录
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 对准时间
ntpdate asia.pool.ntp.org
# 设置硬件时间和系统时间一致并校准
/sbin/hwclock --systohc

----安装git
sudo yum install git

----安装zlib
yum install zlib zlib-devel -y

----安装openssl
yum install openssl-devel -y

----编译安装Python3
# 安装gcc编译器
sudo yum install gcc
# 安装依赖库zlib
yum install zlib zlib-devel -y
# 下载Python3.6.3源码并解压
cd ~/Documents/lib
wget -nd https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz
tar -zxvf Python-3.6.3.tgz
# cd Python3.6.3 开始编译安装
./configure --enable-optimizations
# 编译安装
make && make install
# 检查一下安装是否成功
python3 -V
pip3 -V

## 如果出现这个错误
python3: error while loading shared libraries: libpython3.6m.so.1.0: cannot open shared object file: No such file or directory
# 这样解决
cd  /etc/ld.so.conf.d
vim usr_local_lib.conf
/usr/local/lib
ldconfig

----安装最新vim的准备
# 移除现有的vim
yum remove vim -y
# 安装编译依赖
yum install ncurses-devel -y
# 如果失败，提示unknown host apt.sw.be，手工安装：
# wget http://mirror.centos.org/centos/7/os/x86_64/Packages/ncurses-devel-5.9-13.20130511.el7.x86_64.rpm
# yum install ncurses-devel-5.9-13.20130511.el7.x86_64.rpm

----编译最新版vim
# 如果下载的是zip
# unzip -q vim.zip -d vim
# cd vim/vim-master/src
# 如果用的git下载
git clone https://github.com/vim/vim.git
cd vim
# 配置
./configure --enable-multibyte --enable-pythoninterp=yes --enable-python3interp=yes
# 编译
make
# 安装
make install
# 退出登录
logout

----配置Vundle
yum update
yum upgrade vim # 更新vim
# clone vundle
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
# 配置 vimrc（提供文件.vimrc 和 color主题文件夹）
vim ~/.vimrc
# 安装插件，进入vim之后
:PluginInstall

----配置YouCompleteMe
cd ~/.vim/bundle
rm -rf YouCompleteMe
git clone https://github.com/Valloric/YouCompleteMe
cd YouCompleteMe
git submodule update --init --recursive
# 不要到clang官网下载clang，也不要自己安装cmake，用下面的命令一次安装gcc-c++编译器、cmake等
yum install automake gcc gcc-c++ kernel-devel cmake
python3 install.py --clang-completer

----mysql安装之前卸载mariadb
rpm -qa | grep mariadb
# 将上面输出内容依次放到下面一行命令的参数中(替换xxxxxx)
rpm -e --nodeps xxxxxx

----mysql安装
# 下载地址 https://dev.mysql.com/downloads/mysql/
# 顺序一个都不能错
wget -nd https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.20-1.el7.x86_64.rpm-bundle.tar
mkdir mysql-rmp-dir
tar -xf mysql-5.7.20-1.el7.x86_64.rpm-bundle.tar -C mysql-rmp-dir
yum install net-tools
rpm -ivh mysql-community-common-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.20-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.20-1.el7.x86_64.rpm


--------mysql初始密码问题
----CentOS
cat /var/log/mysqld.log | grep root@localhost
### 输出 [Note] A temporary password is generated for root@localhost: XXru8DV%2hJW
mysqladmin -uroot -p password 123Abcd-
输入查到的密码

或者进入数据库再改
mysql -uroot -p
XXru8DV%2hJW
>ALTER USER USER() IDENTIFIED BY 'Abcd-123';

----Ubuntu
sudo cat /etc/mysql/debian.cnf
进入之后修改root密码
SET PASSWORD FOR 'root'@'localhost' = PASSWORD(新密码)

----不推荐的方式
cd /usr/local/mysql/bin/
sudo su
./mysqld_safe –skip-grant-tables & ./mysql
FLUSH PRIVILEGES;
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('你的新密码');

```

## Ubuntu

```sh
----安装curl
# 这个命令一般装机就有，但是ubuntu 17.10 没有
sudo apt install curl

----安装net-tools
# 如果没有ifconfig，ubuntu 17.10 没有
sudo apt install net-tools

----安装git
sudo apt install git

----安装vim
sudo apt install vim

----配置Vundle
sudo apt-get update # 更新apt
sudo apt-get upgrade vim # 更新vim
# clone vundle
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
# 配置 vimrc（提供文件.vimrc 和 color主题文件夹）
vim ~/.vimrc
# 安装插件，进入vim之后
:PluginInstall

----配置YouCompleteMe
# 如果PluginInstall不能成功创建YouCompleteMe代码目录，cd到~/.vim/bundle执行git clone https://github.com/Valloric/YouCompleteMe手动下载代码
cd ~/.vim/bundle/YouCompleteMe
git submodule update --init --recursive
# 安装Cmake 因为./install.py --clang-completer需要cmake, 可以按照YouCompleteMe的推荐方式：https://github.com/Valloric/YouCompleteMe
# sudo apt install build-essential cmake
# 也可以在官网上直接下载https://cmake.org/download/  推荐使用这种方式，不必下载源码版，下载可执行版，然后做软链接即可
wget -nd -P ~/Documents/lib https://cmake.org/files/v3.10/cmake-3.10.0-Linux-x86_64.tar.gz
cd ~/Documents/lib
tar -zxvf cmake-3.10.0-Linux-x86_64.tar.gz
sudo ln -s ~/Documents/lib/cmake-3.10.0-Linux-x86_64/bin/cmake /usr/local/bin
# 回到~/.vim/bundle/YouCompleteMe目录编译安装
python3 install.py --clang-completer

----安装pip3
sudo apt install python3-pip

----安装virtualenv、virtualenvwrapper
sudo pip3 install virtualenv
sudo pip3 install virtualenvwrapper
配置virtualenvwrapper环境, http://iampythoner.com/2017/10/virtualenv/#Linux_env

----安装mysql
sudo apt install mysql-server
设置mysql禁止开机自动启动

----修改mysql初始密码
sudo cat /etc/mysql/debian.cnf
set password for 'root'@'localhost' = password('123456');

----安装ipython3
sudo pip3 install ipython

----安装oh-my-zsh,首先要安装zsh
# 安装zsh
sudo apt install zsh 
# 查看是否安装成功
which zsh
# 设置zsh为当前用户的默认shell
sudo chsh -s $(which zsh)
此时，注销一下用户，重新登陆，打开终端，发现已经使用zsh了
#然后安装oh-my-zsh
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
# 此时bash设置的环境都无效了，在~/.zshrc第一行添加下面语句，让之前bash设置的环境先生效，然后再加载zsh的环境。在添加之前可以将.bashrc备份，并将含有shopt命令的语句注释，将bash补全的语句注释
source ~/.bashrc
# 添加上面一句之后保存退出，然后让新的zsh配置生效
source ~/.zshrc

----SSH基本配置
# 安装ssh-server,安装完成之后服务自动开启
sudo apt install openssh-server
# 关闭服务
sudo /etc/init.d/ssh stop
# 开启服务
sudo /etc/init.d/ssh start
# 查看服务状态
sudo /etc/init.d/ssh status

----安装uwsgi
# 官网http://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html
# 建议自己编译
cd ~/Document/lib
# 下载
wget https://projects.unbit.it/downloads/uwsgi-latest.tar.gz
# 解压
tar -zxvf uwsgi-latest.tar.gz 
# 进入解压后的目录，当前是2.0.15
cd uwsgi-2.0.15/
# 编译安装，不要使用make
sudo python3 setup.py install
# 检查是否已经安装，安装完成之后当前目录有一个uwsgi可执行程序，同时/usr/local/bin 下面也有拷贝
which uwsgi # 输出/usr/local/bin/uwsgi

----安装nginx
sudo apt install nginx

-----node.js环境安装
# 最好安装绿色版，node更新太频繁，可以将不同的版本放在同一目录下，使用软链接切换
# 将8.9.1下载到~/Portable目录下
wget -nd -P ~/PortableApp/ https://nodejs.org/dist/v8.9.1/node-v8.9.1-linux-x64.tar.xz
# 解压
cd ~/PortableApp
tar -Jxvf node-v8.9.1-linux-x64.tar.xz
# 添加软链接
ln -s ~/PortableApp/node-v8.9.1-linux-x64 ~/PortableApp/node_current
# 在.bashrc配置node环境
vim ~/.bashrc
# 在.bashrc中添加
export NODE_PATH=~/PortableApp/node_current
export PATH=$NODE_PATH/bin:$PATH
# 让bash生效
source ~/.bashrc # 如果使用的是zsh，则source ~/.zshrc
# 检查node是否可以使用
node -v
下次使用其他版本的node，只需要将current_node 链接到新的版本即可

-----后续还要添加的
supervisor安装及配置
redis基本配置
mongo基本配置

-------------------GUI Application-----------------------
Ctrl + H # 桌面版显示隐藏文件/夹

----安装VSCode
microsoft官网 https://code.visualstudio.com/
安装python插件 https://marketplace.visualstudio.com/items?itemName=ms-python.python

----安装pycharm
只需解压绿色版即可 打开bin/pycharm.sh
```