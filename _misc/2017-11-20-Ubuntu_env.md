---
layout: cnblog_post
title:  "ubuntu裸机"
permalink: '/misc/ubuntu_env'
date:   2017-11-20 07:34:39
categories: misc
---

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

----必要的库
sudo apt-get install python-dev
sudo apt-get install zlib1g-dev

----安装pip
# https://pip.pypa.io/en/stable/installing/
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python get-pip.py
----安装ipython2
sudo pip install ipython # 依赖python-dev中的Python.h


----------------pyenv
https://github.com/pyenv/pyenv

git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc
exec "$SHELL"

基本使用：
# 安装指定版本的python
pyenv install 2.7.8
# 例如 v=3.5.2|wget http://mirrors.sohu.com/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v
# build时需要的库：
WARNING: The Python readline extension was not compiled. Missing the GNU readline lib?
ERROR: The Python zlib extension was not compiled. Missing the zlib?
WARNING: The Python bz2 extension was not compiled. Missing the bzip2 lib?
WARNING: The Python sqlite3 extension was not compiled. Missing the SQLite3 lib?
ERROR: The Python ssl extension was not compiled. Missing the OpenSSL lib?
# 解决
sudo apt-get install libreadline6 libreadline6-dev
sudo apt-get install zlib1g-dev
sudo apt-get install libbz2-dev
sudo apt-get install libssl-dev
sudo apt-get install libsqlite3-dev
### Cent OS
yum install readline readline-devel readline-static -y
yum install bzip2-devel bzip2-libs -y
yum install openssl openssl-devel openssl-static -y
yum install sqlite-devel -y
# 查看已经安装的所有的python版本
pyenv versions
# 查看当前使用的环境
pyenv version
# 设置某一个版本为全局的python
pyenv global xxx
------ 配合pyenv-virtualenv
https://github.com/pyenv/pyenv-virtualenv

git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
exec "$SHELL"

--基本使用
#使用当前的pyenv设定的python版本创建虚拟环境
pyenv virtualenv venv34
#列出已经存在的所有的虚拟环境
pyenv virtualenvs
# 激活和退出虚拟环境
pyenv activate
pyenv deactivate
# 删除已经存在的虚拟环境
pyenv uninstall my-virtual-env

----安装最新的vim
# https://blog.csdn.net/gatieme/article/details/52752070
# sudo: add-apt-repository: command not found
# https://www.aliyun.com/jiaocheng/137952.html
sudo apt-get remove vim
sudo add-apt-repository ppa:jonathonf/vim
sudo apt-get update
sudo apt install vim

----安装spacevim
# https://github.com/SpaceVim/SpaceVim
curl -sLf https://spacevim.org/install.sh | bash


----安装pip3
sudo apt install python3-pip

----更新pip3
pip3 install --upgrade pip

----安装ipython3
sudo pip3 install ipython

----安装mysql
sudo apt install mysql-server
设置mysql禁止开机自动启动

----修改mysql初始密码
sudo cat /etc/mysql/debian.cnf
set password for 'root'@'localhost' = password('123456');

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
----使用solarized配色
git clone https://github.com/Anthony25/gnome-terminal-colors-solarized
cd gnome-terminal-colors-solarized
./install.sh


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
sudo make PREFIX=/usr/local/redis install
sudo mkdir /usr/local/redis/conf
sudo cp ../redis.conf /usr/local/redis/conf/redis.conf
vim ~/.bash_profile
    export REDIS_HOME=/usr/local/redis
    export PATH=$REDIS_HOME/bin:$PATH
source ~/.zshrc
# 启动redis服务
redis-server /usr/local/redis/conf/redis.conf

-----redis 开机启动
# https://blog.csdn.net/liulihui1988/article/details/78087495?utm_source=debugrun&utm_medium=referral
cd /usr/local/redis/conf
sudo cp redis.conf 6379.conf
vim 6379.conf
  daemonize yes
cp /.../redis-4.0.4/utils/redis_init_script  /etc/init.d/redis
vim /etc/init.d/redis
  REDISPORT=6379
  # EXEC=/usr/local/bin/redis-server
  EXEC=/usr/local/redis/bin/redis-server
  # CLIEXEC=/usr/local/bin/redis-cli
  CLIEXEC=/usr/local/redis/bin/redis-cli

  PIDFILE=/var/run/redis_${REDISPORT}.pid
  # CONF="/etc/redis/${REDISPORT}.conf"
  CONF="/usr/local/redis/conf/${REDISPORT}.conf"
# can use
sudo service redis start

# https://blog.csdn.net/jlq_diligence/article/details/80680492
sudo apt-get install sysv-rc-conf
sudo sysv-rc-conf keepalived on

sudo sysv-rc-conf redis on # 开机自启动


