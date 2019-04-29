---
title: 搭建私有Tor网络
date: 2019-04-29 16:49:56
categories: 
- [Tor]
- [网络]
tags: 
- Tor
- 网络
- 匿名
images: "/images/post2019/Tor-logo.png"
---

Tor网络由遍布世界各地的Tor节点组成，用户通过Tor发送的数据包在这些节点间加密传输，以达到匿名的目的。为了了解Tor的机制，我搭建了一个私有Tor网络，包含1个目录服务器，3个节点（都可作为出口节点），1个客户端，实现客户机通过私有Tor浏览网页。

（私有Tor网络匿名性显著下降，仅适用于实验学习）


<!-- more -->

---


# 实验网络拓扑



![](/images/post2019/privateTor.jpg)

Client：客户机
DA：Tor目录服务器
Guard、Middle、Exit：Tor中继或出口节点
以上所有机器搭载 Ubuntu 18.04

<br/>
# 安装Tor

`# sudo apt install tor`

其他系统安装参考[官方安装指南](https://2019.www.torproject.org/docs/installguide.html.en)。

<br/>
# 配置目录服务器DA

Tor权威目录服务器（directory authority server）DA提供当前网络中的Tor节点信息，包括节点的地址、公钥等。当一个Tor节点接入网络时，首先要向DA获取活跃节点信息，再从中随机选择三个组成链路（circuit）。

首先，生成DA密钥和证书：

```shell
sudo -u debian-tor mkdir /var/lib/tor/keys
sudo -u debian-tor tor-gencert --create-identity-key -m 12 -a 1.1.1.1:7000 \
-i /var/lib/tor/keys/authority_identity_key \
-s /var/lib/tor/keys/authority_signing_key \
-c /var/lib/tor/keys/authority_certificate  
```

在ubuntu上使用用户名`debian-tor`，在其他系统如centOS上请根据实际更改为`/var/lib/tor`目录的所有者。

以上命令在` /var/lib/tor/keys`目录下为地址为`1.1.1.1`的DA生成的证书和私钥，并开放`7000`端口，其中

`authority_identity_key`：主身份密钥，用于验证其签名密钥的有效性，应离线保存。

`authority_signing_key`：签名密钥，用于签署投票和共识（consensuses）。

`authority_certificate`：DA的证书，用于验证签名密钥。

接下来，生成节点密钥和指纹：

```
sudo -u debian-tor tor --list-fingerprint \
--dirserver "name 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" \
--datadirectory /var/lib/tor/  
```

然后，编辑`/etc/tor/torrc`文件：

```
TestingTorNetwork 1

# 这些配置是为了快速上线
# 避开一些对稳定性的限制
AssumeReachable 1
TestingDirAuthVoteExit *
TestingDirAuthVoteHSDir *
TestingDirAuthVoteGuard *
V3AuthNIntervalsValid 2
TestingMinExitFlagThreshold 0


DataDirectory /var/lib/tor
RunAsDaemon 1
ConnLimit 60
Nickname DA
ShutdownWaitLength 0
PidFile /var/lib/tor/pid
Log notice file /var/lib/tor/notice.log
Log info file /var/lib/tor/info.log
Log debug file /var/lib/tor/debug.log 
ProtocolWarnings 1
SafeLogging 0
DisableDebuggerAttachment 0

# 指定自己的目录服务器
DirAuthority DA orport=5000 no-v2 v3ident=finger1 1.1.1.1:7000 finger2

SocksPort 0
# OrPort监听Tor连接
OrPort 5000
# DirPort监听目录服务
DirPort 7000

NickName DA
Address 1.1.1.1

# 目录服务器
AuthoritativeDirectory 1
V3AuthoritativeDirectory 1
ContactInfo auth0@test.test

# 不作为出口
ExitPolicy reject *:*

# Vote + Dist必须小于1/2 * Interval
TestingV3AuthInitialVotingInterval 300
TestingV3AuthInitialVoteDelay 20
TestingV3AuthInitialDistDelay 20

V3AuthVotingInterval 300
V3AuthVoteDelay 20
V3AuthDistDelay 20
```

其中finger1：

`
cat /var/lib/tor/keys/authority_certificate | grep fingerprint
`

finger2:

 `/var/lib/tor/fingerprint`

再启动tor服务就可以了。


# 配置节点服务器

先停止Tor服务，然后



```
sudo -u debian-tor tor --list-fingerprint --orport 1 --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" --datadirectory /var/lib/tor/
```

配置torrc:

```
TestingTorNetwork 1

AssumeReachable 1
TestingDirAuthVoteExit *
TestingDirAuthVoteHSDir *
V3AuthNIntervalsValid 2
TestingDirAuthVoteGuard *
TestingMinExitFlagThreshold 0

DataDirectory /var/lib/tor
RunAsDaemon 1
ConnLimit 60
Nickname relay3
ShutdownWaitLength 0

PidFile /var/lib/tor/pid
Log notice file /var/lib/tor/notice.log
Log info file /var/lib/tor/info.log
Log debug file /var/lib/tor/debug.log

ProtocolWarnings 1
SafeLogging 0
DisableDebuggerAttachment 0

# 指定目录服务器为DA
DirAuthority DA orport=5000 no-v2 v3ident=finger1 1.1.1.1:7000 finger2

SocksPort 0
OrPort 5000
ControlPort 9051

NickName relayN
Address 2.2.2.2

# 作为出口节点
ExitRelay 1
```

重启Tor服务。

# 配置客户端

同样，先停止Tor服务，然后生成指纹：

`sudo -u debian-tor tor --list-fingerprint --orport 1 --dirserver "x 127.0.0.1:1 ffffffffffffffffffffffffffffffffffffffff" --datadirectory /var/lib/tor/  `

配置Torrc文件：


```

TestingTorNetwork 1

AssumeReachable 1
TestingDirAuthVoteExit *
TestingDirAuthVoteHSDir *
V3AuthNIntervalsValid 2
TestingDirAuthVoteGuard *
TestingMinExitFlagThreshold 0

DataDirectory /var/lib/tor
RunAsDaemon 1
ConnLimit 60
Nickname relay3
ShutdownWaitLength 0

PidFile /var/lib/tor/pid
Log notice file /var/lib/tor/notice.log
Log info file /var/lib/tor/info.log
Log debug file /var/lib/tor/debug.log

ProtocolWarnings 1
SafeLogging 0
DisableDebuggerAttachment 0

# 指定目录服务器为DA
DirAuthority DA orport=5000 no-v2 v3ident=finger1 1.1.1.1:7000 finger2

# 设置Socks端口
SocksPort 9050
ControlPort 9051

NickName client
Address x.x.x.x

```

最后重启Tor服务。

# 测试私有Tor网络

设置浏览器或系统代理为 127.0.0.1:9050 socks5，然后访问网页，使用Wireshark等抓包工具观察，或直接访问[ipinfo.io](www.ipinfo.io)，就可以看到Tor网络的作用啦。

如果遇到问题，可以查看torrc文件中指定位置的notice.log、info.log等日志文件。

# 可能遇到的部分问题

## [err] Error initializing keys; exiting.

在生成keys报错：

```
[err] Another Tor process has locked "/var/lib/tor/keys/secret_id_key". Not writing any new keys.
[err] Error initializing keys; exiting.
```

这个问题非常好解决，先`service tor stop`就可以了。同样的解决方法还适用于报错部分文件被“lock”的情况。

## [WARN] Failed to find node for hop 0 of our path. Discarding this circuit.

如果notice.log中连续出现此告警，可以尝试校准系统时间。特别是目录服务器上。

`sudo ntpdate -v pool.ntp.org`

## Consensus not signed by sufficient number of requested authorities

或者`Not enough good signatures on networkstatus consensus` 之类的关于共识签名的错误。

我个人得到这个报错有两次：一次是torrc文件中，对目录服务器的fingerprint配置有误；另一次是我尝试让一个节点连接两台私有目录服务器，但这两台服务器彼此之前不知道，也无法为对方签署的共识签名。

## not able to authenticate relays

我的某一个客户端始终无法成功与Tor节点握手成功，查看info.log发现大量类似信息：

```
Apr 23 19:52:29.000 [info] **choose_good_exit_server_general(): Chose exit server '$1DDF03E9C34867C581F0DE8D6EAEC30A9188DB11~relay2 at 149.129.x.x'
Apr 23 19:52:29.000 [info] extend_info_from_node(): Not including the ed25519 ID for $1DDF03E9C34867C581F0DE8D6EAEC30A9188DB11~relay2 at 149.129.x.x, since it won't be able to authenticate it.**

```

原因是这台客户端机器在国内，所以...

最后解决办法是用一个不在墙内的服务器S做Tor客户端，同时是WireGuard服务器。国内的机器通过WireGuard连接到S，然后由S转发流量到Tor网络。


# 参考


1. [Tor Manual](https://2019.www.torproject.org/docs/tor-manual.html.en)
2. [Chutney](https://github.com/torproject/chutney) （参考它生成的torrc文件）
2. [Anonymous Routing of Network Traffic Using Tor](https://witestlab.poly.edu/blog/anonymous-routing-of-network-traffic-using-tor/)
3. [How to Setup Private Tor Network](http://ju.outofmemory.cn/entry/175626)





