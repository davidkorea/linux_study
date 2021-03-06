# Linux计划任务与日志管理

1. 计划任务-at-cron-计划任务使用方法
2. 日志的种类和记录的方式-自定义ssh服务日志类型和存储位置
3. 实战-日志切割-搭建远程日志收集服务器
4. 实战-配置公司内网服务器每天定时自动开关机

# 1. 计划任务-at-cron-计划任务使用方法

计划任务的作用是做一些周期性的任务，在生产中的主要用来定期备份数据。CROND这个守护进程是为了周期性执行任务或处理等待事件而存在。 
- 任务调度分两种：系统任务调度，用户任务调度
- 计划任务的安排方式分两种:
  - 一种是定时性的，也就是例行。就是每隔一定的周期就要重复来做这个事情
  - 一种是突发性的，就是这次做完了这个事，就没有下一次了，临时决定，只执行一次的任务
- at和crontab这两个命令：
  - at：它是一个可以处理仅执行一次就结束的指令
  - crontab：它是会把你指定的工作或任务，比如：脚本等，按照你设定的周期一直循环执行下去

## 1.1 at 计划任务的使用
- 语法格式： at  时间
- 服务：atd
#### 1. 查看at服务状态
```
[root@localhost ~]# systemctl status atd
[root@localhost ~]# systemctl start atd
[root@localhost ~]# systemctl is-enabled atd
enabled

[root@localhost ~]# chkconfig --list | grep atd   # centos6 执行此命令，7不能执行
```

#### 2. 创建计划任务
```
[root@localhost at]# date
Fri Mar  1 09:30:38 CST 2019

[root@localhost at]# at 9:35
at> mkdir /abc
at> touch /abc/1.txt<EOT>       # crtl+d 保存退出
job 6 at Fri Mar  1 09:35:00 2019

[root@localhost at]# atq        # 或者at -l
6	Fri Mar  1 09:35:00 2019 a root

[root@localhost at]# at -c 6    # 查看at计划源文件中包含的命令，长篇最后
#!/bin/sh
... ...
mkdir /abc
touch /abc/1.txt
marcinDELIMITER6ed4ae62

[root@localhost at]# tail -5 /var/spool/at/       # 查看at计划源文件中包含的命令，简单方法
a00006018a8adf  .SEQ            spool/          
[root@localhost at]# tail -5 /var/spool/at/a00006018a8adf 
}
${SHELL:-/bin/sh} << 'marcinDELIMITER6ed4ae62'
mkdir /abc
touch /abc/1.txt
marcinDELIMITER6ed4ae62

[root@localhost at]# at -l      # 或者 atq命令
5	Fri Mar  1 09:35:00 2019 a root
[root@localhost at]# atrm 5     # 删除计划任务

```
at计划任务的特殊写法
```
[root@localhost ~]# at 20:00 2018-10-1   在某天 
[root@localhost ~]# at now +10min        在 10分钟后执行
[root@localhost ~]# at 17:00 tomorrow    明天下午5点执行
[root@localhost ~]# at 6:00 pm +3 days   在3天以后的下午6点执行
[root@localhost ~]# at 23:00 < a.txt
```
## 1.2 crontab定时任务的使用
- crond命令定期检查是否有要执行的工作，如果有要执行的工作便会自动执行该工作
- cron是一个linux下的定时执行工具，可以在无需人工干预的情况下运行作业。
- 系统周期性所要执行的工作，如更新whatis数据库, updatedb数据库，日志定期切割，收集系统状态信息，/tmp定期清理
- crontab的参数：
  - ```crontab -u david```       #指定david用户的cron服务
  - ```crontab -l```             #列出当前用户下的cron服务的详细内容
  - ```crontab -u david -l```    #列出指定用户david下的cron服务的详细内容
  - ```crontab -r```             #删除cron服务
  - ```crontab -u david -r```    #root想删除david的cron计划任务
  - ```crontab -e```             #编辑cron服务

