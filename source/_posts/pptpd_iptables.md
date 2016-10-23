---
title: CentOS 7 使用 pptpd 搭建 VPN 的 iptables 配置
date: 2016-10-23 10:41:56
tags: 
    - vpn
    - pptpd
    - linux
categories:
    - 服务端
---

## pptpd设置

主要会操作三个配置文件：
* 主服务配置文件 `/etc/pptpd.conf`
* DNS等选项配置文件 `/etc/ppp/options.pptpd`
* 用户授权配置文件 `/etc/ppp/chap-secrets`

### pptpd服务配置

编辑 `/etc/pptpd.conf` 去掉前面的#去掉：
```
localip 192.168.1
remoteip 192.168.1.234-238,192.168.1.245
```
解释下: localip是pptp使用的ip, 可以随意; remoteip链接到vpn的用户分配到ip的访问, 和localip同一个网段即可.

### DNS设置

编辑 `/etc/ppp/options.pptpd` 去掉ms-dns前面的#，修改成下面的数据：
```bash
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```
解释: 设置链接到vpn的用户如果访问网络时使用的dns, 和他们自己电脑与服务器设置的dns没任何关系.

### VPN账号和密码

编辑 `/etc/ppp/chap-secrets`，直接输入如下字段,vpsma可以换成其他字段，
格式: `用户名 pptpd 密码 IP ` 的形式编写，如果需要多个账号就写多行，一行一个
```
test pptpd 1234 *
```

解释: 这是链接vpn的用户密码, 每行一个, 代表一个用户.
格式说明: 第一列为用户, 依次是 服务器名称, 密码和ip, 中间使用一个空格或者tab隔开.
用户和密码可随意。服务器名(pptpd)不要改,如果修改请保证和`/etc/pptpd.conf`中的`name` 保持一致。后面的*代表ip由pptpd自动分配

### 开启ip转发

编辑 `/etc/sysctl.conf` 文件
```bash
#将“net.ipv4.ip_forward”改为1，开启ip转发。这个不是必须的
net.ipv4.ip_forward=1
#同时注释掉 “net.ipv4.tcp_syncookies = 1” (前面加#)
# net.ipv4.tcp_syncookies = 1

#保存退出`sysctl.conf`文件编辑后，运行下面的命令，能让设置立即生效 
sysctl -p
```

### 配置完成后，重启pptpd服务并设置pptpd开机自启动

```bash
service pptpd start
systemctl enabled pptpd

#启动服务后，可能通过系统日志查看运行情况
tailf /var/log/messages
```

## iptables 配置

### iptables 命令知识

ipdables 配置文件位于 `/etc/sysconfig/iptables`

```bash
#带行号查看当前所有规则 
iptables -L -n --line-numbers
#清除所有规则
iptables -F
#删除指定行号（以下命令中的“5”为指定行号）规则
iptables -D 5
#保存当前配置;相当于旧版/etc/init.d/iptables save
service iptables save
#重启iptables;相当于旧版本/etc/init.d/iptables restart
service iptables restart
#注册iptables服务;相当于旧版 chkconfig iptables on
systemctl enable iptables.service
#开启服务
systemctl start iptables.service
#查看状态
systemctl status iptables.service
```

### 允许连接PPTP服务,1723为pptp服务端口

```bash
iptables -I INPUT -p tcp --dport 1723 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW,RELATED,ESTABLISHED -m tcp --dport 1723 -j ACCEPT
```

### 允许建立VPN隧道,否则无法验证用户名及密码

```
iptables -I INPUT -p gre -j ACCEPT
iptables -A INPUT -p gre -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
```

### 建立NAT转换规则,否则拨上也无法通过远程网关连上公网

```bash
#这里一定看清楚，里面的ip“192.168.1.0/24”要和 /etc/pptpd.conf 的“localip”配置网段对应，还要注意网卡eth0，如果你的网卡不是eth0，就改成你相应的网卡名

#OpenVZ系统用此命令,1.1.1.1为你的VPS的IP地址
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to 1.1.1.1

#XEN系统用这个命令
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
```

### 如果某些网站不能访问，加上

```bash
iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
```


#### 原因分析

linux下pptp搭建的vpn代理上网很慢解决方法。
在pptp所在的linux服务的iptables的*filter表中加入
```bash
-I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
```
或者在命令提示符运行
```bash
/sbin/iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
```

