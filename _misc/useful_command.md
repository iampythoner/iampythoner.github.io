---
permalink: /misc/useful_command
layout: cnblog_post
title:  '非常有用的命令'
date:   2017-10-22 06:34:39
categories: misc
---

常看端口占用情况：

```
sudo lsof -i:80 # 只显示端口 sudo lsof -Pti:80
netstat -tunlp|grep
```
这是最服百度的一次：<a href="https://jingyan.baidu.com/article/546ae1853947b71149f28cb7.html" target='blank'>linux如何查看端口被哪个进程占用？</a>

查看指定协议的端口（使用netstat也是可以的）

```
# lsof -i [protocol] : port
sudo lsof -i tcp:3306
# 结果
COMMAND    PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mysqld  108931 mysql   44u  IPv4 575858      0t0  TCP *:mysql (LISTEN)
```


vim 修改了只读文件，但是想保存住呀

```
:w !sudo tee %
```
:w : Write a file.可以将文件写入，文件仍然是只读模式，通过 :q! 退出<br>
!sudo : Call shell sudo command.<br>
tee : The output of the vi/vim write command is redirected using tee.
% : Triggers the use of the current filename.<br>
Simply put, the ‘tee’ command is run as sudo and follows the vi/vim command on the current filename given.<br>


查看发行版信息

```
cat /proc/version

lsb_release -a
```

查看文件的16进制数据

```
xxd
``` 

木有ip了：

```
dhclient
```

ubuntu桌面版拨号联网

```
nm-connection-editor
```


查看指定协议、指定端口的使用情况 


#### 文本处理

```
awk 'BEGIN{print ""}{print $2}' 1.txt # 换行后输出每行
awk 'BEGIN{print "["}{printf("'\''%s'\'',\n", $2)}END{print "]"}' 1.txt # 将$2放到Python list中
awk 'BEGIN{printf("[")}{printf("'\''%s'\'', ", $2)}END{printf("]")}' 1.txt # 将$2放到Python list中, 不换行

# awk 正则
awk '{match($0, /^a{3}$/, match_result); if (match_result[0]) print $0}' a.txt #正则匹配行内容为'aaa', 输出该行，可简写为下面
awk '{if ($0 ~ /^a{3}$/) print $0}' a.txt
# 分组
awk '{match($0, /^(a{3})$/, match_result); if (match_result[1]) print match_result[1]}' a.txt # 正则匹配行内容为'aaa', 输出分组内容

# comm 比较的前提是都有序
comm -12 a.txt b.txt # 输出相同的行
comm -23 a.txt b.txt # 只在a中不在b中
comm -13 a.txt b.txt # 只在b中不在a中
comm -3 a.txt b.txt  # a特有的和b特有的
comm a.txt b.txt # a、b中全部的

# cut
cut -d' ' -f4 1.txt | sort -r # 以空格分割后按照第二列逆序排列 # d: delimiter
cut -c3- 1.txt # 从每行的第3列的字符开始输出
cut -c3-4 1.txt # 从每行的第3列的字符开始输出到第4列的字符(包含第4列) # c: column

sed -n 8p xx.log # 查看第8行

shuf -n 10 xx.log # 随机取10行

sort -u 等价于 sort | uniq
sort 1.txt | uniq -c  # 统计相同行的个数
sort -t' ' -nk 2 -r # 按照空格分隔，第2列按数字倒序 t:分隔符， n:number r: reverse
# 上面相当于 sort -t' ' -n -k 2 -r 2.txt
# -k 详解
FStart.CStart Modifie,FEnd.CEnd Modifier
-------Start--------,-------End--------
 FStart.CStart 选项  ,  FEnd.CEnd 选项
sort -t' ' -nk 2.1 # 第2列的第一个字符
sort -t' ' -nk 2.1,2.2 # 第2列的第一个字符到第二个字符, 按数字排序
sort -t' ' -nk 2.2 2.txt # 第2列的第二个字符, 按数字排序
sort -t' ' -nk 2.2 -k 1 2.txt # 先按第二列第2个字符排序，若第二个字符相同则按照第1列排序

# tac
tac xx.log # 文件倒序输出，按行：倒数第一行编程第一行

# tr
cat 1.txt | tr '[a-z]' '[A-Z]' # 将小写字母翻译为大写字母

当前文件夹下的文件按照大小排序：
ls -l | sort -nk 5 -r

# 编码
enca --list languages # 查看enca 支持的语言
enca 1.txt -L none # 查看文件编码
iconv --list # 查看iconv支持的编码
iconv -f UCS-2 -t UTF-8 xx.log -o converted.log # 将 UCS-2编码的xx.log 转为 UTF-8编码的converted.log 
```

#### 网络相关


```
wget -x # 强制保留文件夹结构，如wget -x http://fly.srk.fer.hr/robots.txt 保存文件到 fly.srk.fer.hr/robots.txt
wget -nd # 与-x 相反，不保留文件夹结构
wget https://www.gitignore.io/api/ruby -O .gitignore # wget -大O 输出到文件
wget https://www.gitignore.io/api/ruby  -P ~/Documents -O .gitignore # -P 输出到文件夹prefix , 默认是.

nc -l 8080 # 开启TCP/UDP连接


hostname # hostname 管理

netstat -tunlen
# t: TCP u: UDP
# n:Address  and  port  number  of  the  local  end of the socket
# l:Show only listening sockets
# e:（extend）Display additional information
sudo lsof -i:8080 # 查看8080端口的占用情况
```


#### 磁盘管理

```
du --max-depth=1 # mac: du -d1
df 
```
