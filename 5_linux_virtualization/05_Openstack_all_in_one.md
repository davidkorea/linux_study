# openstack-allinone-使用方法

1. 安装 OpenStack 客户端并创建一个cirros demo1云主机
2. 查看创建好的 openstack 项目中的信息和云主机网络连通性
3. openstack web 界面使用方法



# 1. 安装 OpenStack 客户端并创建一个cirros demo1云主机
## 1.1 安装 OpenStack client 端
安装 OpenStack client 端，方便后期使用命令行操作 openstack

#### 1. pip install python-openstackclient
```
[root@server15 ~]# pip install python-openstackclient 
Found existing installation: PyYAML 3.10
Cannot uninstall 'PyYAML'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.

[root@server15 ~]# pip install PyYAML --ignore-installed PyYAML

[root@server15 ~]# pip install python-openstackclient        # 再次安装
Found existing installation: ipaddress 1.0.16
Cannot uninstall 'ipaddress'. It is a distutils installed project and thus we cannot
accurately determine which files belong to it which would lead to only a partial uninstall. 

[root@server15 ~]# pip install ipaddress --ignore-installed ipaddress 

[root@server15 ~]# pip install python-openstackclient        # 再次安装 ok
```
#### 2. pip install python-neutronclient
安装 openstack 网络相关的命令
```
[root@server15 ~]# pip install python-neutronclient 
[root@server15 ~]# pip install pyinotify --ignore-installed pyinotify 
[root@server15 ~]# pip install python-neutronclient         # 再次安装成功。
```
## 1.2 使用 init-runonce 脚本创建一个 openstack 云项目
#### 1. 修改 init-runonce 脚本
修改 init-runonce 脚本，指定浮劢 IP 地址范围，浮劢 IP 就是云主机的公网 IP。init-runonce 是在 openstack 中快速创建一个云项目例子的脚本。

```diff
[root@server15 ~]# vim /usr/share/kolla-ansible/init-runonce

- 12 EXT_NET_CIDR='10.0.2.0/24'
- 13 EXT_NET_RANGE='start=10.0.2.150,end=10.0.2.199' 
- 14 EXT_NET_GATEWAY='10.0.2.1'
 
+ EXT_NET_CIDR='192.168.0.0/24' 
+ EXT_NET_RANGE='start=192.168.0.100,end=192.168.0.200' 
+ EXT_NET_GATEWAY='192.168.0.1'
```
#### 2. 使用 init-runonce 脚本创建一个 openstack 云项目
```
[root@server15 ~]# source /etc/kolla/admin-openrc.sh  # 先加载这个文件，把文件中的环境变量加入系统中，才有权限执行下面的命令
[root@server15 ~]# cd /usr/share/kolla-ansible 
[root@server15 kolla-ansible]# ./init-runonce         # 最后弹出以下命令，复制后直接执行即可

Done.

To deploy a demo instance, run:

openstack server create \
    --image cirros \
    --flavor m1.tiny \
    --key-name mykey \
    --nic net-id=3b04f928-a1bb-4e6f-8fd8-6c7f31261ca5 \
    demo1

[root@server15 kolla-ansible]# openstack server create \   
>     --image cirros \
>     --flavor m1.tiny \
>     --key-name mykey \
>     --nic net-id=3b04f928-a1bb-4e6f-8fd8-6c7f31261ca5 \
>     demo1
```
#### Issue：ImportError: cannot import name decorate
```
[root@server15 kolla-ansible]# ./init-runonce

    from decorator import decorate
ImportError: cannot import name decorate

Done.

To deploy a demo instance, run:

openstack server create \
    --image cirros \
    --flavor m1.tiny \
    --key-name mykey \
    --nic net-id= \
    demo1
```
```
pip install decorate                     # no use
pip install decorator                    # iinstalled already, no use
             
pip install -U decorator                 # Fixed 
# 或者
pip install --ignore-installed  python-openstackclient 
```
#### 3. 给云主机分配浮劢 IP 地址

![](https://i.loli.net/2019/03/24/5c97800b81617.png)
![](https://i.loli.net/2019/03/24/5c97802ed0a95.png)

测试在物理机可以ping通，内网和外网地址
```
DaviddeMacBook-Pro:~ david$ ping 192.168.0.115

DaviddeMacBook-Pro:~ david$ ping 10.0.0.17
```

# 2. 查看openstack项目中的信息和云主机网络连通性

## 2.1 查看路由信息
```
[root@server15 ~]# source /etc/kolla/admin-openrc.sh 
[root@server15 ~]# openstack router list
```
![](https://i.loli.net/2019/03/24/5c9781f001abb.png)
```
[root@server15 ~]# openstack router show demo-router
```
![](https://i.loli.net/2019/03/24/5c97828d878dd.png)

## 2.2 查看网络列表
```
[root@server15 ~]# openstack network list

[root@server15 ~]# openstack subnet list
```
![](https://i.loli.net/2019/03/24/5c9783bde6f71.png)

## 2.3 查看云主机实例的信息
```
[root@server15 ~]# openstack server list

[root@server15 ~]# openstack server show demo1
```

![](https://i.loli.net/2019/03/24/5c9784a543f70.png)

## 2.4 查看已有网络的NameSpace
```
[root@server15 ~]# ip netns
qrouter-e55fd4db-5b92-45ef-93ff-13d15d281cc4 (id: 1)
qdhcp-3b04f928-a1bb-4e6f-8fd8-6c7f31261ca5 (id: 0)
```
```
[root@server15 ~]# ip netns exec qrouter-e55fd4db-5b92-45ef-93ff-13d15d281cc4 ping 192.168.0.115
PING 192.168.0.115 (192.168.0.115) 56(84) bytes of data.
64 bytes from 192.168.0.115: icmp_seq=1 ttl=64 time=9.65 ms
64 bytes from 192.168.0.115: icmp_seq=2 ttl=64 time=1.28 ms
^C
--- 192.168.0.115 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.283/5.468/9.654/4.186 ms

[root@server15 ~]# ip netns exec qrouter-e55fd4db-5b92-45ef-93ff-13d15d281cc4 ping 10.0.0.17
PING 10.0.0.17 (10.0.0.17) 56(84) bytes of data.
64 bytes from 10.0.0.17: icmp_seq=1 ttl=64 time=0.726 ms
64 bytes from 10.0.0.17: icmp_seq=2 ttl=64 time=1.27 ms
^C
--- 10.0.0.17 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.726/0.998/1.271/0.274 ms
```
## 2.5 通过“Floating IP”访问虚拟机
- 用户名"cirros"，密码"cubswin:)"
```
[root@server15 ~]# ssh cirros@192.168.0.115
$ pwd
/home/cirros
$ ping baidu.com
PING baidu.com (220.181.57.216): 56 data bytes
64 bytes from 220.181.57.216: seq=0 ttl=44 time=76.601 ms
^C
--- baidu.com ping statistics ---
2 packets transmitted, 1 packets received, 50% packet loss
round-trip min/avg/max = 76.601/76.601/76.601 ms
```

