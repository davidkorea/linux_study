# 字符界面安装 KVM-阿里云主机使用方法 

1. 在命令行安装 KVM 虚拟机 
2. 在命令行无人执守安装 KVM 虚拟机 
3. 阿里云主机使用方法 

# 1. 在命令行安装 KVM 虚拟机 
#### 1. 上传centos7 iso镜像到/var/lib/libvirt/images/
创建并挂载sdb1分区至/var/lib/libvirt/images/，并拷贝iso镜像至该目录
```
[root@localhost ~]# mount /dev/sdb1 /var/lib/libvirt/images/
```
#### 2. 安装 vnc 客户端软件
方便后期进程连接正在安装中的虚拟机的安装界面
```
[root@localhost ~]# yum install tigervnc -y       # vnc 进程桌面客户端 
[root@localhost ~]# yum install virt-viewer -y    # 后面安装虚拟机时，需要使用 
```
扩展: vmware 迁移到 kvm ，需要使用 virt-v2v ,安装: ```yum install virt-v2v```
#### 3. 确认 libvirtd 服务开启
```
[root@localhost ~]# systemctl status libvirtd
```
#### 4. 通过命令行安装 KVM 虚拟机
- --name=NAME 挃定 Guest 名字
- --ram=MEMORY 挃定内存大小
- --vcpus=VCPUS 挃定虚拟机的 CPU 数量
- --disk path 指定虚拟机磁盘存储文件的路径, size=5 指定虚拟磁盘的大小，单位是G。例:--disk path=/var/lib/libvirt/images/centos-72.img,size=5
- --cdrom=CDROM 挃定用于全虚拟化 Guest 的虚拟光驱， --cdrom=后指定 ISO 或 CDROM 镜像。
- --network 指定虚拟机的网卡模式。如:--network bridge=br0
- -x EXTRA, --extra-args=EXTRA 用来给加载的 kernel 和 initrd 提供额外的内核命令行参数。比 如无人值守安装系统

```
virt-install --name centos-7 --ram 1024 --vcpus=1 --disk path=/var/lib/libvirt/images/centos-72.img,size=5 --accelerate --cdrom /var/lib/libvirt/images/CentOS-7.4-x86_64-DVD.iso --network bridge=br0 --graphics vnc
```
  - 可以通过```virt-viewer```或者```vncviewer 127.0.0.1```实时查看安装进度
  - 若使用xshell执行该命令，会自动通过xstart连接到图形界面


# 2. 使用 ks 文件，无交互字符界面安装

### 1. 搭建本地yum源
### 2. 搭建http
```
[root@localhost ~]# yum install httpd -y 
[root@localhost ~]# systemctl start httpd                     # 目录/var/www/html/会自动生成
[root@localhost ~]# mkdir /var/www/html/centos7/
[root@localhost ~]# mount /dev/cdrom /var/www/html/centos7/   # 挂载光盘至该http目录

[root@localhost ~]# iptables -F #关闭防火墙 访问http://192.168.0.162/centos7/
```
### 3. 上传 ks.cfg 文件到/var/www/html/ 下
修改 ks.cfg 文件
```
[root@xuegod63 html]# vim ks.cfg
改:8 url --url="ftp://192.168.0.162/pub" 
为:8 url --url="http://192.168.0.162/centos7"
```

### 4. 安装虚拟机
注: 你局域网中的 DHCP 服务器自劢分配的 IP 地址是: 192.168.0.0/24 网段才行。 丌然需要自 己搭建一个 DHCP，这样虚拟机安装过程中才可以一个 192.168.0.0 网段的 IP，才可以访问到 http://192.168.0.162/centos7/

因为ks.cfg文件生成到时候，默认网络设置就是DHCP，所以不用自己额外搭建，就用实际路由器到DHCP就可以分配到和宿主机一样到网段

- 参考1: [搭建DHCP](https://github.com/davidkorea/linux_study/blob/master/3_linux_services/3_DHCP.md)
- 参考2: [搭建无人执守安装服务器](https://github.com/davidkorea/linux_study/blob/master/3_linux_services/6_unattended_installation.md)


#### 1. 报错空间不足
```shell
virt-install --name centos7_ks --ram 1024 --vcpus=1 --disk path=/var/lib/libvirt/images/centos7_ks.qcow2,size=13 --accelerate --location=http://192.168.0.162/centos7/ --network bridge=br0 -x "ks=http://192.168.0.162/ks.cfg"
```
![](https://i.loli.net/2019/03/19/5c90bb97c31ab.png)



#### 2. 正常安装，扩大内存至2G，硬盘位20G
```shell
virt-install --name kvm_centos7_ks --ram 2048 --vcpus=1 --disk path=/var/lib/libvirt/images/kvm_centos7_ks.qcow2,size=20 --accelerate --location=http://192.168.0.163/centos7/ --network bridge=br0 -x "ks=http://192.168.0.163/ks.cfg"
```
使用的ks，cfg文件如下
```shell
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --iscrypted $1$ylWwkW5v$Zpe1fUpFalYQ79qQ4VhxO1
# Use network installation
url --url="http://192.168.0.163/centos7"
# System language
lang en_US
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use graphical install
graphical
firstboot --disable
# SELinux configuration
selinux --disabled

# Firewall configuration
firewall --disabled
# Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="xfs" --size=300
part swap --fstype="swap" --size=2000
part / --fstype="xfs" --grow --size=1

%packages
@additional-devel
@development
@platform-devel

%end
```
