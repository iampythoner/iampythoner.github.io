---
layout: cnblog_post
title:  "ubuntu裸机"
permalink: '/misc/ubuntu_env'
date:   2017-11-20 07:34:39
categories: misc
---

ubuntu 17.10

## ubuntu

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

----更新pip3
pip3 install --upgrade pip

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

----为zsh配置powerline系列主题
# 可以到 https://github.com/robbyrussell/oh-my-zsh/wiki/Themes 和 https://github.com/robbyrussell/oh-my-zsh/wiki/External-themes查看主题和扩展主题
# 下载powerline字体 https://github.com/powerline/fonts
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts
./install.sh
cd ..
rm -rf fonts
# 我比较喜欢Bullet train这个主 https://github.com/caiogondim/bullet-train.zsh
cd /Users/Mike/.oh-my-zsh/themes
wget -nd https://raw.githubusercontent.com/caiogondim/bullet-train.zsh/master/bullet-train.zsh-theme
vim ~/.zshrc
ZSH_THEME="bullet-train"
# 250 197 113


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
http://iampythoner.com/misc/nginx_config

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

----编译安装redis
cd ~/Documents/lib
wget -nd http://download.redis.io/releases/redis-4.0.4.tar.gz
tar -zxf redis-4.0.4.tar.gz
cd redis-4.0.4/src
make
sudo make install

-----后续还要添加的
supervisor安装及配置
mongo基本配置

-------------------GUI Application-----------------------
Ctrl + H # 桌面版显示隐藏文件/夹

----安装VSCode
microsoft官网 https://code.visualstudio.com/
安装python插件 https://marketplace.visualstudio.com/items?itemName=ms-python.python

----安装pycharm
只需解压绿色版即可 打开bin/pycharm.sh

----安装sublime 3143
wget -nd https://download.sublimetext.com/sublime_text_3_build_3143_x64.tar.bz
tar jxf sublime_text_3_build_3143_x64.tar.bz2
sudo mv sublime_text_3 /opt/sublime_text
sudo cp sublime_text.desktop /usr/share/applications/
vim /usr/share/applications/sublime_text.desktop
# 修改Icon为
Icon=/opt/sublime_text/Icon/256x256/sublime-text.png
# 注册码
—– BEGIN LICENSE —–
TwitterInc
200 User License
EA7E-890007
1D77F72E 390CDD93 4DCBA022 FAF60790
61AA12C0 A37081C5 D0316412 4584D136
94D7F7D4 95BC8C1C 527DA828 560BB037
D1EDDD8C AE7B379F 50C9D69D B35179EF
2FE898C4 8E4277A8 555CE714 E1FB0E43
D5D52613 C3D12E98 BC49967F 7652EED2
9D2D2E61 67610860 6D338B72 5CF95C69
E36B85CC 84991F19 7575D828 470A92AB
—— END LICENSE ——
```