拨通vpn，在服务器上用netstat –i查看接口，得到
```bash
Iface MTU Met RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0 1500 0 102528561 0 0 0 194391413 0 0 0 BRU
eth1 1500 0 519820535 954 11553 924 208798037 0 0 0 BRU
lo 16436 0 151062 0 0 0 151062 0 0 0 LRU
ppp0 1396 0 19 0 0 0 8 0 0 0 OPRU
```

可知ppp的最大mtu为1396，当然，对应的mss应为（mtu-20字节的IP头部+20字节的TCP 头部=）1356
* 1、计算机网络中的MSS：
MSS: Maximum Segment Size 最大分段大小
MSS最大传输大小的缩写，是TCP协议里面的一个概念。

MSS就是TCP数据包每次能够传输的最大数据分段。为了达到最佳的传输效能，TCP协议在建立连接的时候通常要协商双方的MSS值，这个值TCP协议在实现的时候往往用MTU值代替（需要减去IP数据包包头的大小20Bytes和TCP数据段的包头20Bytes）所以往往MSS为1460。通讯双方会根据双方提供的MSS值得最小值确定为这次连接的最大MSS值。

* 2、mtu是网络传输最大报文包。
mss是网络传输数据最大值。mss加包头数据就等于mtu.
简单说拿TCP包做例子。
报文传输1400字节的数据的话，那么mss就是1400，再加上20字节IP包头，20字节tcp包头，那么mtu就是1400+20+20.

当然传输的时候其他的协议还要加些包头在前面，总之mtu就是总的最后发出去的报文大小。mss就是你需要发出去的数据大小。

假设PC建立了到SERVER的HTTP连接，PC希望从SERVER下载一个大的网页。SERVER接收到PC的请求后开始发送大网页文件，其IP的DF位置1，不允许分片，IP报文长度为1500字节。到达VPN网关2的外网口（以太）后，VPN网关2发现其长度超过了1500个字节，于是将其丢弃，并给SERVER发回一个目的地址不可达的ICMP信息，同时指出“MTU of next hop: 1500”。PC接收到该消息后，又按照1500字节对外发送，又被丢弃，于是就形成了循环，无法通讯。

根据上述的分析，很容易得到如下解决方式，在VPN网关2的出接口设置MTU为1500－4－20＝1476，这样VPN网关2返回ICMP不可达消息时将给出”MTU of next hop: 1476”。SERVER将以1476作为自己的最大MTU对外发送，到达VPN网关1，封装GRE和外层IP头后就不会超过1500而顺利发到对端。

```bash
-I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
```

因为mss是在TCP连接建立开始时，通过带有syn标志的IP数据包进行传输的，所以我们在iptables里面规定，在转发数据时，只要发现产生于ppt*的带有 syn标志数据包时，将其mss设定为1356字节，这样就与ppp0接口的路径MTU向匹配了，数据自然就可以畅通无阻啦。
（注，vpn拨入一个，则建立一个ppt*的虚拟设备，这个可以再linux上用ifcpnfig看到，第一个为ppp1，第二个为ppp2……）

* 3、在iptables里面加入一条规则

```bash
iptables -A FORWARD -p tcp --syn -s 10.87.200.0/31 -j TCPMSS --set-mss 1356
```

因为mss是在TCP连接建立开始时，通过带有syn标志的IP数据包进行传输的，所以我们在iptables里面规定，在转发数据时，只要发现带有 syn标志并且源地址为主机B的IP数据包时，将其mss设定为1356字节，这样就与ppp0接口的路径MTU向匹配了，数据自然就可以畅通无阻啦。

因为mss是在TCP连接建立开始时，通过带有syn标志的IP数据包进行传输的，所以我们在iptables里面规定，在转发数据时，只要发现带有 syn标志并且源地址为主机B的IP数据包时，将其mss设定为1356字节，这样就与ppp0接口的路径MTU向匹配了，数据自然就可以畅通无阻啦。


## 此次在实际 CentOS 环境中完整配置规则
```bash
#!/bin/sh
#允许连接PPTP服务
iptables -I INPUT -p tcp --dport 1723 -j ACCEPT
#允许建立VPN隧道以验证用户名密码
iptables -I INPUT -p gre -j ACCEPT
#建立NAT转换规则
iptables -t nat -I POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
#允许pptpd转发
iptables -I FORWARD -i ppp+ -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
#优化网络数据传输速度
iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
echo "完成"
```