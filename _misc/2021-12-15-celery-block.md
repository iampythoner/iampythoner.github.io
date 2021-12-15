---
layout: cnblog_post
title: celery_block
permalink: '/misc/celery_block'
date: 2021-12-15 00:47:39
categories: misc
---



celery 停止log 不再消费队列中的message

和这位博主一样的问题：
https://www.jianshu.com/p/e26590d0411a




但是我的stack是这样的：

```
# celery 起了三个进程，当前分别是 42793 42794 42795 都卡住了
cat /proc/42793/stack
```

```
[<0>] wait_woken+0x48/0x80
[<0>] sk_wait_data+0x123/0x140
[<0>] tcp_recvmsg+0x5a7/0xb30
[<0>] inet_recvmsg+0x5e/0xf0
[<0>] sock_recvmsg+0x69/0x80
[<0>] __sys_recvfrom+0x19e/0x1d0
[<0>] __x64_sys_recvfrom+0x29/0x30
[<0>] do_syscall_64+0x57/0x190
[<0>] entry_SYSCALL_64_after_hwframe+0x44/0xa9
```


sk_wait_data 等待接收socket数据
我想查到是请求哪个url的时候阻塞了，查看到这个pid对应到网络请求：

```
lsof | grep 42793
```

```
celery     42793                            root   24u     IPv4            3079361      0t0        TCP hostname:47298->xx.xx.xx.xx:http (CLOSE_WAIT)
celery     42793                            root   25u     IPv4            3582163      0t0        TCP hostname:51246->yy.yy.yy.yy:http (ESTABLISHED)
```

请求的ip是 `yy.yy.yy.yy`

然后ping

```
自己推测的url 得到的ip刚好吻合
```

之后又想查到这个阻塞多长时间了，也就是pid对应的网络请求持续的时间：

https://superuser.com/questions/565991/how-to-determine-the-socket-connection-up-time-on-linux

```
function suptime() {
    local addr=${1:?Specify the remote IPv4 address}
    local port=${2:?Specify the remote port number}
    # convert the provided address to hex format
    local hex_addr=$(python -c "import socket, struct; print(hex(struct.unpack('<L', socket.inet_aton('$addr'))[0])[2:10].upper().zfill(8))")
    local hex_port=$(python -c "print(hex($port)[2:].upper().zfill(4))")
    # get the PID of the owner process
    local pid=$(netstat -ntp 2>/dev/null | awk '$6 == "ESTABLISHED" && $5 == "'$addr:$port'"{sub("/.*", "", $7); print $7}')
    [ -z "$pid" ] && { echo 'Address does not match' 2>&1; return 1; }
    # get the inode of the socket
    local inode=$(awk '$4 == "01" && $3 == "'$hex_addr:$hex_port'" {print $10}' /proc/net/tcp)
    [ -z "$inode" ] && { echo 'Cannot lookup the socket' 2>&1; return 1; }
    # query the inode status change time
    local timestamp=$(find /proc/$pid/fd -lname "socket:\[$inode\]" -printf %T@)
    [ -z "$timestamp" ] && { echo 'Cannot fetch the timestamp' 2>&1; return 1; }
    # compute the time difference
    LANG=C printf '%s (%.2fs ago)\n' "$(date -d @$timestamp)" $(bc <<<"$(date +%s.%N) - $timestamp")
}
```

```
addr=yy.yy.yy.yy
port=80
hex_addr=$(python -c "import socket, struct; print(hex(struct.unpack('<L', socket.inet_aton('$addr'))[0])[2:10].upper().zfill(8))")
hex_port=$(python -c "print(hex($port)[2:].upper().zfill(4))")
pid=$(netstat -ntp 2>/dev/null | awk '$6 == "ESTABLISHED" && $5 == "'$addr:$port'"{sub("/.*", "", $7); print $7}')
inode=$(awk '$4 == "01" && $3 == "'$hex_addr:$hex_port'" {print $10}' /proc/net/tcp)
# 得到 inode 3585392
timestamp=$(find /proc/42793/fd -lname "socket:\[3585392\]" -printf %T@)
LANG=C printf '%s (%.2fs ago)\n' "$(date -d @$timestamp)" $(bc <<<"$(date +%s.%N) - $timestamp")
```

得到
```
Tue 14 Dec 2021 01:53:46 AM CST (81465.32s ago)
```

