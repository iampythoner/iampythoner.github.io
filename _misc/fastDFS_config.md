---
permalink: /misc/fastDFS_cnfig
layout: cnblog_post
title:  'fastDFS配置'
date:   2017-12-06 06:34:39
categories: misc
---

<!--Category-->
<div id="navCategory">
    <b>本文目录</b>
	<ul>
		<li><a href="#anchor1_0">安装fastDFS</a></li>
        <li><a href="#anchor2_0">编译安装nginx，安装mod_fastfds模块</a></li>
        <li><a href="#anchor3_0">nginx为fastDFS管理的资源提供访问接口</a></li>
	</ul>
</div><br>

<h2 id="anchor1_0">安装fastDFS</h2>

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

# 测试客户端
sudo cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
sudo vim /etc/fdfs/client.conf
#修改bash_path为base_path=/home/mike/fastdfs/tracker
#修改tracker_server为刚才配置tracker_server的ip:如tracker_server=172.16.134.146:22122
fdfs_upload_file /etc/fdfs/client.conf ~/Desktop/mybin.tar
# 成功之后返回资源ID，如
group1/M00/00/00/rBCGklonohaAMthdAAB4AO3j5Os085.tar
```



<h2 id="anchor2_0">编译安装nginx，安装mod_fastfds模块</h2>

如果本机上本来有使用apt安装的nginx，先卸载

```sh
sudo nginx -s stop
sudo apt remove nginx

sudo rm -rf /usr/sbin/nginx
sudo rm -rf /etc/nginx
sudo rm -rf /var/log/nginx
sudo rm -rf /usr/sbin/nginx
sudo rm -rf /var/www/html
```

开始编译安装nginx和mod_fastfds:

①下载fastdfs-nginx-module

```sh
cd ~/Documents/lib/fastdfs
git clone https://github.com/happyfish100/fastdfs-nginx-module
```

②编译安装nginx和mod_fastfds

使用apt安装依赖项

```
sudo apt install libtool
sudo apt install libpcre3 libpcre3-dev
sudo apt install zlib1g-dev
sudo apt install openssl libssl-dev
```

下载nginx源码

```
cd ~/Documents/lib
wget -nd http://124.205.69.131/files/91820000052E0119/nginx.org/download/nginx-1.10.2.tar.gz
tar zxf nginx-1.10.3.tar.gz
cd nginx-1.10.3
```

配置编译选项

```
# configure
sudo ./configure --prefix=/usr/local/nginx --add-module=/home/mike/Documents/lib/fastdfs/fastdfs-nginx-module/src
# 如果你的电脑上本来就是编译安装了nginx，可以直接使用下面的方式增加module， 具体参照https://github.com/happyfish100/fastdfs-nginx-module/blob/master/INSTALL
# ./configure --add-module=/home/mike/Documents/lib/fastdfs/fastdfs-nginx-module/src
```

然后生成了 objs文件夹，里面是编译和安装的配置文件，同时生成了Makefile文件，配置成功出现下面的内容

```
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + md5: using system crypto library
  + sha1: using system crypto library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

编译

```
sudo make # 相当于执行 make -f objs/Makefile 
```

如果出现了只是警告却报错的情况，如下:

```
src/core/ngx_murmurhash.c: In function ‘ngx_murmur_hash2’:
src/core/ngx_murmurhash.c:37:11: error: this statement may fall through [-Werror=implicit-fallthrough=]
         h ^= data[2] << 16;
         ~~^~~~~~~~~~~~~~~~
src/core/ngx_murmurhash.c:38:5: note: here
     case 2:
     ^~~~
src/core/ngx_murmurhash.c:39:11: error: this statement may fall through [-Werror=implicit-fallthrough=]
         h ^= data[1] << 8;
         ~~^~~~~~~~~~~~~~~
src/core/ngx_murmurhash.c:40:5: note: here
     case 1:
     ^~~~
cc1: all warnings being treated as errors
objs/Makefile:467: recipe for target 'objs/src/core/ngx_murmurhash.o' failed
make[1]: *** [objs/src/core/ngx_murmurhash.o] Error 1
make[1]: Leaving directory '/home/mike/Documents/lib/nginx-1.10.3'
Makefile:8: recipe for target 'build' failed
make: *** [build] Error 2
```

出现警告报错的情况可以这样解决：

```
sudo vim objs/Makefile
去掉'-Werror'
```
 
执行安装

```
sudo make install
# 安装完成之后，建个软连接
sudo ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
```
如果想设置开启自动启动，可以仿照apt安装的方式生成的nginx服务文件写一个开机启动的shell，然后放在/etc/init.d下面，下面是apt安装nginx生成的/etc/init.d/nginx文件：

