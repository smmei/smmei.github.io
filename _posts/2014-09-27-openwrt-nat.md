---
layout: post
title: "iptables配置NAT"
category: tools
---

## NAT

假设一个局域网络有5~10台机器，把它们的默认网关设置成NAT服务器。所有的报文发送给网关，NAT处理这些报文。

NAT接收到一个报文，然后改写它的源/目的地址，再重新计算packet的checksum。

* SNAT
假设这几台机器所在的本地网段是192.168.1.0/24，SNAT会将所有192.168.1.0地址转换成其公网地址217.115.95.34。

* DNAT

---

* iptables-save [-c] [-t table]

-c 保存的时候保存包和字节数量
-t 指定保存哪一个表

* iptables-restore [-c] [-n]


iptables [-t table] command [match] [target/jump]
* -t table 指定一张表，mangle/nat/raw/filter, 默认是filter表
* command 告诉iptable我们想干什么，例如增加或删除一条规则
* match 描述包的特点，用来匹配特定的包。这里我们可以指定包的来源IP地址，网络接口，端口，协议类型等
* target 这里指定了报文要执行的操作。例如把报文发到指定的链上，或丢掉这个报文，或回复一个特定的报文

### table

* nat
* mangle
* filter
* raw

### command

* -A, --append 在链尾添加一条规则

	iptables -A INPUT ...

* -D, --delete 删除一条规则，可以指定完整规则或序号来删除

	iptables -D INPUT --dport 80 -j DROP
	iptables -D INPUT 1

* -R, --replace 替换指定位置的旧规则。与delete工作方式一样，只不过不是删除而是替换

	iptables -R INPUT 1 -s 192.168.0.1 -j DROP

* -I, --insert 在指定的地方插入一条规则

	iptables -I INPUT 1 --dport 80 -j ACCEPT

这里的1就指定插入到第一个位置，成为第一条规则

* -L, --list 列出指定链的所有规则

	iptables -L INPUT

没有指定表，就列出filter表中INPUT链的所有规则

* -F, --flush 删除链上的所有规则

	iptables -F INPUT

* -Z, --zero 计数器清零

	iptables -Z INPUT

* -N, --new-chain 创建一条新链

	iptables -N allowed

在filter上创建了一条叫作allowed的链

* -X, --delete-chain 删除一条链

	iptables -X allowed

* -P, --policy 为链设置默认策略，所有不符合规则的包都强制使用这个策略

	iptables -P INPUT DROP

* -E, rename-chain 修改链的名字

	iptables -E allowed disallowed

### options

### match

#### 通用匹配
#### TCP matches
#### UDP matches
#### ICMP matches
#### 特殊匹配
---



# openwrt 拨号上网

我的一块mt7620n路由器，刷了openwrt，拨号成功，路由器中可以访问外网。PC连接到路由器的LAN上，通过192.168.1.1地址可以访问到路由器的web配置界面，但是PC无法连上外网。

在折腾的过程中，学习到一点东西。

## MTU

