---
title: ssh + iptables + redsocks实现ubuntu全局代理
date: 2019-03-13 21:37:08
categories: 
- [linux]
- [网络]
tags: 
- linux
- 网络
- 代理
---
# 需求
在两台搭载ubuntu的服务器A，B间搭一个跳板，实现A上所有TCP流量由B转发。

# 实现方法
在服务器A上与B建立起SSH隧道，指定转发端口p，然后用iptables将所有TCP流量转发到redsocks的侦听端口，再由redsocks转到端口p发出。

<!-- more -->

# SSH
SSH（Secure Shell）是一种加密网络传输协议，可在不安全的网络中提供安全传输。
## 安装
类Unix系统自带ssh命令，Windows下可用Putty等工具代替。
## SSH隧道
ssh端口转发在两台主机之间搭建了一个数据传输的安全隧道，因此也被称为ssh隧道。ssh端口转发有三种模式：本地端口转发、远程端口转发、动态端口转发。具体内容可参考这篇[玩转SSH端口转发](https://cloud.tencent.com/developer/article/1335416)。

在这里使用SSH的动态端口转发，指定本地端口11223.

```shell
# 将A上经过11223端口的流量由B转发
ssh -D localhost:11223 B_User@B_IP
```
接下来就只需要想办法让A上所有TCP流量都由11223端口出去。


# iptables
iptables是用于配置Linux内核防火墙的命令行工具，它可以按规则检测、修改、转发、重定向和丢弃IPv4数据包。iptables规则主要由表和链组成，[这里](https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))有详细介绍。
## 规则
这里使用iptables把tcp流量全部转发到redsocks的侦听端口11111上（udp等也可以，我没有配置）。

```
# 在nat表上新建REDSOCKS链
iptables -t nat -N REDSOCKS

# 忽略服务器B地址
iptables -t nat -A REDSOCKS -d B_IP -j RETURN 

# 忽略本地地址 
iptables -t nat -A REDSOCKS -d 0.0.0.0/8 -j RETURN 
iptables -t nat -A REDSOCKS -d 10.0.0.0/8 -j RETURN 
iptables -t nat -A REDSOCKS -d 127.0.0.0/8 -j RETURN 
iptables -t nat -A REDSOCKS -d 169.254.0.0/16 -j RETURN 
iptables -t nat -A REDSOCKS -d 172.16.0.0/12 -j RETURN 
iptables -t nat -A REDSOCKS -d 192.168.0.0/16 -j RETURN 
iptables -t nat -A REDSOCKS -d 224.0.0.0/4 -j RETURN 
iptables -t nat -A REDSOCKS -d 240.0.0.0/4 -j RETURN

# 其余tcp包都转发到redsocks端口
iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 11111 
iptables -t nat -A OUTPUT -p tcp -j REDSOCKS 

```

# redsocks

redsocks是一个开源TCP重定向工具，它的github地址在[这里](https://github.com/darkk/redsocks)。

## 安装

```
sudo apt-get update
sudo apt-get install redsocks
```
## 启动服务

```
service redsocks start
```

## 配置
redsocks在ubuntu下的配置文件路径为`/etc/redsocks.conf`。

我的配置文件如下，只配置了tcp：

```
base {
	// debug: connection progress & client list on SIGUSR1
	log_debug = off;

	// info: start and end of client session
	log_info = on;

	/* possible `log' values are:
	 *   stderr
	 *   "file:/path/to/file"
	 *   syslog:FACILITY  facility is any of "daemon", "local0"..."local7"
	 */
	log = "syslog:daemon";

	// detach from console
	daemon = on;

	/* Change uid, gid and root directory, these options require root
	 * privilegies on startup.
	 * Note, your chroot may requre /etc/localtime if you write log to syslog.
	 * Log is opened before chroot & uid changing.
	 */
	user = redsocks;
	group = redsocks;
	// chroot = "/var/chroot";

	/* possible `redirector' values are:
	 *   iptables   - for Linux
	 *   ipf        - for FreeBSD
	 *   pf         - for OpenBSD
	 *   generic    - some generic redirector that MAY work
	 */
	redirector = iptables;
}

redsocks {
	/* `local_ip' defaults to 127.0.0.1 for security reasons,
	 * use 0.0.0.0 if you want to listen on every interface.
	 * `local_*' are used as port to redirect to.
	 * 本地redsocks要监听的地址和端口
	 */

	local_ip = 127.0.0.1;
	local_port = 11111;

	// `ip' and `port' are IP and tcp-port of proxy-server
	// You can also use hostname instead of IP, only one (random)
	// address of multihomed host will be used.
	// socks地址和端口(ssh创建的socks隧道)
	ip = 127.0.0.1;
	port = 11223;


	// known types: socks4, socks5, http-connect, http-relay
	type = socks5;

}

```

# 一键脚本
一键才是最棒的。这个脚本参考了liruqi的博客文章 [使用 redsocks 作强制的系统全局代理](https://liruqi.wordpress.com/2011/04/07/redsocks-for-global-proxy/)。

```
#! /bin/bash

start_ssh() 
{ 
    echo "Start SSH Tunnel : " 
    # redsocks官方不建议使用此工具转发tor流量，但我emm
    torify ssh -Nf -D 11223 user@b_ip
    echo "SSH Tunnel Started." 
}

stop_ssh() 
{ 
	# 这里写得很不好，不过可以用
    SSHID=$(ps -ef|grep 54321|awk '{print $2}')
    SSHID=$SSHID |awk '{print $1}'
    echo $SSHID
    kill -9 ${SSHID}
    echo "SSH Tunnel Daemon Stoped." 
}

case "$1" in 
  start) 
    start_ssh 
    service redsocks start

    # 清空nat表，添加新链 
    iptables -t nat -F
    iptables -t nat -N REDSOCKS

    # 忽略本地地址
    iptables -t nat -A REDSOCKS -d 0.0.0.0/8 -j RETURN 
    iptables -t nat -A REDSOCKS -d 10.0.0.0/8 -j RETURN 
    iptables -t nat -A REDSOCKS -d 127.0.0.0/8 -j RETURN 
    iptables -t nat -A REDSOCKS -d 169.254.0.0/16 -j RETURN 
    iptables -t nat -A REDSOCKS -d 172.16.0.0/12 -j RETURN 
    iptables -t nat -A REDSOCKS -d 192.168.0.0/16 -j RETURN 
    iptables -t nat -A REDSOCKS -d 224.0.0.0/4 -j RETURN 
    iptables -t nat -A REDSOCKS -d 240.0.0.0/4 -j RETURN

    # Anything else should be redirected to port 11111
    iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 11111
    # Any tcp connection should be redirected. 
    iptables -t nat -A OUTPUT -p tcp -j REDSOCKS 
    ;;

  stop) 
    stop_ssh 
    killall -9 redsocks 
    
    # 删除iptables规则 
    iptables -t nat -F OUTPUT 
    iptables -t nat -F REDSOCKS 
    iptables -t nat -X REDSOCKS 
    ;;

  start_ssh) 
    start_ssh 
    ;;

  stop_ssh) 
    stop_ssh 
    ;;


  *) 
    echo "Usage: redsocks start|stop|start_ssh|stop_ssh" >&2 
    exit 3 
    ;; 
esac

```

# 其他

1、建立使用tor的ssh：在两端安装并启动tor服务后，使用封装好的torify命令发起ssh请求即可。参考[SSH over Tor](https://medium.com/@tzhenghao/how-to-ssh-over-tor-onion-service-c6d06194147)。

2、一个未解疑问：为什么我直接使用iptables转发到ssh隧道端口就不行呢...
