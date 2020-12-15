# 作业2+17组+NAT

## 实验说明

NAT network address translation，将IP数据包中的源地址或目的地址进行翻译转换的一项技术，主要被用来将内部地址转换为公共或internet地址。

本次实验使用静态NAT配置，应用一对一静态映射，帮助同学了解学习NAT的基本使用和应用场景及理念，以便进一步学习动态NAT复用和基于NAT的TCP负载分担等高级应用。

## 网络拓扑

> 拓扑中的接口为 Cisco Packet 模拟器中的接口，实际配置中需要参考使用的设备情况修改对应命令。
>
> 学院机房电脑的配置下，f0/0口为f0/0，f1/0口为f0/1，s2/0口为s0/0/0。

![静态NAT网络拓扑](.\静态NAT-telnet.jpg)

网络拓扑由三部分组成：

* 配置网络环境：PC0，Switch0；（网段：**172.16.0.0 /24**）
* 外部网络环境，模拟Internet网络环境：PC1，RTA；（网段：**192.168.1.0 /30**，**192.168.3.0 /24**）
* 内部网络环境，模拟公司内部网络环境：PC2，RTB，RTC，Switch1。（内部网段：**10.0.0.0 /8**，给定地址块网段：**192.168.1.32 /27**）

## 实验目标

本次实验通过在pc0上运行项目，在web页面上通过telnet对RTA、RTB、RTC三个路由器进行配置。

本次实验的目标是在路由器RTB上应用NAT技术，将内部网络环境的PC2主机和路由器RTC映射到给定的地址块（**192.168.1.32 /27**），从而实现以下功能：

1. 位于外部网络环境的PC1主机无法访问到具体内部网络（**10.0.0.0 /8**）；
2. 位于外部网络环境的PC1主机可以通过分配给内部网络的外部地址块（**192.168.1.32 /27**）访问到位于内部网络环境的pc2主机；
3. 位于内部网络环境的PC2主机可以访问到内部的其他路由（RTC）；
4. 位于内部网络环境的PC2主机可以访问到位于外部网络的PC1主机。

## 0. 初始化

本节对主机、路由器进行相关手动配置，包括主机的网络设置、路由器重命名和关闭命令查找功能、以及配置路由器以支持telnet设置。

* 因为主机模拟的是实际环境中用户的使用主机，所以不使用telnet进行配置；
* 此次实验并不需要对交换机进行配置，所以未启用交换机的telnet配置功能；
* 三台路由器的telnet网段为 **172.16.0.0 /24**，telnet密码和enable密码均为**123456**。

### 0.1. 配置主机的网络设置

#### PC0

* IP Address 172.16.0.1
* Subnet Mask 255.255.255.0

#### PC1

* IP Address 192.168.3.2
* Subnet Mask 255.255.255.0
* Gateway 192.168.3.1

#### PC2

* IP Address 10.0.0.11
* Subnet Mask 255.0.0.0
* Gateway 10.0.0.1

### 0.2. 路由器重命名和移除命令查找功能

* 使用重命名便于区分路由器；
* 移除命令查找结果错误输入命令时长时间等待的问题。

#### RTA

~~~
enable
configure terminal
hostname RTA
no ip domain-lookup
exit
exit
~~~

#### RTB

~~~
enable
configure terminal
hostname RTB
no ip domain-lookup
exit
exit
~~~

#### RTC

~~~
enable
configure terminal
hostname RTC
no ip domain-lookup
exit
exit
~~~

### 0.3. 配置路由器telnet

对RTA、RTB、RTC三台路由器进行配置，以支持PC0上通过telnet对三台路由器进行访问，从而进行进一步的相关配置。

* 除了telnet之外还需要配置enable的密码以支持修改设置。

#### RTA

~~~
enable
configure terminal
interface f1/0
ip address 172.16.0.2 255.255.255.0
no shutdown
exit
line vty 0 15
password 123456
login
exit
enable password 123456
exit
exit
~~~

#### RTB

~~~
enable
configure terminal
interface f1/0
ip address 172.16.0.3 255.255.255.0
no shutdown
exit
line vty 0 15
password 123456
login
exit
enable password 123456
exit
exit
~~~

#### RTC

~~~
enable
configure terminal
interface f1/0
ip address 172.16.0.4 255.255.255.0
no shutdown
exit
line vty 0 15
password 123456
login
exit
enable password 123456
exit
exit
~~~

## 1. 配置路由器接口

本节在PC0上通过telnet对三台路由器进行相关配置，并对配置完成的网络连接使用ping命令进行检查，为后续步骤配置NAT做好准备。

### 在PC0上配置RTA接口

~~~
telnet 172.16.0.2
// 密码 123456
enable
// 密码 123456
configure terminal
interface f0/0
ip address 192.168.3.1 255.255.255.0
no shutdown
interface s2/0
ip address 192.168.1.1 255.255.255.252
no shutdown
clock rate 9600
exit
exit
exit
~~~

