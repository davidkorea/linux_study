# DHCP
1. DHCP服务器工作原理
2. 使用DHCP为局域网中的机器分配IP地址
3. 使用DHCP为服务器分配固定IP地址
4. ntpdate加计划任务同步服务器时间

# 1. DHCP服务器工作原理

## 1.1 DHCP服务概述
1. 名称：DHCP  - Dynamic Host Configuration Protocol  动态主机配置协议
2. 功能：DHCP(Dynamic Host Configuration Protocol，动态主机配置协议)是一个局域网的网络协议，使用UDP协议工作， 主要有两个用途：
    - 给内部网络或网络服务供应商自动分配IP地址，主机名，DNS服务器，域名
    - 配合其它服务，实现集成化管理功能。如：无人执守安装服务器

3. 特点 C/S 模式
    - 自动分配IP地址，方便管理
    - DHCP不会同时租借相同的IP地址给两台主机；
    - DHCP管理员可以约束特定的计算机使用特定的IP地址；
    - 可以为每个DHCP作用域设置很多选项；
    - 客户机在不同子网间移动时不需要重新设置IP地址。每次都自动获取IP地址就可以了。
4. DHCP的缺点: 
    - 当网络上存在多服务器时，一个DHCP服务器不能查出已被其它服务器租出去的IP地址；
    - DHCP服务器不能跨路由器与客户机通信，除非路由器允许BOOTP协议转发。
5. 端口：DHCP服务使用端口67(bootps) 68(bootpc) 。

    ```shell
    [root@server162 ~]# vim /etc/services 
       71 bootps          67/tcp                          # BOOTP server
       72 bootps          67/udp
       73 bootpc          68/tcp          dhcpc           # BOOTP client
       74 bootpc          68/udp          dhcpc
    ```
6. DHCP协议由 bootp协议发展而来，是BOOTP的增强版本，bootps代表服务端端口， bootpc代表客户端端口
7. bootp协议：引导程序协议（BOOTP）。它可以让无盘工作站从一个中心服务器上获得IP地址，为局域网中的无盘工作站分配动态IP地址，并不需要每个用户去设置静态IP地址。
8. BOOTP有一个缺点：您在设定前须事先获得客户端的硬件地址，而且，MCA地址与IP的对应是静态的。换而言之，BOOTP非常缺乏“动态性”，若在有限的IP资源环境中，BOOTP的一对一对应会造成非常可观的浪费。
9. DHCP可以说是BOOTP的增强版本，它分为两个部分：一个是服务器端，而另一个是客户端。所有的IP网络设定数据都由DHCP服务器集中管理，并负责处理客户端的DHCP要求；而客户端则会使用从服务器分配下来的IP环境数据。比较BOOTP, DHCP透过“租约”的概念，有效且动态的分配客户端的TCP/IP设定，而且，作为兼容考虑，DHCP也完全照顾了BOOTP Client的需求。

