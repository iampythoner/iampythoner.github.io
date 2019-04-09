---
layout: cnblog_post
title:  "mac_lang_env"
permalink: '/misc/mac_lang_env'
date:   2018-05-05 07:34:39
categories: misc
---

mac 上各种语言环境的配置

mac cmd + shift + F 最近使用

### Java

```
$ ls /Library/Java/JavaVirtualMachines
### /Library/Java/JavaVirtualMachines 这个目录本身为空，当安装了JDK之后才会有内容，如果安装了多个JDK，都会在这个目录下


$ /usr/libexec/java_home
Unable to find any JVMs matching version "(null)".
No Java runtime present, try --request to install.

### 注意： /usr/libexec/java_home 链接到/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java_home，这个文件还有整个/System/Library/Frameworks/JavaVM.framework目录下的内容不会改变，他会动态读取系统中最新安装的JDK作为默认的JDK，因此在环境变量中做如下配置和不配置的效果是一样的，都是用了最新安装的JDK：

export JAVA_HOME=$(/usr/libexec/java_home)
可以验证测试 在安装新的JDK之后，执行source ~/.bash_profile之后 JAVA_HOME的值也通过/usr/libexec/java_home命令执行之后发生了改变：如我在安装10.0.1的JDK之后，变为如下

echo $JAVA_HOME  # /Library/Java/JavaVirtualMachines/jdk-10.0.1.jdk/Contents/Home

因此对JDK安装目录和/usr/libexec/java_home的总结如下：
/Library/Java/JavaVirtualMachines 生成目录新安装的JDK的目录
/usr/libexec/java_home 运行链接原身后指向新安装的JDK版本的新生成的目录下的/Contents/Home

所以想要在环境变量中配置指定的JDK版本，应该这样：
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home"
export CLASS_PATH="$JAVA_HOME/lib"
export PATH=".:$JAVA_HOME/bin:$PATH"


考虑到频繁切换时反复修改~/.bash_profile比较麻烦，所以可以在/Library/Java/JavaVirtualMachines/下创建Current软链接，指向要切换的版本，这样每次只要修改软链接即可。
```

### Python 

```
安装之前/Library/Frameworks/Python.framework目录不存在
系统自带的Python安装在这个目录/System/Library/Frameworks/Python.framework ,试用pip安装三方库带的二进制工具，也没有安装到这个目录(暂不确定，总之不太方便管理，即使想使用Python2也建议在官网下载安装包安装，而不是直接使用系统自带的)。
安装新的Python环境放在/Library/Frameworks/Python.framework目录(无论是2、3都会放在这个位置)

使用安装包安装Python2，会在写bashrc(Python3安装包会写)，因此python和python2依然是系统自带的，如果想试用自己的安装的Python2，还得手动修改一下bashrc
vim ~/.bash_profile
    PATH="/Library/Frameworks/Python.framework/Versions/2.7/bin:$PATH"
# 为Python2 安装ipython
# 先通过确认一下当前使用的Python和pip2
pip2 -V # pip 9.0.1 from /Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages (python 2.7)
# 安装 
pip2 install ipython -i https://pypi.douban.com/simple/

为Python3安装ipython 同上，只不过将pip2 换成pip3
因为我在~/.bash_profile设置Python3的bin目录优先搜索，所以安装完ipython之后，启动的是python3的ipython， 可以直接将/Library/Frameworks/Python.framework/Versions/3.6/bin目录下的ipython直接删掉(这个目录下还有一个ipython3可执行文件),这样就可以做到，ipython启动2的ipython，ipython3启动3的ipython。


# 把上面的都安装完之后再安装pyenv

pyenv 查看当前python解释器路径
pyenv which python
```

### Go

```
下载dmg安装包安装，然后做如下配置：

export GOROOT=/usr/local/go
export GOPATH=~/Documents/go
export PATH=$GOPATH/bin:$PATH
```

### Ruby

```
如何快速正确的安装 Ruby, Rails 运行环境
https://ruby-china.org/wiki/install_ruby_guide/

rvm 官网
https://www.rvm.io/

ruby官网
https://www.ruby-lang.org/
```


先翻个墙