- cron -e 中编辑时的语法
  - 格式说明
  ```
  Example of job definition:
  .---------------- minute (0 - 59)
  |  .------------- hour (0 - 23)
  |  |  .---------- day of month (1 - 31)
  |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  |  |  |  |  |
  *  *  *  *  * user-name  command to be executed
  ```
  - 参数说明
  
  |符号|说明|示例|
  |-|-|-|
  |* |代表取值范围内的数字| (任意/每)|
  | / | 指定时间的间隔频率 | */10   0-23/2(0-23时每两小时) |
  | -| 代表从某个数字到某个数字 | 8-17  |
  | ，| 分开几个离散的数字 |6,10-13,20|

#### 1. 创建计划任务
- 用户级别的计划任务 /var/spool/cron/
  ```
  [root@localhost ~]# systemctl start crond
  [root@localhost ~]# systemctl enable crond

  [root@localhost ~]# crontab -e    # 进入vim页面编辑命令
  """
  2 10 * * * tar zcvf /opt/grub2.tar.gz /boot/grub2
  """
  no crontab for root - using an empty one
  crontab: installing new crontab
  [root@localhost ~]# crontab -l    # 查看
  2 10 * * * tar zcvf /opt/grub2.tar.gz /boot/grub2

  [root@localhost ~]# ll /var/spool/cron/     # 查看系统中全部的cron计划任务
  total 4
  -rw-------. 1 root root 50 Mar  1 10:01 root
  ```
- 系统级别的计划任务 /etc/crontab 
    也可以直接在/etc/crontab中添加计划任务，与上面使用语法相同。 不建议用户级别的任务全部放到系统级别下执行，但如果都是root身份执行，不好区分的话就都可以。
    
  - 使用crontab命令的注意事项：
    - 环境变量的问题
      - ```SHELL=/bin/bash```                        #指定操作系统使用哪个shell
      - ```PATH=/sbin:/bin:/usr/sbin:/usr/bin```     #系统执行命令的搜索路径
    - 清理邮件日志，比如使用重定向 ```> /dev/null  2>&1```
    
  ```
  [root@localhost ~]# ll /etc/crontab 
  -rw-r--r--. 1 root root 451 Jun 10  2014 /etc/crontab
  
  [root@localhost ~]# vim /etc/crontab 

  SHELL=/bin/bash                        #指定操作系统使用哪个shell
  PATH=/sbin:/bin:/usr/sbin:/usr/bin     #系统执行命令的搜索路径
  MAILTO=root                            #将执行任务的信息通过邮件发送给xx用户

  # For details see man 4 crontabs

  # Example of job definition:
  # .---------------- minute (0 - 59)
  # |  .------------- hour (0 - 23)
  # |  |  .---------- day of month (1 - 31)
  # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  # |  |  |  |  |
  # *  *  *  *  * user-name  command to be executed

  ~                                                                                              
  ```

  ```
  [root@localhost ~]# ll /etc/cron    # 2 times tab
  cron.d/       cron.deny     cron.monthly/ cron.weekly/  
  cron.daily/   cron.hourly/  crontab     
  
  [root@localhost ~]# find /etc/cron* -type f
  /etc/cron.d/0hourly
  /etc/cron.d/raid-check
  /etc/cron.d/sysstat
  /etc/cron.daily/logrotate
  /etc/cron.daily/man-db.cron
  /etc/cron.daily/mlocate
  /etc/cron.deny
  /etc/cron.hourly/0anacron
  /etc/crontab
  ```
    - cron.d/       #是系统自动定期需要做的任务，但是又不是按小时，按天，按星期，按月来执行的，那么就放在这个目录下面。
    - cron.deny     #控制用户是否能做计划任务的文件;
    - cron.monthly/  #每月执行的脚本;
    - cron.weekly/   #每周执行的脚本;
    - cron.daily/     #每天执行的脚本;
    - cron.hourly/   #每小时执行的脚本;
    - crontab       #主配置文件 也可添加任务;
  
#### 2. 常见写法：
- 每天晚上21:00 重启apache
  - ```0 21 * * * /etc/init.d/httpd  restart```
- 每月1、10、22日的4 : 45重启apache。
  - ```45 4 1,10,22 * *  /etc/init.d/httpd  restart```
