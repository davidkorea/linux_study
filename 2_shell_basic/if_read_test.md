# 条件测试语句和if流程控制语句的使用
1. read命令键盘读取变量的值
2. 流程控制语句if
3. test测试命令
4. 流程控制过程中复杂条件和通配符
5. 实战-3个shell脚本实战

# 1. read命令键盘读取变量的值

从键盘读取变量的值，通常用在shell脚本中与用户进行交互的场合。该命令可以一次读取多个变量的值，变量和输入的值都需要使用空格隔开。在read命令后面，如果没有指定变量名，读取的数据将被自动赋值给特定的变量REPLY

- 读取多个值，从标准输入读取一行，直至遇到第一个空白符或换行符。把用户键入的第一个词存到变量first中，把该行的剩余部分保存到变量last中
  ```shell
  [root@localhost ~]# read first last
  1 2
  [root@localhost ~]# echo $first $last 
  1 2
  ```
  
- ```read -s ``` 将你输入隐藏，值赋给passwd。隐藏密码信息
  ```shell
  [root@localhost ~]# read -s passwd
  [root@localhost ~]# echo $passwd 
  11111
  ```
- ```read -t ``` 输入的时间限制, 超过5秒没有输入，直接退出
  ```shell
  [root@localhost ~]# read -t 5 time
  ```
- ```read -n ``` 输入的长度限制, 最多只接受3个字符
  ```shell
  [root@localhost ~]# read -n 3 length
  jgl[root@localhost ~]# echo $length 
  jgl
  ```
- ```read -r ```允许让输入中的内容包括：空格、/、\、 ？等特殊字符串
  ```shell
  [root@localhost ~]# read -r line
  hjkjh jg jk $#!&^ kjh
  [root@localhost ~]# echo $line 
  hjkjh jg jk $#!&^ kjh 
  ```
- ```read -p ``` 用于给出提示符，同echo –n “…“来给出提示符
  ```shell
  [root@localhost ~]# read -p "pls input: " pass
  pls input: 12345
  [root@localhost ~]# echo $pass
  12345
  ```
  ```shell
  [root@localhost ~]# echo -n "please input: "; read pass
  please input: 12345
  [root@localhost ~]# echo $pass
  12345
  ```
- read案例an'li
  ```shell
  [root@localhost ~]# vim read.sh
    #!/bin/bash
    read -p "name: " name
    read -p "age: " age
    read -p "gender: " gender
    clear     # 清屏
    cat<<eof
    ####################
    name:   $name
    age:    $age
    gender: $gender
    ####################
    eof

  [root@localhost ~]# bash read.sh 
  ####################
  name:   david
  age:    12
  gender: man
  ####################
  ```
# 2. 流程控制语句if

### 2.1 if-then-fi
```shell
if condition
then
  command
fi
```
```shell
if condition ; then
  command
fi
```

- 示例
  ```shell
  [root@localhost ~]# vim if.sh
    #!/bin/bash
    if ls /mnt ; then             # if `ls /mnt` 也可以
            echo "ok"
    fi

  [root@localhost ~]# bash if.sh 
  if.sh: line 4: syntax error: unexpected end of file
  [root@localhost ~]# vim +4 if.sh 
  [root@localhost ~]# bash if.sh 
  ok
  ```
### 2.2 if-then-else-fi
```shell
if condition ; then
  command1
else
  command2
fi
```
- 示例
  ```shell
  [root@localhost ~]# vim if.sh 
    #!/bin/bash
    read -p "user name: " user
    if grep $user /etc/passwd ; then
            echo "login success"
    else
            echo "login failed"
    fi  
  
  [root@localhost ~]# bash if.sh 
  user name: xerox
  xerox:x:1000:1000:xerox:/home/xerox:/bin/bash
  login success
  ```
### 2.3 multi if-then elif-...-else-fi
```bash
if condition1 ; then
  command1
elif condition2 ; then
  command2
elif condition3 ; then
  ...
else
  commandn
fi
```
- 示例
  ```bash
  [root@localhost ~]# vim if.sh 
    #!/bin/bash
    read -p "user name: " user
    if grep $user /etc/passwd ; then
            echo "user $user exsits"
    elif ls -d /home/$user ; then
            echo "no account, but '/home/'$user exsits"
    else
            echo "no $user account and home dir"
    fi

  [root@localhost ~]# bash if.sh 
  user name: david
  david:x:1001:1001::/home/david:/bin/bash
  user david exsits
  
  [root@localhost ~]# bash if.sh 
  user name: frank
  ls: cannot access /home/frank: No such file or directory
  no frank account and home dir
  ```
# 3. test测试命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

格式：test 测试条件, 如果结果是对的，也叫结果为真，用$?=0表示，反之为假，用非0表示

### 3.1 数值(integer only)比较

|参数|说明|示例|
|-|-|-|
|-eq|等于则为真|[ “$a” -eq “$b” ]|
|-ne|不等于则为真|[ “$a” -ne “$b” ]|
|-gt|大于则为真|[ “$a” -gt “$b” ]|
|-ge|大于等于则为真|[ “$a” -ge “$b” ]|
|-lt|小于则为真|[ “$a” -lt “$b” ]|
|-le|小于等于则为真|[ “$a” -le “$b” ]|

- 在做数值比较时，只能用整数
  ```
  [root@localhost ~]# vim num.sh
    #!/bin/bash
  read -p "please input 2 num: " num1 num2
  if [ $num1 -gt $num2  ] ; then
          echo "$num1 is bigger than $num2"
  elif [ $num1 -lt $num2  ] ; then
          echo "$num1 is less than $num2"
  else
          echo "$num1 equals $num2"
  fi
  
  [root@localhost ~]# bash num.sh 
  please input 2 num: 1 2
  1 is less than 2
  [root@localhost ~]# bash num.sh 
  please input 2 num: 22 22
  22 equals 22
  [root@localhost ~]# bash num.sh 
  please input 2 num: 3 1
  3 is bigger than 1
  ```
### 3.2 字符串比较

|参数|说明|示例|
|-|-|-|
|==|等于则为真|[ “$a” == “$b” ]|
|!=|不相等则为真|[ “$a” != “$b” ]|
|-z 字符串|字符串的长度为零则为真|[ -z “$a” ]|
|-n 字符串|字符串的长度不为空则为真|[ -n “$a” ]|
|str1 > str2|str1大于str2为真|[ str1 \\> str2 ]|
|str1 < str2|str1小于str2为真|[ str1 \\< str2 ]|



- 根据用户名判断是否是超级管理员
  ```
  [root@localhost ~]# vim str.sh
    #!/bin/bash
    read -p "inout user: " user
    if [ $user == "root" ] ; then
            echo "you are admin"
    else
            echo "you are general user"
    fi

  [root@localhost ~]# bash str.sh 
  inout user: root
  you are admin
  ```

- 字符串大小比较
  - 大于号和小于号必须转义，要不然SHELL会把它当成重定向符号
  - 大于和小于它们的顺序和sort排序是不一样的
  - 在test比较测试中，它使用的是ASCII顺序，大写字母是小于小写字母的
  ```
  [root@localhost ~]# bash str.sh 
    #!/bin/bash
    read -p "inout 2 strings: " str1 str2
    if [ $str1 \> $str2 ] ; then
            echo "$str1 > $str2"
    else
            echo "$str1 < $str2"
    fi
    
  [root@localhost ~]# bash str.sh 
  inout user: aabc Aabc
  aabc > Aabc
  ```