```
$ brew install gpg
$ brew install gnupg
$ brew upgrade gnupg

-------- 如果.zshrc、.bash_profile等shell配置中有如下内容，务必注释掉！！！！！！！！,否则导入的签名不会起作用
# export SSL_CERT_FILE=$HOME/GlobalSignRootCA.pem

$ gpg --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

gpg: key 105BD0E739499BDB: 5 个签名因密钥遗失而未被检查
gpg: 密钥 105BD0E739499BDB：公钥 “Piotr Kuczynski <piotr.kuczynski@gmail.com>” 已导入
gpg: key 3804BB82D39DC0E3: 105 个签名因密钥遗失而未被检查
gpg: 密钥 3804BB82D39DC0E3：“Michal Papis (RVM signing) <mpapis@gmail.com>” 59 个新的签名
gpg: 未找到任何绝对信任的密钥
gpg: 处理的总数：2
gpg:               已导入：1
gpg:         新的签名：59


---旧的：
    $ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

    gpg: directory '/Users/mike/.gnupg' created
    gpg: keybox '/Users/mike/.gnupg/pubring.kbx' created
    gpg: 从公钥服务器接收失败：Server indicated a failure


$ curl -sSL https://rvm.io/mpapis.asc | gpg --import -

gpg: key 3804BB82D39DC0E3: 47 signatures not checked due to missing keys
gpg: /Users/mike/.gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 3804BB82D39DC0E3：公钥“Michal Papis (RVM signing) <mpapis@gmail.com>”已导入
gpg: 合计被处理的数量：1
gpg:           已导入：1
gpg: 没有找到任何绝对信任的密钥


$ curl -sSL https://get.rvm.io | bash -s stable
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: 签名建立于 一  9/11 04:59:21 2017 CST
gpg:               使用 RSA 密钥 E206C29FBF04FF17
gpg: 完好的签名，来自于“Michal Papis (RVM signing) <mpapis@gmail.com>” [未知]
gpg:               亦即“Michal Papis <michal.papis@toptal.com>” [未知]
gpg:               亦即“[jpeg image of size 5015]” [未知]
gpg: 警告：这把密钥未经受信任的签名认证！
gpg:       没有证据表明这个签名属于它所声称的持有者。
主钥指纹： 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
子钥指纹： 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/Users/mike/.rvm/archives/rvm-1.29.3.tgz'

Installing RVM to /Users/mike/.rvm/
    Adding rvm PATH line to /Users/mike/.profile /Users/mike/.mkshrc /Users/mike/.bashrc /Users/mike/.zshrc.
    Adding rvm loading line to /Users/mike/.profile /Users/mike/.bash_profile /Users/mike/.zlogin.
Installation of RVM in /Users/mike/.rvm/ is almost complete:

  * To start using RVM you need to run `source /Users/mike/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
```


```
# 查看当前rvm版本
rvm -v
# rvm 1.29.3 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]

# 将ruby china源添加到配置文件中
echo "ruby_url=https://cache.ruby-china.org/pub/ruby" >> ~/.rvm/user/db
```

```
rvm requirements # installs dependencies for building ruby

#### 遇到这个错误：
Checking requirements for osx.
Updating Homebrew...
Installing requirements for osx.
Updating system.........
Installing required custom packages: homebrew/versions.
Error running 'requirements_osx_brew_install_custom homebrew/versions',
please read /Users/mike/.rvm/log/1529740655/install_custom.log
Requirements installation failed with status: 1.

#### 查看上面的日志：
$ cat ~/.rvm/log/1529740655/install_custom.log
[2018-06-23 15:57:37] requirements_osx_brew_install_custom
requirements_osx_brew_install_custom ()
{
    \typeset __tap;
    for __tap in "$@";
    do
        brew tap "${__tap}" || return $?;
    done
}
current path: /Users/mike
PATH=.:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/bin:/Users/mike/Documents/go/bin:/Users/mike/.pyenv/plugins/pyenv-virtualenv/shims:/Users/mike/.pyenv/shims:/Users/mike/.pyenv/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:.:/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/bin:/Users/mike/Documents/go/bin:/Users/mike/.pyenv/plugins/pyenv-virtualenv/shims:/Users/mike/.pyenv/shims:/Users/mike/.pyenv/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/VMware Fusion.app/Contents/Public:/usr/local/go/bin:/Users/mike/.rvm/bin:/Users/mike/.rvm/bin:/Users/mike/.rvm/bin
command(2): requirements_osx_brew_install_custom homebrew/versions
+ typeset __tap
+ for __tap in '"$@"'
+ brew tap homebrew/versions
Error: homebrew/versions was deprecated. This tap is now empty as all its formulae were migrated.
+ return 1


# 在github上找到了解决方法：https://github.com/rvm/rvm/issues/4303
# 将rvm checkout为最新的稳定版, 相当于
rvm get head

# 然后继续执行
rvm requirements # 这次ok了
Checking requirements for osx.
Installing requirements for osx.
Updating system..........
Installing required packages: autoconf, automake, libtool, pkg-config, coreutils, gcc@6, openssl@1.1........
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.

# 安装2.4.4版本Ruby
rvm install 2.4.4

# 查看安装的所有解释器
rvm list

# 查看当前的版本
rvm current

rvm use <version> # 切换
rvm remove <version> # 移除
rvm list known # 查看可以安装的所有版本, 如果没有显示最新版本应先 rvm get head

rvm use 2.2.1 --default # 使用2.2.1版本 并设置为rvm默认ruby版本
```

Bunlde 和 gem 使用

```
# 每个rvm添加的ruby环境默认内置了gem和bundle
# gem负责管理三方库、bundle负责项目依赖管理(三方库版本一致等管理)

# 更换当前gem使用的源
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
# 查看当前源
gem sources --list
# 更新缓存
gem sources -u
# 安装三方库
gem install xxx
# 从本机安装gem包
gem install -l [gemname].gem
# 安装三方库时指定版本
gem install [gemname] --version=[ver]
# 删除已经安装的指定包
gem uninstall [gemname] --version=[ver]
# 更新已经安装的所有gem包
gem update
# 更新指定的三方包
gem update [gemname] --version=[ver]

# 查看已经安装的三方库
gem list
# 更新 gem
gem update --system


# bundle 基本操作
# 创建Gemfile,使用bundle进行依赖管理
bundle init
# 安装Gemfile里面的依赖库
bundle install
```