- 每月1到10日的4 : 45重启apache。
  - ```45 4 1-10 * *   /etc/init.d/httpd  restart```
- 每隔两天的上午8点到11点的第3和第15分钟重启apach
  - ```3,15 8-11 */2 * *  /etc/init.d/httpd  restart```
- 晚上11点到早上7点之间，每隔一小时重启apach
  - ```0 23-7/1 * * * /etc/init.d/apach restart```
- 周一到周五每天晚上 21:15 寄一封信给 root@panda:
  - ```15 21 * * 1-5  mail -s "hi" root@panda < /etc/fstab```
- 互动：crontab不支持每秒。 每2秒执行一次脚本，怎么写？ 在脚本的死循环中，添加命令 sleep 2 ，执行30次自动退出，然后添加，计划任务：```* * * * *  /back.sh``` 


#### 3. 案例

> 每天2：00备份/etc/目录到/tmp/backup下面
> 将备份命令写入一个脚本中
> 每天备份文件名要求格式： 2017-08-19_etc.tar.gz
> 在执行计划任务时，不要输出任务信息
> 存放备份内容的目录要求只保留三天的数据

```
[root@localhost ~]# vim /etc/crontab 

  SHELL=/bin/bash                        # 指定操作系统使用哪个shell
  PATH=/sbin:/bin:/usr/sbin:/usr/bin     # 系统执行命令的搜索路径
  MAILTO=root                            # 将执行任务的信息通过邮件发送给xx用户

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

0 2 * * * root /bin/back.sh   # 系统执行命令的搜索路径，已在上方定义。
                              # 或者直接写绝对路径 /root/backup.sh
~                                            
```
```
===========================================================
# 提前测试需执行命令
mkdir /tmp/backup
tar zcf etc.tar.gz /etc
find /tmp/backup -name “*.tar.gz” -mtime +3 -exec rm -rf {}\;
============================================================
[root@localhost ~]# crontab -l
0 2 * * * /root/backup.sh & > /dev/null   # 后台执行backu.sh并将标准输出放入null，即不输出

[root@localhost ~]# cat backup.sh 
#!/bin/bash
find /tmp/backup -name "*.tar.gz" -mtime +3 -exec rm -f {}\;
#find /tmp/backup -name "*.tar.gz" -mtime +3 -delete
#find /tmp/backup -name "*.tar.gz" -mtime +3 |xargs rm -f
tar zcf /tmp/backup/`date +%F`_etc.tar.gz /etc
```
工作中备份的文件不要放到/tmp,因为过一段时间，系统会清空备/tmp目录。

# 2. 日志种类及记录方式-自定义ssh服务日志类型和存储位置

在centos7中，系统日志消息由两个服务负责处理：systemd-journald和rsyslog

## 2.1 常见日志文件的作用
系统日志文件概述：/var/log目录保管由rsyslog维护的，里面存放的一些特定于系统和服务的日志文件

|日志文件|用途|
|-|-|
|/var/log/message|大多数系统日志消息记录在此处。有也例外的：如与身份验证，电子邮件处理相关的定期作业任务等|
|/var/log/secure|安全和身份验证相关的消息和登录失败的日志文件。  ssh远程连接产生的日志|
|/var/log/maillog|与邮件服务器相关的消息日志文件|
|/var/log/cron|与定期执行任务相关的日志文件|
|/var/log/boot.log|与系统启动相关的消息记录|
|/var/log/dmesg|与系统启动相关的消息记录|


#### 1. 使用/var/log/secure文件查看暴力破解系统的ip
```
[root@localhost ~]# grep Failed /var/log/secure
Feb 28 17:19:16 localhost sshd[31061]: Failed password for invalid user ROOT from 192.168.0.219 port 4743 ssh2
Feb 28 17:19:22 localhost sshd[31061]: Failed password for invalid user ROOT from 192.168.0.219 port 4743 ssh2
Mar  1 11:09:29 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:33 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:36 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:40 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:44 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2

[root@localhost ~]# grep Failed /var/log/secure | awk '{print $11}' | uniq -c  
      2 ROOT                                    # awk按列（空格区分）查看，打印第十一列，count unique
      5 192.168.0.219
```
#### 2. 使用/var/log/btmp文件查看暴力破解系统的用户

