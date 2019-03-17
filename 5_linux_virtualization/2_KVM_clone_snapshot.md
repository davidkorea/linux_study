# 第二章 KVM 虚拟机克隆和快照

1. KVM 虚拟机克隆方法
2. 虚拟机常用镜像格式对比
3. KVM 虚拟机快照功能使用方法
4. virsh 命令常见用法
5. KVM 常用镜像格式转换

# 1. KVM 虚拟机克隆

### 1.1 KVM虚拟机镜像

虚拟机镜像: 就是整个虚拟机文件。 不是操作系统光盘镜像 rhel6.5.iso
```
[root@localhost ~]# cd /var/lib/libvirt/images/
[root@localhost images]# ls
CentOS-7-x86_64-DVD-1810.iso  kvm_centos7.qcow2

[root@localhost images]# ll -h
总用量 15G
-rw-r--r-- 1 qemu qemu 4.3G 2月  20 00:04 CentOS-7-x86_64-DVD-1810.iso
-rw------- 1 qemu qemu  11G 3月  17 22:19 kvm_centos7.qcow2
```

### 1.2 基于 centos7.0 克隆一台虚拟机

1. 克隆前，centos7.0 需要提前关机
2. 语法:virt-clone -o 原虚拟机 -n 新虚拟机 -f 新虚拟机镜像存放路径 选项:-o old -n new
```
[root@localhost ~]# virt-clone -o kvm_centos7 -n clone_kvm_centos7 -f /var/lib/libvirt/images/clone_kvm_centos7.img
正在分配 'clone_kvm_ce 18% [===              ]  43 MB/s | 1.9 GB  03:14 ETA 
```
3. 查看克隆后到镜像文件大小
```
[root@localhost images]# ll -h
总用量 12G
-rw------- 1 root root 1.3G 3月  17 22:42 clone_kvm_centos7.img    # 只有1.3G
-rwx------ 1 root root  11G 3月  17 22:39 kvm_centos7.qcow2
```
### 1.3 KVM 虚拟机组成
一台 KVM 虚拟机由两部分组成:虚拟机配置文件和镜像img
1. 查看配置文件
```
[root@localhost ~]# ll /etc/libvirt/qemu
总用量 16
drwxr-xr-x  2 root root   29 3月  17 17:33 autostart
-rw-------  1 root root 4465 3月  17 22:42 clone_kvm_centos7.xml
-rw-------  1 root root 4449 3月  17 17:15 kvm_centos7.xml
drwx------. 3 root root   42 1月  30 02:34 networks
```
2. 查看远虚拟机和克隆虚拟机配置文件到差别
```
[root@localhost ~]# cd /etc/libvirt/qemu/
[root@localhost qemu]# ls
autostart  clone_kvm_centos7.xml  kvm_centos7.xml  networks
[root@localhost qemu]# vimdiff kvm_centos7.xml clone_kvm_centos7.xml 
还有 2 个文件等待编辑
```
![](https://i.loli.net/2019/03/17/5c8e50c73b6c1.png)

### 1.4 测试克隆机器
打开后，可以正常联网，不用做额外操作
![](https://i.loli.net/2019/03/17/5c8e52eb5f097.png)

之前到版本，开启后查看到到MAC地址和xml配置文件到MAC并不一致，需要修改
- 在 rhel6 下 kvm 克隆后的操作
  - 登录新克隆的虚拟机删除原来的 mac 和 IP 地址，让新克隆的机器可以上网
    ```
    [root@xuegod63 ~]# rm -rf /etc/udev/rules.d/70-persistent-*
    [root@xuegod63 ~]#vim /etc/sysconfig/network-scripts/ifcfg-eth0 #写入以下内容
    ```
    ![](https://i.loli.net/2019/03/17/5c8e53eb90c5e.png)
    注: 记得把 ONBOOT="no" 改为: ONBOOT="yes"
    注: 把原配置文件中的 MAC 和 UUID 地址删除，然后修改一个和原虚拟机不一样的 IP，reboot #重启生效
  - 方法 2
    ```
    [root@xuegod63 ~]# start_udev # 重新启劢 udev 服务，自劢生成刚删除的/etc/udev/rules.d/70-persistent-*文件
                                  # 新生成的 udev 文件，会使用新系统的 MAC 地址。 
    [root@xuegod63 ~]# service network restart
    ```