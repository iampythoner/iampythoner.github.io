---
layout: cnblog_post
title:  "deploy_with_git_hook"
permalink: '/misc/deploy_with_git_hook'
date:   2018-01-22 07:34:39
categories: misc
---


主要参考 https://www.jekyll.com.cn/docs/deployment-methods/

#### 创建 server 钩子

```
ssh deployer@xx.xx.xx.xx
mkdir -p Deploy/iampythoner.git
cd Deploy/iampythoner.git/
git --bare init
vim hooks/post-receive
chmod a+x hooks/post-receive
	#!/bin/sh

	GIT_REPO=$HOME/Deploy/iampythoner.git
	#GIT_REPO=https://github.com/iampythoner/iampythoner.github.io
	TMP_GIT_CLONE=$HOME/tmp/iampythoner
	PUBLIC_WWW=/var/www/iampythoner

	git clone $GIT_REPO $TMP_GIT_CLONE
	jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
	rm -Rf $TMP_GIT_CLONE
	exit
```

#### 开发机器配置远程仓库

```
git remote add deploy username@xx.xx.xx.xx:~/Deploy/iampythoner.git
```

#### nginx 配置

```
http {
	# ....
	include iampythoner.conf
}
```

ampythoner.conf

```
server {
    listen       80;
    server_name  iampythoner.com www.iampythoner.com;

    location / {
         alias /var/www/iampythoner/;
    }
}
```

#### DNS解析

| 记录类型 | 主机记录 | 解析线路(isp) | 记录值 |
| -- | -- | -- | -- |
| CNAME | www | 默认 | iampythoner.com |
| A | @ | 默认 | xx.xx.xx.xx |


#### 补充

有几个不足的地方，不应该每次都是删除之前的，然后git clone,
应该使用git pull 做增量更新。

另外，由于产生的静态文件和jekyll动态运行的结果不一致，所以现在没有使用nginx，而是使用了jekyll测试服务器，并且没有使用nginx


```
#!/bin/sh                   
                                                                                
GIT_REPO=$HOME/Deploy/iampythoner.git
TMP_GIT_CLONE=$HOME/tmp/iampythoner
PUBLIC_WWW=/var/www/iampythoner
                            
#git clone $GIT_REPO $TMP_GIT_CLONE
#jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
#rm -Rf $TMP_GIT_CLONE      
                            
# if folder is empty        
#git clone $GIT_REPO $PUBLIC_WWW 
unset $(git rev-parse --local-env-vars)
cd $PUBLIC_WWW              
git pull $GIT_REPO          
pkill -f jekyll             
bundle exec jekyll server -BI 
#jekyll serve -BI -s $PUBLIC_WWW
exit  
```


##### 附: jekyll 组件安装过程

```
sudo apt install ruby ruby-dev
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
gem install jekyll-paginate jekyll-gist jekyll-archives # 缺少的依赖
sudo gem install jekyll
```




