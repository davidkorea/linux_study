
# 1. xl 常用命令
- create
- destroy
- shutdown，虚拟机需要可以接收shutdown命令，busybox是不可以的
- reboot，重启
- pci
  - pci-list
  - pci-attach 热插拔
  - pci-detach
- 暂停
  - pause，暂停主机，定格在内存中，和vmware的挂起不一样。宿主机关机，虚拟机的未保存信息全部会丢失
  - unpause，解除暂停
- 挂起
  - save，挂起，将dom0内存中的数据转存至磁盘文件中。磁盘文件可以自行制定，还需要指定创建时的配置文件，这是因为这个busybox虚拟机的内核不在其本身的磁盘镜像中，而是在dom0中
    - ```xl save busybox-001 /tmp/busybox.img /etc/xen/busybox_conf```
  - restore，从指定的磁盘中恢复  
    - ```xl restore /etc/xen/busybox_conf /tmp/busybox.img```
    ```
    [root@localhost ~]# xl save busybox-002 /tmp/busybox          # 并需需要制定启动配置文件
    Saving to /tmp/busybox new xl format (info 0x3/0x0/1138)
    xc: info: Saving domain 8, type x86 PV
    xc: Frames: 65536/65536  100%
    xc: End of stream: 0/0    0%
    [root@localhost ~]# ls /tmp/ 
    busybox        
    [root@localhost ~]# xl li
    Name                                        ID   Mem VCPUs      State   Time(s)
    Domain-0                                     0  3271     4     r-----    5814.8
    busybox-001                                  7   256     2     -b----      84.9
    
    [root@localhost ~]# xl restore /tmp/busybox                   # 并需需要制定启动配置文件
    Loading new save file /tmp/busybox (new xl fmt info 0x3/0x0/1138)
     Savefile contains xl domain config in JSON format
    Parsing config from <saved>
    xc: info: Found x86 PV domain from Xen 4.8
    xc: info: Restoring domain
    xc: info: Restore successful
    xc: info: XenStore: mfn 0x4c800, dom 0, evt 1
    xc: info: Console: mfn 0x4c7ff, dom 0, evt 2
    [root@localhost ~]# xl li
    Name                                        ID   Mem VCPUs      State   Time(s)
    Domain-0                                     0  3271     4     r-----    5820.5
    busybox-001                                  7   256     2     -b----      85.0
    busybox-002                                  9   256     2     -b----       0.1
    ```
- migrate
- cdrom
  - cd-insert
  - cd-eject 弹出光驱
- mem-max
- men-set
- vcpu
  - vcpu-list
    - ```xl vcpu-list busybox-001```
  - vcpu-pin
   - ```xl vcpu-pin busybox-001 0 3```，虚拟机vcpu的第0个核心，固定在物理cpu的第三个核心
  - vcpu-set，指定使用几个vcpu核心，需要少于创建虚拟机时给定的vcpu个数
    ```
    [root@localhost ~]# xl vcpu-list busybox-002
    Name                                ID  VCPU   CPU State   Time(s) Affinity (Hard / Soft)
    busybox-002                          9     0    3   -b-       0.3  all / all
    busybox-002                          9     1    2   -b-       0.3  all / all
    [root@localhost ~]# xl vcpu-pin busybox-002 0 1       # 虚拟机vcpu0核心绑定至物理cpu1核心
    [root@localhost ~]# xl vcpu-list busybox-002
    Name                                ID  VCPU   CPU State   Time(s) Affinity (Hard / Soft)
    busybox-002                          9     0    1   -b-       0.4  1 / all
    busybox-002                          9     1    2   -b-       0.3  all / all
    [root@localhost ~]# xl vcpu-set busybox-002 1         # 指定虚拟机仅使用1个vcpu
    [root@localhost ~]# xl vcpu-list busybox-002
    Name                                ID  VCPU   CPU State   Time(s) Affinity (Hard / Soft)
    busybox-002                          9     0    1   -b-       0.6  1 / all
    busybox-002                          9     1    -   --p       0.4  all / all
    ```
- info，查看xen-hypervisor的信息
- domid
- domname
- dmesg，显示虚拟机启动时，引导时的信息
- top，显示各虚拟机资源占用情况
- network
  - network-list，查看现有虚拟机的网络接口
  - network-attach，网卡热插，添加新网卡
    - ```x network-attach busybox-001 bridge=xenbr1```，宿主机上会显示vif1.0和vif1.1两个网卡
  - network-detach
    - ```x network-detach busybox-001 1```，后面的1时网卡设备的idx，需要使用network-list命令来查看