/var/log/btmp文件是记录错误登录系统的日志。如果发现/var/log/btmp日志文件比较大，大于1M，就算大了，就说明很多人在暴力破解ssh服务，此日志需要使用lastb程序查看。

```
[root@localhost ~]# lastb
xerox    ssh:notty    192.168.0.211    Fri Mar  1 11:09 - 11:09  (00:00)    
xerox    ssh:notty    192.168.0.211    Fri Mar  1 11:09 - 11:09  (00:00)    
xerox    ssh:notty    192.168.0.211    Fri Mar  1 11:09 - 11:09  (00:00)    

btmp begins Fri Mar  1 11:09:29 2019
```

发现后，使用防火墙，拒绝掉：命令如下：
```
iptables -A INPUT -i eth0 -s. 192.168.0.211 -j DROP
```
查看恶意ip试图登录次数：
```
lastb | awk  '{ print $3}'  | sort | uniq -c | sort -n
```
清空日志：
- 方法1：```[root@localhost ~]# > /var/log/btmp```
- 方法2：```rm -rf /var/log/btmp  && touch /var/log/btmp```
两者的区别？使用方法2，因为创建了新的文件，而正在运行的服务，还用着原来文件的inode号和文件描述码，所需要重启一下rsyslog服务。建议使用方法1  > /var/log/btmp


#### 3. 使用/var/log/wtmp文件查看每个用户成功登录次数和持续时间等信息
可以用last命令输出wtmp中内容，last显示到目前为止，成功登录系统的记录

```
[root@localhost ~]# last    # 或者last -f /var/log/wtmp 
xerox    pts/1        192.168.0.219    Fri Mar  1 11:09   still logged in   
root     pts/0        192.168.0.219    Fri Mar  1 09:27   still logged in   
```


## 2.2 日志的记录方式: 分类→ 级别→

- 日志的分类:
  - daemon,  后台进程相关  
  - kern,  	内核产生的信息
  - lpr,   	 打印系统产生的
  - authpriv,  安全认证
  - cron,   	 定时相关
  - mail, 	 邮件相关
  - syslog,  	日志服务本身的
  - news, 	 新闻系统
  - local0~7,  自定义的日志设备，8个系统保留的类，供其它的程序使用或者是用户自定义

- 日志的级别: 0 严重 -> 7 不严重

    |编码|优先级|严重性|
    |-|-|-|
    |7|debug|信息对开发人员调试应用程序有用，在操作过程中无用|
    |6|info|正常的操作信息，可以收集报告，测量吞吐量等|
    |5|notice|注意，正常但重要的事件
    |4|warning|警告，提示如果不采取行动。将会发生错误。比如文件系统使用90%|
    |3|err|错误，阻止某个模块或程序的功能不能正常使用|
    |2|crit|关键的错误，已经影响了整个系统或软件不能正常工作的信息|
    |1|alert|警报,需要立刻修改的信息|
    |0|emerg|紧急，内核崩溃等严重信息|

## 2.3 rsyslog日志服务

- rhel5: 服务名称syslog  -> 配置文件  /etc/syslog.conf
- rhel6-7: 服务名称rsyslog -> 配置文件  /etc/rsyslog.conf

