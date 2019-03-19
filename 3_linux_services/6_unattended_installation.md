# 搭建无人执守安装服务器 - 批量安装
1. 搭建无人执守安装服务器服务器常见概念
2. 搭建无人执守安装服务器服务器安装及相关配置文件
3. 实战：为公司内网搭建一个搭建无人执守安装服务器

# 1. 搭建无人执守安装服务器服务器常见概念
- 需要使用到的服务：PXE + DHCP + TFTP + Kickstart+ FTP/HTTP

- 安装三四十台机器，一般就需要无人值守安装了
- 新购买服务器 - 上架 - RAID - 重启，快捷键，从网卡启动 - DHCP server, 三个服务安装在一台服务器上
<p align="center">
  <img src="https://i.loli.net/2019/03/18/5c8f1368ec875.png" width=300, height=300>
</p>

- 安装的时候，最开始，通过快捷键进入网络引导模式。不建议更改第一启动位置。因为安装好系统，重启还会再次通过网络引导安装系统

#### 1. PXE原理和概念：  
- 严格来说，PXE 并不是一种安装方式，而是一种引导的方式。
- 进行 PXE 安装的必要条件是要安装的计算机中包含一个 PXE 支持的网卡（NIC），即网卡中必须要有 PXE Client。PXE （Pre-boot Execution Environment）协议使计算机可以通过网络启动。协议分为 client 和 server 端，PXE client 在网卡的 ROM 中，当计算机引导时，BIOS 把 PXE client 调入内存执行，由 PXE client 将放置在远端的文件通过网络下载到本地运行。现在网卡都支持。
- 运行 PXE 协议需要设置 DHCP 服务器 和 TFTP 服务器。DHCP 服务器用来给 PXE client（将要安装系统的主机）分配一个 IP 地址，由于是给 PXE client 分配 IP 地址，所以在配置 DHCP 服务器时需要增加相应的 PXE 设置。此外，在 PXE client 的 ROM 中，已经存在了 TFTP Client。PXE Client 通过 TFTP 协议到 TFTP Server 上下载所需的文件。

#### 2. 什么是KickStart 
- KickStart是一种无人职守安装方式。KickStart的工作原理是通过记录典型的安装过程中所需人工干预填写的各种参数，并生成一个名为 ks.cfg的文件
- 在其后的安装过程中（不只局限于生成KickStart安装文件的机器）当出现要求填写参数的情况时，安装程序会首先去查找 KickStart生成的文件，当找到合适的参数时，就采用找到的参数，当没有找到合适的参数时，才需要安装者手工干预。
- 如果KickStart文件涵盖了安装过程中出现的所有需要填写的参数时，安装者完全可以只告诉安装程序从何处取ks.cfg文件，然后去忙自己的事情。等安装完毕，安装程序会根据ks.cfg中设置的重启选项来重启系统，并结束安装。


#### 3. 执行 PXE + KickStart安装需要准备内容：
- DHCP 服务器用来给客户机分配IP
- TFTP 服务器用来存放PXE的相关文件，比如：系统引导文件
- FTP 服务器用来存放系统安装文件，需要挂载光盘镜像
- KickStart所生成的ks.cfg配置文件；
- 带有一个 PXE 支持网卡的将安装的主机

# 2. 搭建无人执守安装服务器服务器安装及相关配置文件

### 1. 配置yum基本环境
```
[root@server162~]# mount  /dev/cdrom  /mnt
[root@server162~]# vi /etc/yum.repos.d/serverl.repo #在/etc/yum.repos.d目录下创建以.repo结尾的文件
```
### 2. 安装ftp服务以及开启服务，设置为开机自动启动
- 纯净系统
```
[root@server162 ~]# yum install vsftpd -y
[root@server162 ~]# systemctl start vsftpd
[root@server162 ~]# systemctl enable vsftpd
```
- 之前配置国ftp的清空，更改vsftp设置
  - 实现匿名可以直接访问pub
  - team1 11111 依然可以向以前一样通过filezilla进行有账户访问/var/www/html
```
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf 

 12 anonymous_enable=YES
 29 anon_upload_enable=YES
 33 anon_mkdir_write_enable=YES
""" 并没有改动
103 local_root=/var/www/html
104 chroot_list_enable=YES
105 # (default follows)
106 chroot_list_file=/etc/vsftpd/chroot_list
107 allow_writeable_chroot=YES
"""
"""
120 ssl_enable=NO         # 只是禁用来这一行
121 #ssl_enable=YES
122 allow_anon_ssl=NO
123 force_local_data_ssl=YES
124 force_local_logins_ssl=YES
125 force_anon_logins_ssl=YES
126 force_anon_data_ssl=YES
127 ssl_tlsv1=YES
128 ssl_sslv2=YES
129 ssl_sslv3=YES
130 require_ssl_reuse=NO
131 ssl_ciphers=HIGH
132 rsa_cert_file=/etc/vsftpd/.sslkey/vsftpd.pem
133 rsa_private_key_file=/etc/vsftpd/.sslkey/vsftpd.pem
"""
```
### 3. 安装TFTP,修改tftp配置文件及开启服务
```
[root@server162 ~]# yum install tftp tftp-server xinetd -y
```
  修改第13，14行
```
[root@server162 ~]# vim /etc/xinetd.d/tftp 

  1 # default: off
  2 # description: The tftp server serves files using the trivial file transfer \
  3 #       protocol.  The tftp protocol is often used to boot diskless \
  4 #       workstations, download configuration files to network-aware printers, \
  5 #       and to start the installation process for some operating systems.
  6 service tftp
  7 {
  8         socket_type             = dgram
  9         protocol                = udp
 10         wait                    = yes
 11         user                    = root
 12         server                  = /usr/sbin/in.tftpd
 13         server_args             = -s /tftpboot
 14         disable                 = no
 15         per_source              = 11
 16         cps                     = 100 2
 17         flags                   = IPv4
 
[root@server162 ~]# systemctl start xinetd          # 启动服务
[root@server162 ~]# lsof -i:69
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
xinetd  9102 root    5u  IPv4  51445      0t0  UDP *:tftp 
```

- ```server_args = -s /tftpboot```: 是tftp服务器运行时的参数。-s /tftpboot表示服务器默认的目录是 /tftpboot,当执行put a.txt时，文件会被放到服务器的/tftpboot/a.txt，省去你敲put a /tftpboot/的麻烦。你也可以加其它服务器运行参数到这，具体可以执行man tftpd命令查阅。

- 参数-c: 上传文件时，服务器上没有。就自动创建这个文件。默认tftp客户端，只能上传tftp服务器已经有的文件。也就是只能传上去并覆盖服务器上的原文件。如果想上传原来目录中没有的文件，需要修改tftp服务器的配置文件并重起服务。需要修改如下：```server_args = -s /tftpboot -c```

  TFTP (Trivial File Transfer Protocol)，中译简单文件传输协议或小型文件传输协议. 大家一定记得在2003年8月12日全球爆发冲击波（Worm.Blaster）病毒，这种病毒会监听端口69,模拟出一个TFTP服务器，并启动一个攻 击传播线程,不断地随机生成攻击地址，进行入侵。另外tftp被认为是一种不安全的协议而将其关闭，同时也是防火墙打击的对象，这也是有道理的。tftp 在嵌入式linux还是有用武之地的。需要打开防火墙，允许tftp访问网络。