- 磁盘 
> **测试发现，并不支持qcow2磁盘格式，raw热插成功**
  - 创建新磁盘镜像，qcow2可以创建快照，而raw不可以
    - ```qemu-img create -f qcow2 -o ? /images/xen/busybox1.2.img```，？问号可以查看可使用参数
    - ```qemu-img create -f qcow2 -o size=5G,prealloation=metadata /images/xen/busybox1.2.qcow2```，只预创建出metadata
  - block-list，现有磁盘设备
  - block-attach
    - ```xl block-attach busybox-001 '/images/xen/busybox1.2.img,qcow2,xvdb,w'```
    - 进入虚拟机```fdish -l```查看全部磁盘，```fdisk  /dev/xvdb```可以进行磁盘分区
  - block-datach
    - ```xl block-attach busybox-001 51728```，磁盘idx需用通过block-list来查看
    ```
    [root@localhost ~]# qemu-img create -f raw -o size=1G /images/xen/busybox3.img
    Formatting '/images/xen/busybox3.img', fmt=raw size=1073741824 
    
    [root@localhost ~]# xl block-attach busybox-001 '/images/xen/busybox3.img,raw,xvdc,rw'
    [root@localhost ~]# xl block-list busybox-001
    Vdev  BE  handle state evt-ch ring-ref BE-path                       
    51712 0   7   4    13    8        /local/domain/0/backend/vbd/7/51712
    51728 0   7   3    15    770      /local/domain/0/backend/qdisk/7/51728   # qcow2格式，在虚拟机中fdisk无法显示
    51744 0   7   4    16    771      /local/domain/0/backend/vbd/7/51744
    [root@localhost ~]# xl console busybox-001
    blkfront: xvdc: flush diskcache: enabled; persistent grants: enabled; indirect descriptors: enabled;
     xvdc: unknown partition table

    / # fdisk -l

    Disk /dev/xvda: 2147 MB, 2147483648 bytes
    255 heads, 63 sectors/track, 261 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

    Disk /dev/xvda doesn't contain a valid partition table

    Disk /dev/xvdc: 1073 MB, 1073741824 bytes
    255 heads, 63 sectors/track, 130 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

    Disk /dev/xvdc doesn't contain a valid partition table
    ```
    ```
    [root@localhost ~]# xl block-list busybox-001
    Vdev  BE  handle state evt-ch ring-ref BE-path                       
    51712 0   7      4     13     8        /local/domain/0/backend/vbd/7/51712
    51728 0   7      3     15     770      /local/domain/0/backend/qdisk/7/51728
    51744 0   7      4     16     771      /local/domain/0/backend/vbd/7/51744
    [root@localhost ~]# xl block-detach busybox-001 51744
    [root@localhost ~]# xl block-detach busybox-001 51728   # 删除qcow2磁盘时，也会报错，实际上已经删除
    libxl: error: libxl_device.c:1086:device_backend_callback: unable to remove device with path /local/domain/0/backend/qdisk/7/51728
    libxl: error: libxl.c:2008:device_addrm_aocomplete: unable to remove vbd with id 51728
    libxl_device_disk_remove failed.
    
    [root@localhost ~]# xl block-list busybox-001
    Vdev  BE  handle state evt-ch ring-ref BE-path                       
    51712 0   7      4     13     8        /local/domain/0/backend/vbd/7/51712
    ```
- uptime，查看运行时长

# 2. 使用domU自有kernel启动domU
## 2.1 创建镜像文件
- ```qemu-img create -f qcow2 -o size=5G,preallocation=metadata /images/xen/busybox3.img```
## 2.2 磁盘分区，格式化
- 分区
  - boot，kernel，initrd
  - 根文件系统
- 关联到本地回环设备losetup，/dev目录下的loop1-loop7都可以当作本地回环设备
  - 其实 mount -o loop 命令就会自动将loop0-7关联至本地回环设备，比如/images/xen/busybox.img
  - 也可以通过losetup来手动实现
  - losetup -a 显示所有已用loop设备
  - losetup -f 显示第一个可用loop设备
  
