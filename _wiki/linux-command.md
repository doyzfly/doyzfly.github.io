---
layout: wiki
title: 常用的 Linux 命令
categories: Linux
description: 个人常用的 Linux 命令。
keywords: Linux
---

## 常用命令
### find
用于查找文件，例子：
- 查找 /root 目录下大于100M的文件
```bash
find ./root -size +100M
```

## 网络相关

### netstat
用于查询服务器TCP/UDP相关的内容，包括提供服务的端口信息，维持的连接信息。  
该命令默认可能没有安装，ubuntu系统安装命令为：  
```bash
sudo apt install net-tools
```
ubuntu系统安装命令为： 
```bash
sudo yum install -y net-tools
```

- 查看服务器当前提供TCP服务的端口信息
```bash
sudo netstat -nltp

doyzfly-server:~$ sudo netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      990/nginx: master p
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      710/systemd-resolve
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      961/sshd
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      5476/cupsd
tcp        0      0 0.0.0.0:8088            0.0.0.0:*               LISTEN      990/nginx: master p
tcp6       0      0 :::8080                 :::*                    LISTEN      1281/websock_rtsp_p
tcp6       0      0 :::80                   :::*                    LISTEN      990/nginx: master p
tcp6       0      0 :::22                   :::*                    LISTEN      961/sshd
tcp6       0      0 ::1:631                 :::*                    LISTEN      5476/cupsd
tcp6       0      0 :::4000                 :::*                    LISTEN      1281/websock_rtsp_p
```

- 查看服务器当前提供TCP服务的端口信息
```bash
sudo netstat -nlup

doyzfly-server:~$ sudo netstat -nlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           864/avahi-daemon: r
udp        0      0 127.0.0.53:53           0.0.0.0:*                           710/systemd-resolve
udp        0      0 0.0.0.0:631             0.0.0.0:*                           5477/cups-browsed
udp        0      0 0.0.0.0:44098           0.0.0.0:*                           864/avahi-daemon: r
udp6       0      0 :::5353                 :::*                                864/avahi-daemon: r
udp6       0      0 :::36924                :::*                                864/avahi-daemon: r
```

- 查看服务器维持的TCP连接信息
```bash
sudo netstat -nat

doyzfly-server:~$ sudo netstat -nat
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8088            0.0.0.0:*               LISTEN
tcp        0    172 10.161.32.200:22        10.161.61.93:49929      ESTABLISHED
tcp6       0      0 :::8080                 :::*                    LISTEN
tcp6       0      0 :::80                   :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:631                 :::*                    LISTEN
tcp6       0      0 :::4000                 :::*                    LISTEN
```



- 查看服务器维持的TCP连接的统计信息
```bash
sudo netstat -ant | awk '{++s[$NF]} END {for(k in s) print k,s[k]}'

doyzfly-server:~$ sudo netstat -ant | awk '{++s[$NF]} END {for(k in s) print k,s[k]}'
LISTEN 10
ESTABLISHED 1
established) 1
State 1
```