## 1.2. DHCP服务运行原理
![](https://i.loli.net/2019/03/14/5c89ea88a6530.png)

### 1. DHCP Discovery 发现阶段

即DHCP客户端寻找DHCP服务端的过程，对应于客户端发送DHCP Discovery，因为DHCP Server对应于DHCP客户端是未知的，所以DHCP 客户端发出的DHCP Discovery报文是广播包，源地址为0.0.0.0目的地址为255.255.255.255。网络上的所有支持TCP/IP的主机都会收到该DHCP Discovery报文，但是只有DHCP Server会响应该报文。
    
注意：客户端执行DHCP DISCOVER 后，如果没有DHCP 服务器响应客户端的请求，客户端会随机使用169.254.0.0/16 网段中的一个IP 地址配置本机地址。

169.254.0.0/16是windows的自动专有IP寻址范围，也就是在无法通过DHCP获取IP地址时，由系统自动分配的IP地址段。 

早先的Linux上并不会产生这条路由，现在有这条路由大概是为了和windows兼容。
```
[root@xuegod63 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
0.0.0.0         192.168.1.1     0.0.0.0         UG    0      0        0 eth0
```
### 2. DHCP Server 提供阶段
DHCP Server提供阶段，即为DHCP Server响应DHCP Discovery所发的DHCP Offer阶段，即DHCP服务器提供IP地址的阶段。在网络中接收到DHCPdiscover发现信息的DHCP服务器都会做出响应，它从尚未出租的IP地址中挑选一个分配给DHCP客户机，向DHCP客户机发送一个包含出租的IP地址和其他设置的DHCPoffer提供信息 

### 3. DHCP Client 确认阶段
即DHCP客户机选择某台DHCP服务器提供的IP地址的阶段。如果有多台DHCP服务器向DHCP客户机发来的DHCPoffer提供信息，则DHCP客户机只接受第一个收到的DHCPoffer提供信息，然后它就以广播方式回答一个DHCPrequest请求信息，该信息中包含向它所选定的DHCP服务器请求IP地址的内容。之所以要以广播方式回答，是为了通知所有的DHCP服务器，他将选择某台DHCP服务器所提供的IP地址 

### 4. DHCP Server确认阶段
即DHCP服务器确认所提供的IP地址的阶段。当DHCP服务器收到DHCP客户机回答的DHCPrequest请求信息之后，它便向DHCP客户机发送一个包含它所提供的IP地址和其他设置的DHCPack确认信息，告诉DHCP客户机可以使用它所提供的IP地址。然后DHCP客户机便将其TCP/IP协议与网卡绑定，另外，除DHCP客户机选中的服务器外，其他的DHCP服务器都将收回曾提供的IP地址 

### 5. DHCP Client重新登录网络
当DHCP Client重新登录后，就不需要再发送DHCP discover发现信息了，而是直接发送包含前一次所分配的IP地址的DHCP request请求信息。当DHCP服务器收到这一信息后，它会尝试让DHCP客户机继续使用原来的IP地址，并回答一个DHCP ack确认信息。如果此IP地址已无法再分配给原来的DHCP客户机使用时（比如此IP地址已分配给其它DHCP客户机使用），则DHCP服务器给DHCP客户机回答一个DHCP nack否认信息。当原来的DHCP客户机收到此DHCP nack否认信息后，它就必须重新发送DHCP discover发现信息来请求新的IP地址。

### 6. DHCP Client更新租约
DHCP获取到的IP地址都有一个租约，租约过期后，DHCP Server将回收该IP地址，所以如果DHCP Client如果想继续使用该IP地址，则必须更新租约。更新的方式就是，当当前租约期限过了一半后，DHCP Client都会发送DHCP Renew报文来续约租期。

# 2. 使用DHCP为局域网中的机器分配IP地址

- 安装```yum -y install dhcp```
- 主配置文件：/etc/dhcp/dhcpd.conf，DHCP主程序包安装好后会自动生成主配置文件的范本文件/usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample

> 应用案例
> 
> 公司有60 台计算机，IP 地址段为192.168.1.1-192.168.1.254，子网掩码是255.255.255.0，网关为192.168.1.1，192.168.1.2-192.168.1.30 网段地址给服务器配置，客户端可以使用的地址段为192.168.1.100-200，其余剩下的IP 地址为保留地址。
>
> 在路由器上设置 或 wimdows server 上设置 超级简单，鼠标点十几下而已

## 2.1 配置DHCP Server
1. 配置dhcpd.conf文件
```shell
[root@server162 ~]# vim /etc/dhcp/dhcpd.conf 
    #
    # DHCP Server Configuration file.
    #   see /usr/share/doc/dhcp*/dhcpd.conf.example
    #   see dhcpd.conf(5) man page
    #
```
2. 根据文件内提示，复制末班文件来覆盖此文件
```
[root@server162 ~]# cp /usr/share/doc/dhcp*/dhcpd.conf.example /etc/dhcp/dhcpd.conf 
cp: overwrite '/etc/dhcp/dhcpd.conf'? y
```
3. 真正配置时，清空里面全部内容，全部手写下面内容即可
```shell
[root@server162 ~]# cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak     # 清空之前，先备份一下

[root@server162 ~]# > /etc/dhcp/dhcpd.conf 
[root@server162 ~]# vim /etc/dhcp/dhcpd.conf 
    subnet 192.168.1.0 netmask 255.255.255.0 {
            range 192.168.1.100 192.168.1.200;
            option domain-name-servers 192.168.1.1;
            option domain-name "xerox.kr";              # 没有的话，随便配，不影响
            option routers 192.168.1.1;
            option broadcast-address 192.168.1.255;     # 广播一般是这个地址
            default-lease-time 600;
            max-lease-time 7200;                        # 租赁期限可以7天,3个月,99999    
    }
```

4. VM添加网卡vmnet4
```
[root@server162 ~]# ifconfig ens39 down
[root@server162 ~]# ifconfig ens39 up
[root@server162 ~]# ifconfig ens39 192.168.1.10/24      # 给新添加网卡配置192.168.1.*网段的一个ip
[root@server162 ~]# ifconfig ens39                      # 此时并没有ifcfg-ens39网卡配置文件
ens39: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.10  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::7b96:753a:53a:c9b3  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:57:73:3c  txqueuelen 1000  (Ethernet)
        RX packets 2027533  bytes 162150877 (154.6 MiB)
        RX errors 0  dropped 294498  overruns 0  frame 0
        TX packets 126  bytes 18061 (17.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
在做无人值守的时候ftp无法访问192.168.1.10. 检查之前操作，是因为上面只是临时配了192.168.1.10/24这个ip，reboot后就没了ip地址。启动dhcpd服务会报错。

也就是常说的，网卡都没有1网段的ip，dhcp中配置了1网段，服务肯定起不来。自己都不在这个网段，怎么让这个配置文件生效。

所以还是创建ifcfg-ens39网卡配置文件
```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens39"
DEVICE="ens39"
ONBOOT="yes"
IPADDR=192.168.1.10
PREFIX=24
GATEWAY=192.168.1.1
DNS1=114.114.114.114
DNS2=8.8.8.8
```
5. 启动dhcpd服务
```
[root@server162 ~]# systemctl enable dhcpd
Created symlink from /etc/systemd/system/multi-user.target.wants/dhcpd.service to /usr/lib/systemd/system/dhcpd.service.
[root@server162 ~]# systemctl is-enabled dhcpd
enabled
[root@server162 ~]# systemctl start dhcpd
```
注意在写好配置文件之前，不要启动服务。因为网卡目前是192.168.0.162，dhcp要负责sutnet为192.168.1.*，肯定会启动不成功。自己0都不在自己要控制的网段1里面，要再创建一个网卡vmnet4，配置好1网段的ip，再启动写好的配置文件的服务
```
[root@server162 ~]# ps aux | grep dhcp              # 查看启动后的进程

[root@server162 ~]# netstat -nlutp |grep dhcp       # 查看端口，netstat -anutp |grep dhcp结果一样
udp        0      0 0.0.0.0:67       0.0.0.0:*      58847/dhcpd         
```

## 2.2 配置DHCP Client
1. VM添加网卡vmnet4（这个可以随便选），但是IP要和服务器是同一个vmnet下，这样才是同一个网段，开机后才能从dhcp server获取的IP地址。

**其实此时已经获取来192.168.1.100这个网址**，虽然还没有ifcfg-ens38这个网卡配置文件

```
[root@client163 ~]# ls /etc/sysconfig/network-scripts/ifcfg-ens*
/etc/sysconfig/network-scripts/ifcfg-ens33
```
```
[root@client163 ~]# ip a
5: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:ce:03:2f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global noprefixroute dynamic ens38
       valid_lft 135sec preferred_lft 135sec
    inet6 fe80::eff8:e943:6350:7aa5/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
2. 复制一份网卡配置文件
改成dhcp模式，删除ip地址，UUID删不删无所谓
```
[root@client163 ~]# cp /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-ens38
[root@client163 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens38
    TYPE="Ethernet"
    PROXY_METHOD="none"
    BROWSER_ONLY="no"
    BOOTPROTO="dhcp"
    DEFROUTE="yes"
    IPV4_FAILURE_FATAL="no"
    NAME="ens38"
    DEVICE="ens38"
    ONBOOT="yes"
```
3. 重启网卡
```
[root@client163 ~]# ifdown ens38 && ifup ens38

[root@client163 ~]# route -n                            # 查看路由
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.0.1     0.0.0.0         UG    100    0        0 ens33
0.0.0.0         192.168.1.1     0.0.0.0         UG    101    0        0 ens38
192.168.0.0     0.0.0.0         255.255.255.0   U     100    0        0 ens33
192.168.1.0     0.0.0.0         255.255.255.0   U     101    0        0 ens38
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0

[root@client163 ~]# cat /etc/resolv.conf                # 查看防火墙            
# Generated by NetworkManager
search xerox.kr
nameserver 8.8.8.8
nameserver 192.168.1.1
```
## 2.3 回到DHCP服务器，查看租约数据库文件
- 租约数据库文件 相当于 房屋租赁合同
- 租约数据库文件用于保存一系列的租约声明，其中包含客户端的主机名、MAC 地址、分配到的IP地址，以及IP地址的有效期等相关信息。
- 这个数据库文件是可编辑的ASCII 格式文本文件。每当发生租约变化的时候，都会在文件结尾添加新的租约记录。
- DHCP 刚安装好后租约数据库文件dhcpd.leases 是个空文件/var/lib/dhcpd/dhcpd.leases
- 当DHCP 服务正常运行后就可以使用cat 命令查看租约数据库文件内容了
```
[root@server162 ~]# cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.2.5

lease 192.168.1.100 {
  starts 4 2019/03/14 07:55:21;
  ends 4 2019/03/14 08:05:21;
  cltt 4 2019/03/14 07:55:21;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:ce:03:2f;
  client-hostname "client163";
}
server-duid "\000\001\000\001$\034\266\322\000\014)Ws<";

lease 192.168.1.100 {
  starts 4 2019/03/14 07:55:32;
  ends 4 2019/03/14 08:05:32;
  cltt 4 2019/03/14 07:55:32;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:ce:03:2f;
  client-hostname "client163";
}
lease 192.168.1.100 {
  starts 4 2019/03/14 07:55:43;
  ends 4 2019/03/14 08:05:43;
  cltt 4 2019/03/14 07:55:43;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:ce:03:2f;
  client-hostname "client163";
}
```
## 2.4 配置文件/etc/dhcp/dhcpd.conf解释
```shell
# option definitions common to all supported networks...      # 定义全局配置，通用于所有支持的网络选项.

option domain-name "example.org";                             # 为客户端指定所属的域，可填写可不填

option domain-name-servers ns1.example.org, ns2.example.org;  # 为客户端指定DNS服务器地址，配置dhcp ip的同时，可以获取dns

default-lease-time 600;       # 作用：定义默认IP 租约时间，以秒为单位的租约时间。
# 50%:续约。(续不上继续用)
# 87.5%:再次续约。(续不上找别人)
# DHCP工作站除了在开机的时候发出 DHCPrequest 请求之外，
# 在租约期限一半的时候也会发出 DHCPrequest，如果此时得不到 DHCP服务器的确认的话，工作站还可以继续使用该IP；
# 当租约期过了87.5%时，如果客户机仍然无法与当初的DHCP服务器联系上，它将与其它 DHCP服务器通信。
# 如果网络上再没有任何DHCP协议服务器在运行时，该客户机必须停止使用该IP地址，并从发送一个Dhcpdiscover数据包开 始，再一次重复整个过程。
# 要是您想退租，可以随时送出 DHCPRELEASE 命令解约，就算您的租约在前一秒钟才获得的。

max-lease-time 7200;       # 作用：定义客户端IP租约时间的最大值，当客户端超过租约时间，却尚未更新IP 时，最长可以使用该IP 的时间；
# 比如，机器在开机获得IP地址后，然后关机了。这时，当时间过了default-lease-time 600秒后，没有机器向DHCP续约，DHCP会保留7200秒，保留此IP地址不用于分配# 给其它机器。 当超过7200秒后，将不再保留此IP地址给此机器。
# 注意: 2个时间都是以秒为单位的租约时间，该项参数可以作用在全局配置中，也可以作用在局部配置中。

log-facility local7;   #定义日志类型为  local7

subnet： # 声明一般用来指定IP 作用域、定义为客户端分配的IP 地址池等等。声明格式如下：
subnet 声明一个网段 netmask 子网掩码 {
    range ip范围
    dns
    domain name
    option routers 默认网关
    广播地址
    期限 600
    最大期限 7200
}
```


# 3. 使用DHCP为服务器分配固定IP地址

在DHCP 中的IP 地址绑定用于给客户端分配固定IP 地址。比如服务器需要使用固定IP 地址就可以使用IP 地址绑定，通过MAC 地址与IP 地址的对应关系为指定的物理地址计算机分配固定IP地址。整个配置过程需要用到 host 声明和hardware、fixed-address 参数。

**除非dhcp服务器宕机，否则这个ip地址会一值只给这台机器使用**

- host 主机名 {......}作用：用于定义保留地址
- hardware 类型硬件地址作用：定义网络接口类型和硬件地址。常用类型为以太网（ethernet）,地址为MAC 地址。
- fixed-address IP 地址作用：定义DHCP 客户端指定的IP 地址。

## 3.1 DHCP服务器更改配置文件
```
subnet 192.168.1.0 netmask 255.255.255.0 {
        range 192.168.1.100 192.168.1.200;
        option domain-name-servers 192.168.1.1;
        option domain-name "xerox.kr";
        option routers 192.168.1.1;
        option broadcast-address 192.168.1.255;
        default-lease-time 600;
        max-lease-time 7200;
        host client163 {                                # 声明设备hostname，写什么不重要吧？
                hardware ethernet 00:0c:29:ce:03:2f;    # 需要固定ip的设备mac地址
                fixed-address 192.168.1.111;            # 需要固定为的ip
        }
}
```
重启dhcpd服务
```
[root@server162 ~]# systemctl restart dhcpd
```
## 3.2 客户机重启网卡
```
[root@client163 ~]# ifdown ens38 && ifup ens38
Device 'ens38' successfully disconnected.
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/25)
[root@client163 ~]# ifconfig ens38
ens38: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.111  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:fece:32f  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ce:03:2f  txqueuelen 1000  (Ethernet)
        RX packets 26  bytes 8046 (7.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1128  bytes 143987 (140.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

注意：
在生成环境中使用DHCP服务，往往需要结合实际是网络环境来搭建，很多公司采用路由器的DHCP服务来提供IP地址

一般公司会直接再路由器上面配置dhcp，但是测试环境下，需要用到linux配置dhcp服务器

所有的网络设备，业务服务器，都要固定ip

# 4. 时间同步 ntp

## 4.1 ntpdate, ntpd
同步时间 是linux初始化配置中必做的一步，刚装好系统就要配置

- 公网ntp服务器``` ntpdate ntp1.aliyun.com```
- 内网ntp服务器，一般路由器，或者AD服务器 可以作为ntp服务器
### 1. 服务端
只与ntpd服务有关，与ntpdate服务无关
```
[root@server162 ~]# yum install ntp -y
[root@server162 ~]# systemctl enable ntpd
[root@server162 ~]# systemctl start ntpd
```
配置文件，不用更改也可直接用
```
[root@server162 ~]# vim /etc/ntp.conf 
```
### 2. 客户端
```
[root@client163 ~]# ntpdate 192.168.0.162
14 Mar 17:22:33 ntpdate[2605]: adjust time server 192.168.0.162 offset 0.000375 sec
```

但这样的同步，只是强制性的将系统时间设置为ntp服务器时间。只是治标不治本。所以，一般配合cron命令，来进行定期同步设置。在crontab中添加： 
```0 12 *  * * /usr/sbin/ntpdate 192.168.0.1 ```

## 4.2 让NTP服务同步硬件时间
ntp服务，默认只会同步系统时间。如果想要让ntp同时同步硬件时间，可以设置/etc/sysconfig/ntpdate 文件。 
在/etc/sysconfig/ntpdate 文件中，把SYNC_HWCLOCK=no 改成SYNC_HWCLOCK=yes
就可以让硬件时间与系统时间一起同步。

## 4.3 linux系统时间和BIOS时间是不是一定一样？
```
hwclock -r    :读出BIOS的时间参数
hwclock -w    :将当前系统时间写入BIOS中。
```