#### 1. 关联loop设备和磁盘镜像文件
- ```losetup /dev/loop1 /images/xen/busybox3.img```
#### 2. 对loop1分区
- ```fdisk /dev/loop1```，注意下面分区时的+加号
  - partition 1 = +50M
  - partition 2 = +1G
  - w
- ```kpartx -av /dev/loop1```，显示loop1的分区loop1p1和loop1p2
  - ```ls /dev/mapper```下面可以看到分区路径loop1p1和loop1p2，可以直接cd进行访问
#### 3. 格式化分区
- ```mke2fs -t ext2 /dev/mapper/loop1p1```
- ```mke2fs -t ext2 /dev/mapper/loop1p2```
此时可以按照分区进行分别挂载，并拷贝相应文件至相应分区
#### 4. 挂载分区
- 在/mnt下为每个分区创建一个挂载目录
  - ```mkdir /mnt{boot,sysroot}```
- ```mount /dev/mapper/loop1p1 /mnt/boot/```
- ```mount /dev/mapper/loop1p2 /mnt/sysroot/```
## 2.3 拷贝内核文件至分区目录
#### 1. boot目录
- ```cp /boot/vmlinuz-2.6.32 /mnt/boot/vmlinux```
- ```cp /boot/initramfs-2.6.32.img /mnt/boot/initramfs.img```
- 安装grub
  - ```grub-install --root-directory=/mnt /dev/loop1```，此处指定磁盘，而不是分区，命令会自动在/mnt下找到boot目录，把grub分拣安装进去
  - 会提示no corresponding BIOS drive，但无所谓，查看ls /mnt/boot/，会出现grub目录，而且该grub目录下会生成一些文件
  - 在/mnt/boot/grub/目录下创建grub.conf文件
    ```
    default=0
    timeout=5
    title BusyBox(kernel-2.6.32)
      root (hd0,0)
      kernel /vmlinuz root=/dev/xvda1 ro selinux=0 init=/bin/sh
      initrd /initramfs.img
    ```
#### 2. sysroot目录
- ```cp -a /root/busybox-1.23.0/_install/* /mnt/sysroot/```
- ```cd /mnt/sysroot```
  - ```mkdir -pv lib/modules etc dev var proc sys tmp```
  - ```cp /lib/modules/2.6.32/kernel/drivers/net/xen-netfront.ko lib/modules``` 
- sync，同步一下

#### 3. 拆除loop设备
- ```umount /mnt/boot```，umount关闭访问映射路径
- ```umount /mnt/sysroot```
- ```kpartx -d /dev/loop1```
- ```losetup -d /dev/loop1```，上一步执行之后，losetup -a应该就已经查看不到loop设备了

## 2.4 准备虚拟机配置文件
- ```cp /etc/xen/busybox_conf busybox_conf3```
- vim busybox_conf3
  ```
  name=busybox-003
  #kernel
  #ramdisk
  #extra
  vif = ['bridge=xenbr0']
  disk = ['/images/xen/busybox3.img,qcow2,xvda,rw']
  #root
  bootloader = '/usr/bin/pygrub'
  builder = ‘hvm'
  ```
  - pygrub会当作mdr的第一个来引导，它会自动读取设备上的第二个，即/images/xen/busybox3.img

## 2.5 创建虚拟机
- xl -v create /etc/xen/busybox_conf3，失败
  - 改配置文件，最后加一行 ```builder = ‘hvm’```，也还是失败
  - qcow2格式 不支持！！！！！
## 2.6 重新再创建一个raw的磁盘镜像  

- ```qemu-img create -f raw -o size=2G /images/xen/busybox3.img```
- 剩下步骤与上面一直
- 创建好惊现之后，测试raw格式的磁盘是否可以被正在运行的虚拟机busybox-001识别
  - xl block-attach busybox-001 '/imagex/xen/busybox3.img,raw,xvdb,w'
  - xl console busybox-001
  - fdisk -l，发现可以识别额到xvdb设备
  - mount /dev/xvdb /etc，ls /etc查看磁盘挂载成功，umount拆掉即可
  - xl block-detach busybox-001 51728，拆掉磁盘，磁盘设备idx需要通过block-list查看

> 手动制作镜像并创建虚拟机操作完成
> - 但是测试时，创建完成吸进，xl list查看状态是p，而不是b或者r
> - xl console无法进入虚拟机，报错busybox-003 is an invalid domain identifier (rc=-6)


