-----mongo安装和配置
这里使用的是ubuntu14.04 codename：trusty 安装mongo3.0的例子：
1.添加mongo公共GPG密钥
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
2.添加阿里的源
echo "deb http://mirrors.aliyun.com/mongodb/apt/ubuntu/dists/trusty/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
3.重新加载本地安装包列表数据库
sudo apt-get update
4.安装
sudo apt-get install -y mongodb-org

安装完之后，直接以以下配置启动了
/usr/bin/mongod --config /etc/mongod.conf
链接mongo创建root用户
    use admin
    db.createUser({user: 'root', pwd: 'root', roles: [{role: 'root', db: 'admin'}]}
退出修改/etc/mongod.conf添加验证
# 如果是yaml格式的配置
security:
  authorization: enabled
# 其他格式的配置
auth=true

重启mongo
sudo service mongod restart

链接
mongo
    use admin;
    db.auth('root', 'root')

    #为数据库dbaa添加用户mike
    #切到dbaa
    use dbaa
    db.createUser({user: 'mike', pwd: 'mike_pwd', roles: [{role: 'dbAdmin',db: 'dbaa'}, {role: 'readWrite', db: 'dbaa'}]})
    #退出
使用mike链接数据库dbaa
mongo
    use dbaa
    db.auth('mike', 'mike_pwd')
    #输入 show users 提示以下错误
    #E QUERY    Error: not authorized on admin to execute command { usersInfo: 1.0 }
    # 解决方法：使用root登录授予mike权限
    # 切回admin
    use admin
    db.auth('root', 'root')
    # 再切回dbaa
    use dbaa
    # 再授予权限
    db.grantRolesToUser('mike', [{ role: "dbOwner", db: "sandbox"}])
    # 使用mike认证登录
    db.auth('mike', 'mike_qwer')
    # 此时show users可以执行了
    show users

------elasticsearch
sudo tar zxf jdk-8u161-linux-x64.tar.gz -C /usr/local
vim ~/.bashrc
    ### JDK
    export JAVA_HOME=/usr/local/jdk1.8.0_161
    export JRE_HOME=$JAVA_HOME/jre
    export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
    export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
# 验证一下JDK安装
java -version
javac -version

sudo unzip elasticsearch-6.2.3.zip -d /usr/local
vim ~/.bashrc
    ### ES
    export ES_HOME=/usr/local/elasticsearch-6.2.3
    export PATH=$ES_HOME/bin:$PATH
sudo chown mike:mike -R /usr/local/elasticsearch-6.2.3/
# 启动es
elasticsearch
# 测试
curl http://localhost:9200/

--启动遇到问题
[o.e.b.BootstrapChecks    ] [4WpsNl1] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65536] is too low, increase to at least [262144]

解决方法：
sudo vi /etc/sysctl.conf
    # vm.max_map_count=65536
    vm.max_map_count=655360
sysctl -p


--es外网访问
vim config/elasticsearch.yml
    network.host: 0.0.0.0
启动es出现这个错误
...[4WpsNl1] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [1] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

解决方法：
sudo vi /etc/security/limits.conf
    # elasticsearch config start
    * soft nofile 65536
    * hard nofile 131072
    * soft nproc 2048
    * hard nproc 4096
    # elasticsearch config end
然后一定要重启机器，重启重启重启，不重启自杀
# ulimit -Hn 查看max file descriptors限制

然后又遇到这个问题：
...[4WpsNl1] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [1] bootstrap checks failed
[1]: max number of threads [2048] for user [mike] is too low, increase to at least [4096]

解决方法：
sudo vi /etc/security/limits.d/90-nproc.conf
    * soft nproc 4096
再重启机器，重启重启重启，不重启自杀

重新登录服务器，这个时候运行elasticsearch可以外网访问

----安装kibana
sudo tar zxf kibana-6.2.4-linux-x86_64.tar.gz -C /usr/local
启动
cd /usr/local/kibana-6.2.4-linux-x86_64
bin/kibana

--外网访问
vim config/kibana.yml
    server.host: "0.0.0.0"

-----后续还要添加的
supervisor安装及配置

-------------------不再使用的-----------------------
----安装virtualenv、virtualenvwrapper
sudo pip3 install virtualenv
sudo pip3 install virtualenvwrapper
配置virtualenvwrapper环境, http://iampythoner.com/2017/10/virtualenv/#Linux_env

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


-------------------GUI Application-----------------------
Ctrl + H # 桌面版显示隐藏文件/夹

----安装VSCode
microsoft官网 https://code.visualstudio.com/
安装python插件 https://marketplace.visualstudio.com/items?itemName=ms-python.python

----安装pycharm
只需解压绿色版即可 打开bin/pycharm.sh

----安装sublime 3143
wget -nd https://download.sublimetext.com/sublime_text_3_build_3143_x64.tar.bz2
tar jxf sublime_text_3_build_3143_x64.tar.bz2
sudo mv sublime_text_3 /opt/sublime_text
cd /opt/sublime_text
sudo cp sublime_text.desktop /usr/share/applications/
sudo vim /usr/share/applications/sublime_text.desktop
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