```
[root@localhost ~]# vim /etc/rsyslog.conf 
# rsyslog configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### MODULES ####

# The imjournal module bellow is now used as a message source instead of imuxsock.
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imjournal # provides access to the systemd journal
#$ModLoad imklog # reads kernel messages (the same are read from journald)
#$ModLoad immark  # provides --MARK-- message capability

# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
#$ModLoad imtcp
#$InputTCPServerRun 514


#### GLOBAL DIRECTIVES ####

# Where to place auxiliary files
$WorkDirectory /var/lib/rsyslog

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# File syncing capability is disabled by default. This feature is usually not required,
# not useful and an extreme performance hit
#$ActionFileEnableSync on

# Include all config files in /etc/rsyslog.d/
$IncludeConfig /etc/rsyslog.d/*.conf

# Turn off message reception via local log socket;
# local messages are retrieved through imjournal now.
$OmitLocalLogging on

# File to store the position in the journal
$IMJournalStateFile imjournal.state


#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log


# ### begin forwarding rule ###
# The statement between the begin ... end define a SINGLE forwarding
# rule. They belong together, do NOT split them. If you create multiple
# forwarding rules, duplicate the whole block!
# Remote Logging (we use TCP for reliable delivery)
#
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
#$ActionQueueFileName fwdRule1 # unique name prefix for spool files
#$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
#$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
#$ActionQueueType LinkedList   # run asynchronously
#$ActionResumeRetryCount -1    # infinite retries if host is down
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
#*.* @@remote-host:514
# ### end of the forwarding rule ###
```

- ```#$UDPServerRun 514```, 允许514端口接收使用UDP协议转发过来的日志
- ```#$InputTCPServerRun 514```, 允许514端口接收使用TCP协议转发过来的日志
- ```#kern.*    /dev/console```, 内核类型的所有级别日志 →存放到→    
- ```*.info;mail.none;authpriv.none;cron.none     /var/log/messages```,所有的类别级别是info以上 除了mail,authpriv,cron (产生的日志太多,不易于查看) 

- ```authpriv.*    /var/log/secure```, 认证的信息→存放→ 
- ```mail.*        -/var/log/maillog```, 邮件相关的信息→存放→，“- ”号：邮件的信息比较多,现将数据存储到内存,达到一定大小,全部写到硬盘.有利于减少I/O进程的开销。数据存储在内存,如果关机不当数据消失  
- ```cron.*        /var/log/cron```, 计划任务相关的信息→存放→  
- ```local7.*      /var/log/boot.log```, 开机时显示的信息→存放→  

- 日志输入的规则
  - ```.info```,   	大于等于info级别的信息全部记录到某个文件
  - ```.=级别```,   仅记录等于某个级别的日志。例:.=info  只记录info级别的日志  
  - ```.!级别```, 	除了某个级别意外,记录所有的级别信息。例.!err  除了err外记录所有
  - ```.none```,  指的是排除某个类别  例： mail.none  所有mail类别的日志都不记录

## 2.4 自定义ssh服务的日志类型和存储位置
#### 1. 把local0类别的日志，保存到 /var/log/sshd.log路径
```
[root@localhost ~]# vim /etc/rsyslog.conf    # 把local0类别的日志，保存到 /var/log/sshd.log路径
 72 # Save boot messages also to boot.log
73 local7.*                                   /var/log/boot.log
74 local0.*                                   /var/log/ssdh.log   #新增
```
#### 2. 定义ssh服务的日志类别为local0，编辑sshd服务的主配置文件
```
[root@localhost ~]# vim /etc/ssh/sshd_config 
 30 # Logging
 31 #SyslogFacility AUTH
 32 #SyslogFacility AUTHPRIV # 原有日志类型
 33 SyslogFacility local0
 
[root@localhost ~]# systemctl restart rsyslog   # 先重启rsyslog服务(生效配置)
[root@localhost ~]# systemctl restart sshd    # 再重启sshd服务.生成日志
[root@localhost ~]# cat /var/log/ssdh.log     # 应该是sshd.log 写错了
Mar  1 13:36:30 localhost sshd[44628]: Server listening on 0.0.0.0 port 22.
Mar  1 13:36:30 localhost sshd[44628]: Server listening on :: port 22.
```


# 3. 实战-日志切割-搭建远程日志收集服务器

## 3.1 日志的切割
  在linux下的日志会定期进行滚动增加，我们可以在线对正在进行回滚的日志进行指定大小的切割（动态），如果这个日志是静态的。比如没有应用向里面写内容。那么我们也可以用split工具进行切割，其中Logrotate支持按时间和大小来自动切分,以防止日志文件太大。

  日志是很大的,如果让日志无限制的记录下去，是一件很可怕的事情，日积月累就有几百兆占用磁盘的空间，如果你要找出某一条可用信息：→海底捞针。当日志达到某个特定的大小,我们将日志分类,之前的日志保留一个备份,再产生的日志创建一个同名的文件保存新的日志。

