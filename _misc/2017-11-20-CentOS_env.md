---
layout: cnblog_post
title:  "CentOS裸机"
permalink: '/misc/CentOS_env'
date:   2017-11-20 06:34:39
categories: misc
---

## CentOS

mini版 centOS 7, <br>
aliyun 不需要打开网卡和时间校准两个步骤

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

----创建新用户
useradd -m mike
passwd mike
vim /etc/sudoers
# 在root    ALL=(ALL)   ALL下面添加一行 mike    ALL=(ALL)   ALL

-----------------------------mike用户进行剩下的操作

----安装git
sudo yum install git

---pyenv 和 pyenv-virtualenv
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
yum install -y git
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc
exec "$SHELL"

git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
exec "$SHELL"

# sohu源最近不能用了 去这里https://www.python.org/ftp/python/ 找对应版本下载 放到~/.pyenv/cache 下

---- 安装开发工具包
yum groupinstall "Development Tools" -y

----安装编译Python必要的包
yum install zlib zlib-devel -y
yum install bzip2-devel -y # WARNING: The Python bz2 extension was not compiled. Missing the bzip2 lib? 
yum install openssl-devel -y
yum install sqlite-devel -y
yum install readline readline-devel readline-static -y
yum install mysql-devel -y


# ModuleNotFoundError: No module named '_ctypes'
# make: *** [install] Error 1
yum install libffi-devel


----------
更新系统Python为Python3.7
①卸掉原来的Python3
②https://askubuntu.com/questions/865554/how-do-i-install-python-3-6-using-apt-get#answer-865569
③安装pip: https://pip.pypa.io/en/stable/installing/
----------




----mongo shell升级
使用云端mongo数据库时，遇到过mongo shell版本过低登录不上的问题，可以参照这个给CentOS配置yum源，升级新版的mongo shell
https://blog.csdn.net/xyang81/article/details/51749989
---mongo权限
mongo服务附带auth参数运行，需要验证登录
可以创建一个admin账户，之后所有的账号都可以由这个账号产生,而不是像大部分的网上的文章说的狗屎一样的做法(关闭auth，创建账户，再开启auth，Shit！！！)
use admin
db.createUser({
    user: "root",
    pwd: "xxx",
    roles: [{
        role: "userAdminAnyDatabase",
        db: "admin"
    }]
})
更多权限控制和管理的内容，查看https://www.cnblogs.com/shiyiwen/p/5552750.html

----安装nginx
http://iampythoner.com/misc/nginx_config

----编译安装redis
cd ~/Documents/lib
wget -nd http://download.redis.io/releases/redis-4.0.4.tar.gz
tar -zxf redis-4.0.4.tar.gz
cd redis-4.0.4/src
make
sudo make PREFIX=/usr/local/redis install
sudo mkdir /usr/local/redis/conf
sudo cp ../redis.conf /usr/local/redis/conf/redis.conf
vim ~/.bash_profile
    export REDIS_HOME=/usr/local/redis
    export PATH=$REDIS_HOME/bin:$PATH
source ~/.zshrc
# 启动redis服务
redis-server /usr/local/redis/conf/redis.conf

----mysql安装之前卸载mariadb
sudo rpm -qa | grep mariadb
# 将上面输出内容依次放到下面一行命令的参数中(替换xxxxxx)
sudo rpm -e --nodeps xxxxxx

----mysql安装
# 下载地址 https://dev.mysql.com/downloads/mysql/
# 顺序一个都不能错
wget -nd https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.20-1.el7.x86_64.rpm-bundle.tar
mkdir mysql-rpm-dir
tar -xf mysql-5.7.20-1.el7.x86_64.rpm-bundle.tar -C mysql-rpm-dir
sudo yum install net-tools
sudo rpm -ivh mysql-community-common-5.7.20-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.20-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.20-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.20-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.20-1.el7.x86_64.rpm


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


-------------------------------------------------------------不再使用
----编译安装Python3
# 安装gcc编译器
sudo yum install gcc
# 安装依赖库zlib
sudo yum install zlib zlib-devel -y
# 下载Python3.6.3源码并解压
cd ~/Documents/lib
wget -nd https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz
tar -zxvf Python-3.6.3.tgz
# cd Python3.6.3 开始编译安装
./configure --enable-optimizations --enable-shared
# 编译安装
sudo make && sudo make install
# 检查一下安装是否成功
python3 -V
pip3 -V

----更新pip3
pip3 install --upgrade pip

## 如果出现这个错误
python3: error while loading shared libraries: libpython3.6m.so.1.0: cannot open shared object file: No such file or directory
# 这样解决
cd  /etc/ld.so.conf.d
vim usr_local_lib.conf
/usr/local/lib
ldconfig

----安装virtualenv、virtualenvwrapper
sudo pip3 install virtualenv
sudo pip3 install virtualenvwrapper
配置virtualenvwrapper环境, http://iampythoner.com/2017/10/virtualenv/#Linux_env

----安装最新vim的准备
# 移除现有的vim
sudo yum remove vim -y
# 安装编译依赖
sudo yum install ncurses-devel -y
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
./configure --enable-multibyte --enable-pythoninterp=yes --enable-python3interp=yes --with-features=huge
# 编译
make CFLASS="-02 -D_FORTIFY_SOURCE=1"
###### 如果遇到if_py_both.h:*: undefined reference to 'PyUnicodeUCS4_AsEncodedString'问题，可以在看一下这个文章：http://vim-dev.vim.narkive.com/3nRfeAJJ/vim-cannot-be-built-in-centos-7-due-to-if-py-both-h-undefined-reference-to-pyunicodeucs4
# 安装
sudo make install

----------现在用spacevim了，就不再配置Vundle和YouCompleteMe了
----配置Vundle
sudo yum update
sudo yum upgrade vim # 更新vim
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
sudo yum install automake gcc gcc-c++ kernel-devel cmake
# Compiling YCM with semantic support for C-family languages
sudo python3 install.py --clang-completer
# 如果要安装所有语言的提示可以使用 ./install.py --all 前提是确保 xbuild, go, tsserver, node, npm, rustc, and cargo tools 都被安装，并且在PATH配置了

```


