# DNS服务器搭建

## 基本概念

一些基本概念的介绍：
- 顶级域：Top-level domains（TLD），域名设计的初期，顶级域名分为两类，一类为国家及地区双字代码顶级域（Country Code Top-level domains，ccTLD）和通用顶级域（general Top-level domains，gTLD）。2011年开始，ICANN开始批准新的TLD命名，在该年之后被批准的通用顶级域名被称为新通用顶级域名（New general Top-level domains，NewgTLD）。
- 全限定域名：Fully Qualified Domain Name（FQDN），同时带有主机名和域名的名称。FQDN可以从逻辑上准确地表示出主机的位置，所以说全域名是主机名的一种完全表示形式。

学习本文除了要求对以上一些概念的了解外，你还需要知道：
- Linux服务器的IP、DNS、Gateway的基本概念和配置方法
- `iptabls`基础知识

## DNS查询与解析

DNS查询方式既有递归操作，也有迭代操作。
递归查询是客户端发起一次请求给DNS服务器，通过多次查找返回正确解析。
迭代是发出多次请求查询不同的DNS服务器。

DNS解析包括正向解析和反向解析：正向解析就是将FDQN转换为IP地址，而反向解析则是相反的转换。

DNS监听TCP和UDP都是53端口。

DNS记录：
- SOA记录：起始授权机构记录
- NS记录
- A记录
- MX记录
- AAAA记录
- PTR记录
- CNAME：

## DNS服务器搭建实验

### 实验构想
DNS服务器S0同时也为网络提供地址转换服务（作为子网的网关），它的一块网卡链接到VMnet0（桥接网络），另一块网卡链接到VMnet1（仅主机网络），在VMnet1网络中有两台静态IP服务器，服务器S1运行了一个HTTP网站，服务器S2作为本实验的客户端机器。

我们首先保证在VMnet1当中，S2可以访问到S1的HTTP服务；然后，我们在VMnet1中的S0当中添加网关服务，保证S1、S2可以访问外网；最后，我们配置正确的DNS服务，将S2请求的特定FQDN解析为S1的网络服务。

### 实验条件

- 实验平台：openSUSE 15.2 Leap
- 虚拟机平台：VMware Workstation 15.7
- 虚拟机操作系统：Ubuntu 20.04 LTS Server
- 虚拟硬件设置：
  - 处理器：2个单核心CPU
  - 内存：1000MB
  - 硬盘：5GB

### 1. HTTP服务配置

首先，我们建设在VMnet1中可访问的HTTP服务。
假设此时我们刚将S1、S2的网卡修改为仅主机模式。

在S1当中，使用Python3内置`http.server`提供简单http服务:
```bash
sudo python3 -m http.server 80
```
> 注：因为curl命令默认访问80端口，所以如果此处不标明80端口，在后续的命令中需要指定端口。

为了访问此服务，我们需要获取到S1服务器的IPv4地址，使用命令：
```bash
ip address
```

在S2当中，尝试使用`curl`命令访问S1，在我的实验当中，S1的IPv4地址为`172.16.26.130`：
```bash
curl 172.16.26.130
```
如果S2是可访问的，那么curl会很快返回一个网页的内容，否则该命令将会长时间不返回结果指导超时退出。

如果出现无法访问的情况，请使用`ip address`检查S1和S2是否在同一IPv4子网，如果出现IP地址上的问题，可以通过以下一些思路配置网络地址：
- 如果VMnet1启用了dhcp，可以考虑使用`dhclient`获取IPv4地址
- 考虑使用`ip address add`或者`ip address del`进行网卡IPv4地址的管理

如果需要重复开关S1、S2进行测试，它们的网卡ip地址可能发生变化。如果希望实验易复现，可以考虑使用netplan组件对网卡进行自动静态IP配置。

本文采用的是采用netplan对网络进行配置。

#### netplan配置虚拟机静态IP

在正式配置之前，我们为VMnet1配置一个易识别的子网IP，本文的配置是`1.0.0.0/24`。在VMware Workstation的任务栏中点击Edit -> Virtual Network Editor -> VMnet1，输入相应配置即可。

而后我们可以编写一个`/etc/netplan/01-static-vmnet1-addr.yaml`（命名自选）:
```yaml
network:
    version: 2
    ethernets:
        ens37:
            dhcp4: false
            addresses:
                - 1.0.0.10/24
            nameservers:
                addresses:
                    - 1.0.0.10
            gateway4: 1.0.0.10
```
配置中，`ens37`为仅主机网络虚拟网卡名称。注意此时在VMnet1当中并没有有效的网关和DNS服务器，所以这里的`gateway4`和`nameservers`配置仅仅是用做修改的模板，我们真正关注的是`dhcp4`和`addresses`的设置。

在完成配置之后，使用`netplan apply`使配置生效。

-----

在解决了IP问题之后，我们可以正常从S2访问到S1的简易HTTP服务。

### 2. 配置网关服务器

网关服务器的主要职责是对通过的流量做地址翻译，从而实现对内外网沟通的支持。
硬件层面，网关服务器至少需要两块网卡以连通两个网络。
软件层面，数据转发工作可以通过`iptables`组件来完成。

具体的配置工作可以参考以下脚本内容：
```bash
#!/bin/bash

WAN_IF=ens33
LAN_IF=ens37

# 使能内核转发
echo 1 > /proc/sys/net/ipv4/ip_forward

# 关闭Ubuntu内置ufw防火墙
systemctl stop ufw.service

# 清除所有规则
iptables -F
iptables -t nat -F
iptables -t mangle -F
iptables -X

# 允许所有已建立的连接和LAN连接
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i $LAN_IF -m state --state NEW -j ACCEPT

# 配置NAT和转发
## 从WAN向LAN的转发
iptables -A FORWARD -i $WAN_IF -o $LAN_IF -m state --state ESTABLISHED,RELATED -j ACCEPT
## 从LAN向WAN的转发
iptables -A FORWARD -i $LAN_IF -o $WAN_IF -j ACCEPT
## 配置NAT(二选一)
iptables -t nat -A POSTROUTING -o $WAN_IF -j MASQUERADE
#iptables -t nat -A POSTROUTING -o $WAN_IF -j SNAT --to-source 192.168.2.3
```
其中`ESTABLISHED,RELATED`状态，表示仅允许已建立连接的回包；最后一条SNAT规则中，`--to-source 192.168.2.3`表示将数据包源LAN地址，修改为网关外网地址，也就是S0在VMnet0当中的网络地址，本例中为`192.168.2.3`。伪装链`MASQUERADE`则能够自动完成这个操作，对于网关外网IP可变的情况，推荐使用这种方式。

运行完以上命令序列之后，尝试在S2当中访问外网：
```bash
ping 223.5.5.5
```
可以发现，网络已经连通了。如果无法从S2当中访问外网，除检查以上配置之外，还需要确保S0网关服务器本身能够连通外网，比如在S0当中编写恰当的`netplan`配置文件等。

### 3. 配置DNS服务器

首先安装`bind`服务，在Ubuntu网关服务器内：
```bash
sudo apt install bind9
```

# 参考

[域名-Wikipedia](https://zh.wikipedia.org/zh-hans/%E5%9F%9F%E5%90%8D)
[Linux 服务器做网关](https://blog.csdn.net/JackLiu16/article/details/80155558)
[iptables 快速入门](https://blog.firemiles.top/2019/iptables%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/)