```
#!/bin/sh

### BEGIN INIT INFO
# Provides:	  nginx
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/nginx
NAME=nginx
DESC=nginx

# Include nginx defaults if available
if [ -r /etc/default/nginx ]; then
	. /etc/default/nginx
fi

STOP_SCHEDULE="${STOP_SCHEDULE:-QUIT/5/TERM/5/KILL/5}"

test -x $DAEMON || exit 0

. /lib/init/vars.sh
. /lib/lsb/init-functions

# Try to extract nginx pidfile
PID=$(cat /etc/nginx/nginx.conf | grep -Ev '^\s*#' | awk 'BEGIN { RS="[;{}]" } { if ($1 == "pid") print $2 }' | head -n1)
if [ -z "$PID" ]; then
	PID=/run/nginx.pid
fi

if [ -n "$ULIMIT" ]; then
	# Set ulimit if it is set in /etc/default/nginx
	ulimit $ULIMIT
fi

start_nginx() {
	# Start the daemon/service
	#
	# Returns:
	#   0 if daemon has been started
	#   1 if daemon was already running
	#   2 if daemon could not be started
	start-stop-daemon --start --quiet --pidfile $PID --exec $DAEMON --test > /dev/null \
		|| return 1
	start-stop-daemon --start --quiet --pidfile $PID --exec $DAEMON -- \
		$DAEMON_OPTS 2>/dev/null \
		|| return 2
}

test_config() {
	# Test the nginx configuration
	$DAEMON -t $DAEMON_OPTS >/dev/null 2>&1
}

stop_nginx() {
	# Stops the daemon/service
	#
	# Return
	#   0 if daemon has been stopped
	#   1 if daemon was already stopped
	#   2 if daemon could not be stopped
	#   other if a failure occurred
	start-stop-daemon --stop --quiet --retry=$STOP_SCHEDULE --pidfile $PID --name $NAME
	RETVAL="$?"
	sleep 1
	return "$RETVAL"
}

reload_nginx() {
	# Function that sends a SIGHUP to the daemon/service
	start-stop-daemon --stop --signal HUP --quiet --pidfile $PID --name $NAME
	return 0
}

rotate_logs() {
	# Rotate log files
	start-stop-daemon --stop --signal USR1 --quiet --pidfile $PID --name $NAME
	return 0
}

upgrade_nginx() {
	# Online upgrade nginx executable
	# http://nginx.org/en/docs/control.html
	#
	# Return
	#   0 if nginx has been successfully upgraded
	#   1 if nginx is not running
	#   2 if the pid files were not created on time
	#   3 if the old master could not be killed
	if start-stop-daemon --stop --signal USR2 --quiet --pidfile $PID --name $NAME; then
		# Wait for both old and new master to write their pid file
		while [ ! -s "${PID}.oldbin" ] || [ ! -s "${PID}" ]; do
			cnt=`expr $cnt + 1`
			if [ $cnt -gt 10 ]; then
				return 2
			fi
			sleep 1
		done
		# Everything is ready, gracefully stop the old master
		if start-stop-daemon --stop --signal QUIT --quiet --pidfile "${PID}.oldbin" --name $NAME; then
			return 0
		else
			return 3
		fi
	else
		return 1
	fi
}

case "$1" in
	start)
		log_daemon_msg "Starting $DESC" "$NAME"
		start_nginx
		case "$?" in
			0|1) log_end_msg 0 ;;
			2)   log_end_msg 1 ;;
		esac
		;;
	stop)
		log_daemon_msg "Stopping $DESC" "$NAME"
		stop_nginx
		case "$?" in
			0|1) log_end_msg 0 ;;
			2)   log_end_msg 1 ;;
		esac
		;;
	restart)
		log_daemon_msg "Restarting $DESC" "$NAME"

		# Check configuration before stopping nginx
		if ! test_config; then
			log_end_msg 1 # Configuration error
			exit $?
		fi

		stop_nginx
		case "$?" in
			0|1)
				start_nginx
				case "$?" in
					0) log_end_msg 0 ;;
					1) log_end_msg 1 ;; # Old process is still running
					*) log_end_msg 1 ;; # Failed to start
				esac
				;;
			*)
				# Failed to stop
				log_end_msg 1
				;;
		esac
		;;
	reload|force-reload)
		log_daemon_msg "Reloading $DESC configuration" "$NAME"

		# Check configuration before stopping nginx
		#
		# This is not entirely correct since the on-disk nginx binary
		# may differ from the in-memory one, but that's not common.
		# We prefer to check the configuration and return an error
		# to the administrator.
		if ! test_config; then
			log_end_msg 1 # Configuration error
			exit $?
		fi

		reload_nginx
		log_end_msg $?
		;;
	configtest|testconfig)
		log_daemon_msg "Testing $DESC configuration"
		test_config
		log_end_msg $?
		;;
	status)
		status_of_proc -p $PID "$DAEMON" "$NAME" && exit 0 || exit $?
		;;
	upgrade)
		log_daemon_msg "Upgrading binary" "$NAME"
		upgrade_nginx
		log_end_msg $?
		;;
	rotate)
		log_daemon_msg "Re-opening $DESC log files" "$NAME"
		rotate_logs
		log_end_msg $?
		;;
	*)
		echo "Usage: $NAME {start|stop|restart|reload|force-reload|status|configtest|rotate|upgrade}" >&2
		exit 3
		;;
esac
```

<h2 id="anchor3_0">nginx为fastDFS管理的资源提供访问接口</h2>

```sh
# 添加配置文件
cd ~/Documents/lib/fastdfs/fastdfs-nginx-module/
sudo cp mod_fastdfs.conf /etc/fdfs
sudo cp http.conf /etc/fdfs
sudo cp mime.types /etc/fdfs

# 修改 mod_fastdfs.conf
sudo vim /etc/fdfs/mod_fastdfs.conf
# 以下是修改的内容
connect_timeout=10
tracker_server=ip:22122
url_have_group_name=true
store_path0=/home/mike/fastdfs/storage

# 修改nginx配置文件，添加fastDFS服务
sudo vim /usr/local/nginx/conf/nginx.conf
# 添加以下服务
server {
    listen       8888;
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
# 启动nginx
sudo nginx
```



