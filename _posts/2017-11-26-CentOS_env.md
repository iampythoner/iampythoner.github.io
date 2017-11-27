---
layout: cnblog_post
title:  "CentOS env"
date:   2017-11-26 06:34:39
categories: Linux
---

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

----安装zlib即必要的库
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
----方法1
cd  /etc/ld.so.conf.d
vim usr_local_lib.conf
/usr/local/lib
ldconfig
---方法2
logout再重新登录

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