[解决openwrt无线上网问题](http://blog.chinaunix.net/uid-108863-id-3076334.html)

我查看另一台正常路由器的MTU设置的是1480。openwrt默认的是1500。

PPPOE拨号时，因为PPPOE协议要占掉8节字，所以MTU最大设置为1492。

	iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1400:1536 -j TCPMSS --clamp-mss-to-pmtu

`--tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu`来动态选择最大MTU。


## NAT

[Network Address Translation](http://zh.wikipedia.org/zh-cn/%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2)

在IP包通过路由器或防火墙时重写源IP地址或目的IP地址的技术。NAT可以使多台主机通过一个公有IP地址访问外网。

iptables可以实现NAT功能。

## iptables NAT配置

prerouting -> forward -> postrouting

* prerouting 先检查是不是本地网段的地址。
* 如果不是，就开始查找路由表，找到后经过forward转发
* postrouting时进行NAT转换

NAT转换发生在postrouting阶段。

### SNAT

源地址改变，目的地址不变

### DNAT

源地址不变，目的的址改变。将一个IP的端口转换成另一个IP的另外一个端口号。

openwrt中

	iptables -t nat -A POSTROUTING -i br-lan -o pppoe-wan -j MASQUERADE


转载一篇文章
[OpenWrt路由器iptables防火墙自行手工设置示例](http://blog.appdevp.com/archives/282)

```
OpenWrt默认安装自带了iptables防火墙，并且默认设置了不少规则和策略，尤其是自定义了很多用户规则链，看起来比较复杂。
用iptables -nL 查看，会发现特别多自定义用户链，
root@myopenwrt:~# iptables -nL
Chain INPUT (policy ACCEPT)
target prot opt source destination
bw_ingress all — 0.0.0.0/0 0.0.0.0/0
ACCEPT all — 0.0.0.0/0 0.0.0.0/0
ACCEPT all — 0.0.0.0/0 0.0.0.0/0 ctstate RELATED,ESTABLISHED
ACCEPT all — 0.0.0.0/0 0.0.0.0/0
input_rule all — 0.0.0.0/0 0.0.0.0/0
input all — 0.0.0.0/0 0.0.0.0/0

Chain FORWARD (policy DROP)
target prot opt source destination
bw_ingress all — 0.0.0.0/0 0.0.0.0/0
ingress_restrictions all — 0.0.0.0/0 0.0.0.0/0
egress_restrictions all — 0.0.0.0/0 0.0.0.0/0
ACCEPT all — 0.0.0.0/0 0.0.0.0/0
ACCEPT all — 0.0.0.0/0 0.0.0.0/0
ACCEPT all — 0.0.0.0/0 0.0.0.0/0 ctstate RELATED,ESTABLISHED
forwarding_rule all — 0.0.0.0/0 0.0.0.0/0
forward all — 0.0.0.0/0 0.0.0.0/0
reject all — 0.0.0.0/0 0.0.0.0/0

Chain OUTPUT (policy ACCEPT)
target prot opt source destination
ACCEPT all — 0.0.0.0/0 0.0.0.0/0
ACCEPT all — 0.0.0.0/0 0.0.0.0/0 ctstate RELATED,ESTABLISHED
ACCEPT all — 0.0.0.0/0 0.0.0.0/0
output_rule all — 0.0.0.0/0 0.0.0.0/0
output all — 0.0.0.0/0 0.0.0.0/0

Chain bw_ingress (2 references)
target prot opt source destination
all — 0.0.0.0/0 0.0.0.0/0 bandwidth –id total1-download-2-449 –type combined –current_bandwidth 0 –reset_interval 2 –reset_time 2 –intervals_to_save 449
all — 0.0.0.0/0 0.0.0.0/0 match-set local_addr_set dst bandwidth –id bdist1-download-minute-15 –type individual_dst –reset_interval minute –intervals_to_save 15
all — 0.0.0.0/0 0.0.0.0/0 bandwidth –id total2-download-minute-359 –type combined –current_bandwidth 0 –reset_interval minute –intervals_to_save 359
all — 0.0.0.0/0 0.0.0.0/0 match-set local_addr_set dst bandwidth –id bdist2-download-900-24 –type individual_dst –reset_interval 900 –reset_time 900 –intervals_to_save 24
all — 0.0.0.0/0 0.0.0.0/0 bandwidth –id total3-download-180-479 –type combined –current_bandwidth 0 –reset_interval 180 –reset_time 180 –intervals_to_save 479
all — 0.0.0.0/0 0.0.0.0/0 match-set local_addr_set dst bandwidth –id bdist3-download-hour-24 –type individual_dst –reset_interval hour –intervals_to_save 24
all — 0.0.0.0/0 0.0.0.0/0 bandwidth –id total4-download-7200-359 –type combined –current_bandwidth 0 –reset_interval 7200 –reset_time 7200 –intervals_to_save 359
all — 0.0.0.0/0 0.0.0.0/0 match-set local_addr_set dst bandwidth –id bdist4-download-day-31 –type individual_dst –reset_interval day –intervals_to_save 31
all — 0.0.0.0/0 0.0.0.0/0 bandwidth –id total5-download-day-365 –type combined –current_bandwidth 0 –reset_interval day –intervals_to_save 365
all — 0.0.0.0/0 0.0.0.0/0 match-set local_addr_set dst bandwidth –id bdist5-download-month-12 –type individual_dst –reset_interval month –intervals_to_save 12

Chain egress_restrictions (1 references)
target prot opt source destination
egress_whitelist all — 0.0.0.0/0 0.0.0.0/0

Chain egress_whitelist (1 references)
target prot opt source destination

Chain forward (1 references)
target prot opt source destination

Chain forwarding_rule (1 references)
target prot opt source destination

Chain ingress_restrictions (1 references)
target prot opt source destination
ingress_whitelist all — 0.0.0.0/0 0.0.0.0/0

Chain ingress_whitelist (1 references)
target prot opt source destination

Chain input (1 references)
target prot opt source destination

Chain input_rule (1 references)
target prot opt source destination

Chain output (1 references)
target prot opt source destination

Chain output_rule (1 references)
target prot opt source destination

Chain pf_loopback_B (0 references)
target prot opt source destination

Chain reject (1 references)
target prot opt source destination
REJECT tcp — 0.0.0.0/0 0.0.0.0/0 reject-with tcp-reset
REJECT all — 0.0.0.0/0 0.0.0.0/0 reject-with icmp-port-unreachable

——————————————————————————————————-
上面的显示结果，是不是看着就有点懵了！建议直接清空好了，完全可以根据自己的需要重新配置iptables，可满足一般用户需求。
下面我们来一步一步设置，完全根据自身需要进行定制。

root@myopenwrt:~#vi /etc/firewall.user
用户自定义的防火墙规则可以存在/etc/firewall.user文件中，实际就是iptables具体的设置命令。

##首先清空系统默认规则（filter表）
iptables -F #删除所有规则链中的所有规则
iptables -X #删除用户自定义规则链
iptables -Z #计数清零

#设置各规则链的默认策略。
iptables -P INPUT   DROP
iptables -P OUTPUT  ACCEPT
iptables -P FORWARD DROP

iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i br-lan -j ACCEPT
iptables -A INPUT -m state –state RELATED,ESTABLISHED -j ACCEPT

iptables -A FORWARD -i br-lan -o pppoe-wan  -j ACCEPT
iptables -A FORWARD -m state –state RELATED,ESTABLISHED -j ACCEPT
#iptables -A FORWARD -p tcp –dport 10080 -j ACCEPT
#iptables -A FORWARD -p udp –dport 10080 -j ACCEPT

##清空系统默认规则（nat表）
iptables -t nat -F
iptables -t nat -X
iptables -t nat -Z

iptables -t nat -P PREROUTING  ACCEPT
iptables -t nat -P POSTROUTING ACCEPT
iptables -t nat -P OUTPUT      ACCEPT

##pppoe拨号网络支持nat
iptables -t nat -A POSTROUTING -i br-lan -o pppoe-wan -j MASQUERADE
#iptables -t nat -A PREROUTING -p tcp  –dport 10080  -j DNAT –to-destination 10.10.7.2
#iptables -t nat -A PREROUTING -p udp  –dport 10080  -j DNAT –to-destination 10.10.7.3

##清空系统默认规则（mangle表）
#iptables -t mangle -F
#iptables -t mangle -X
#iptables -t mangle -Z
#iptables -t mangle -A PREROUTING  -i pppoe-wan -j TTL –ttl-inc 1
#iptables -t mangle -A POSTROUTING -o pppoe-wan -j TTL –ttl-set 128
#iptables -t mangle -A POSTROUTING -o pppoe-wan -j IPID –ipid-pace 1
#iptables -I FORWARD -p tcp –tcp-flags RST RST -j DROP
```