### 在PC0上配置RTB接口

~~~
telnet 172.16.0.3
// 密码 123456
enable
// 密码 123456
configure terminal
interface f0/0
ip address 10.0.0.1 255.0.0.0
no shutdown
interface s2/0
ip address 192.168.1.2 255.255.255.252
no shutdown
clock rate 9600
exit
exit
exit
~~~

### 在PC0上配置RTC接口

~~~
telnet 172.16.0.4
// 密码 123456
enable
// 密码 123456
configure terminal
interface f0/0
ip address 10.0.0.2 255.0.0.0
no shutdown
exit
exit
exit
~~~

### 检查

此时，网络拓扑已基本链接，各路由器和主机应当可以ping通各自直连的接口，而无法ping通非直连的接口。如：

* PC1可以ping通192.168.3.1和192.168.1.1，但无法ping通192.168.1.2和10.0.0.11；
* PC2可以ping通10.0.0.1,10.0.0.2和192.168.1.2，但无法ping通192.168.1.1和192.168.3.2。

## 2. 配置数据流转发

因为存在外部网络环境和内部网络环境两个自知系统，所以不在它们之间启用路由协议，本节即进行数据流转发的相关配置。

### 在PC0上配置RTA

为RTA配置一条到达子网 **192.168.1.32/27** 的静态路由，这是分配给内部网络的地址块网段。

~~~
telnet 172.16.0.2
// 密码 123456
enable
// 密码 123456
configure terminal
ip route 192.168.1.32 255.255.255.224 192.168.1.2
exit
exit
~~~

### 在PC0上配置RTB

配置RTB，将所有的非本地数据流转发到RTA。

~~~
telnet 172.16.0.3
// 密码 123456
enable
// 密码 123456
configure terminal
ip route 0.0.0.0 0.0.0.0 192.168.1.1
exit
exit
~~~

### 在PC0上配置RTC

因为不在 **10.0.0.0 /8** 网段上运行路由选择协议，所以配置RTC使用一条到RTA的缺省路由

~~~
telnet 172.16.0.4
// 密码 123456
enable
// 密码 123456
configure terminal
ip route 0.0.0.0 0.0.0.0 10.0.0.1
exit
exit
~~~

### 检查

此时，在上一轮检查的基础上应当有以下变化：

* RTB应当可以ping通以下接口：
  * PC1（192.168.3.2）；
  * PC2（10.0.0.11）；
  * RTA s2/0口（192.168.1.1）；
  * RTA f0/0口（192.168.3.1）；
  * RTC f0/0口（10.0.0.2）。
* RTA应当无法ping通RTC的f0/0口；
* RTC应当无法ping通RTA的s2/0口；
* PC1应当可以ping通192.168.1.2。

## 3. 配置静态NAT

本节将通过telnet对RTB路由器进行配置，从而为内部网络环境的PC2主机和RTC路由器配置静态NAT转换。

### 在PC0上配置RTB

~~~
telnet 172.16.0.3
// 密码 123456
enable
// 密码 123456
configure terminal
ip nat inside source static 10.0.0.2 192.168.1.34
ip nat inside source static 10.0.0.11 192.168.1.35
interface f0/0
ip nat inside
interface s2/0
ip nat outside
exit
exit
exit
~~~

### 检查

此时，静态NAT的相关配置已经全部完成：

* 位于外部网络环境的PC1主机依然无法ping通具体内部网络（**10.0.0.0 /8**），但可以ping通192.168.1.34，可以访问到内部网络环境；

* 位于内部网络环境的PC2主机可以ping通192.168.3.1，可以访问到外部网络环境；

* 在RTB上检查NAT转换，应当有192.168.1.35到10.0.0.11和192.168.1.34到10.0.0.2的记录。

    ~~~
    telnet 172.16.0.3
    // 密码 123456
    enable
    // 密码 123456
    show ip nat translations
    exit
    ~~~

## 4. 测试用例

### RTA

1. ping 10.0.0.2【失败】；
2. ping 10.0.0.11【失败】；
3. ping 192.168.1.34【成功】；
4. ping 192.168.1.35【成功】。

### RTB

1. ping 192.168.1.1【成功】；
2. ping 192.168.3.2【成功】；
3. ping 10.0.0.2【成功】；
4. ping 10.0.0.11【成功】；
5. 检查NAT转换（show ip nat translations），应当有192.168.1.35到10.0.0.11和192.168.1.34到10.0.0.2的记录。

### RTC

1. ping 10.0.0.1【成功】；
2. ping 10.0.0.11【成功】；
3. ping 192.168.1.1【成功】；
4. ping192.168.3.2【成功】。