- logrotate配置文件主要有：```/etc/logrotate.conf``` 以及 ```/etc/logrotate.d/ ```这个子目录下的明细配置文件。
- logrotate的执行由crond服务调用的。```vim /etc/cron.daily/logrotate   #查看logrotate脚本内容```
- logrotate程序每天由cron在指定的时间（/etc/crontab）启动

## 3.2 解读配置文件
```
[root@localhost ~]# vim /etc/logrotate.conf 

# see "man logrotate" for details
# rotate log files weekly
weekly    # 每周执行回滚，或者说每周执行一次日志回滚

# keep 4 weeks worth of backlogs
rotate 4    # 表示日志切分后历史文件最多保存离现在最近的多少份,切分后一共保存多少个子份

# create new (empty) log files after rotating old ones
create    # 指定新创建的文件的权限与所属主与群组

# use date as a suffix of the rotated file
dateext   # 使用日期为后缀的回滚文件  #可以去/var/log目录下看看

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {               
    monthly                     # 每月轮换一次，切分一次
    create 0664 root utmp
        minsize 1M              # 文件超过1M→进行回滚，超过1M，及时不到1个月也会切分
    rotate 1
}

/var/log/btmp {                 
    missingok                   # 如果文件丢失，将不报错
    monthly                     # 每月轮换一次，切分一次
    create 0600 root utmp       # 设置btmp这个日志文件的权限，属主，属组
    rotate 1                    # 日志切分后历史文件最多保存1份，不含当前使用的日志

}

# system-specific logs may be also be configured here.
```

- 其它参数说明：
	- ```monthly```: 日志文件将按月轮循。其它可用值为‘daily’，‘weekly’或者‘yearly’。
	- ```rotate 5```: 一次将存储5个归档日志。对于第六个归档，时间最久的归档将被删除。
	- ```compress```: 在轮循任务完成后，已轮循的归档将使用gzip进行压缩。
	- ```delaycompress```: 总是与compress选项一起用，delaycompress选项指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行。这在你或任何软件仍然需要读取最新归档时很有用。
  - ```missingok```: 在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误。
  - ```notifempty```: 如果日志文件为空，轮循不会进行。
  - ```create 644 root root```: 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件。
  - ```postrotate/endscript```: 在所有其它指令完成后，postrotate和endscript里面指定的命令将被执行。在这种情况下，rsyslogd 进程将立即再次读取其配置并继续运行。
  - /var/lib/logrotate/status中默认记录logrotate上次轮换日志文件的时间。



## 3.3 实战-使用logrotate进行ssh日志分割
```
[root@localhost ~]# vim /etc/logrotate.d/sshd
[root@localhost ~]# cat !$
cat /etc/logrotate.d/sshd
/var/log/sshd.log{
	missingok
	weekly
	create 0600 root root
	minsize 1M
	rotate 3 
}

[root@localhost ~]# systemctl restart rsyslog

[root@localhost ~]# logrotate -d /etc/logrotate.d/sshd
reading config file /etc/logrotate.d/sshd
Allocating hash table for state file, size 15360 B

Handling 1 logs

rotating pattern: /var/log/sshd.log weekly (3 rotations)
empty log files are rotated, only log files >= 1048576 bytes are rotated, old logs are removed
considering log /var/log/sshd.log
  log does not need rotating (log has been already rotated)[root@localhost ~]# 
  
[root@localhost ~]# logrotate -vf /etc/logrotate.d/sshd     # -v 显示指令执行过程 -f 强制执行
reading config file /etc/logrotate.d/sshd
Allocating hash table for state file, size 15360 B

Handling 1 logs

rotating pattern: /var/log/sshd.log forced from command line (3 rotations)
empty log files are rotated, only log files >= 1048576 bytes are rotated, old logs are removed
considering log /var/log/sshd.log
  log needs rotating
rotating log /var/log/sshd.log, log->rotateCount is 3
dateext suffix '-20190301'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
renaming /var/log/sshd.log.3 to /var/log/sshd.log.4 (rotatecount 3, logstart 1, i 3), 
old log /var/log/sshd.log.3 does not exist
renaming /var/log/sshd.log.2 to /var/log/sshd.log.3 (rotatecount 3, logstart 1, i 2), 
old log /var/log/sshd.log.2 does not exist
renaming /var/log/sshd.log.1 to /var/log/sshd.log.2 (rotatecount 3, logstart 1, i 1), 
old log /var/log/sshd.log.1 does not exist
renaming /var/log/sshd.log.0 to /var/log/sshd.log.1 (rotatecount 3, logstart 1, i 0), 
old log /var/log/sshd.log.0 does not exist
log /var/log/sshd.log.4 doesn't exist -- won't try to dispose of it
fscreate context set to system_u:object_r:var_log_t:s0
renaming /var/log/sshd.log to /var/log/sshd.log.1
creating new /var/log/sshd.log mode = 0600 uid = 0 gid = 0
set default create context

[root@localhost ~]# ls /var/log/sshd*
/var/log/sshd.log  /var/log/sshd.log.1

[root@localhost ~]# ll -h /var/log/sshd*	# 强行5次拆分
-rw-------. 1 root root   0 Mar  1 14:38 /var/log/sshd.log
-rw-------. 1 root root   0 Mar  1 14:38 /var/log/sshd.log.1
-rw-------. 1 root root   0 Mar  1 14:35 /var/log/sshd.log.2
-rw-------. 1 root root 219 Mar  1 14:14 /var/log/sshd.log.3
```

## 3.4 配置远程日志服务器-实现日志集中的管理
需要2台虚拟机，暂时不操作，方法如下。

实验拓扑图： 
![](https://i.loli.net/2019/03/01/5c78d5543ed16.png)

```
server端配置
[root@xuegod63 ~]# vim  /etc/rsyslog.conf   # 	使用TCP协议方式，收集日志
改：19 #$ModLoad imtcp
    20 #$InputTCPServerRun 514
为：
19 $ModLoad imtcp
 20 $InputTCPServerRun 514
注：使用UDP协议→速度快→不保证数据的完整，使用TCP协议→可靠.完整
[root@xuegod63 ~]# systemctl  restart  rsyslog   #重新启动 rsyslog
查看服务监听的状态：
[root@xuegod63 ~]# netstat -anlpt| grep 514
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      45631/rsyslogd      
tcp6       0      0 :::514                  :::*                    LISTEN      45631/rsyslogd      

服务端验证:
在服务端关闭selinux和防火墙
[root@xuegod63 ~]# getenforce 
Enforcing
[root@xuegod63 ~]# setenforce 0   #关闭selinux功能
[root@xuegod63 ~]#getenforce 
Permissive
[root@xuegod63 ~]# systemctl stop firewalld
[root@xuegod63 ~]# systemctl status firewalld
[root@xuegod63 ~]# iptables -F    #清空防火墙规则 

 client端配置
登录xuegod64.cn
[root@xuegod63 ~]# vim /etc/rsyslog.conf  #在90行之后，插入
*.*   @@192.168.1.63:514

注： *.* 所有类别和级别的日志   ； @@192.168.1.63:514  远端tcp协议的日志服务端的IP和端口
重启rsyslog 服务
[root@xuegod64 ~]# systemctl restart rsyslog.service

查看日志：
[root@xuegod63 ~]# tail -f /var/log/messages | grep xuegod64 --color   #动态查看日志

在客户端xuegod64进行测试
语法： logger  要模拟发送的日志
[root@xuegod64 ~]# logger  “aaaaa”
[root@xuegod63 ~]# tail -f /var/log/messages | grep xuegod64 --color  #服务器端到查看消息
May 21 16:32:16 xuegod64 root: aaaaa
注：
总结：服务器使用udp协议，客户端只能使用的配置文件中这一行只能有一个@
*.*  @192.168.1.64:514
服务器使用tcp协议，客户端只能使用的配置文件中这一行必须有两个@@
*.*  @@192.168.1.64:514

```
