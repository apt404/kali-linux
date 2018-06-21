---
typora-root-url: ..
---

# 144：Metasploit Framework 和MSF架构

[TOC]

> - MSF(Metasploit简称MSF)默认集成与Kali-Linux系统中
> - 使用Postgresql数据库存储数据
>   - 早期版本需要先启动数据库再启动MSF
>



#### 1.MSF架构图

![](/image/1529566149032.png)



Rex （包含系统底层一些库）

- 基本功能库,用于完成日常基本任务,无需人工手动编码实现
- 处理 socket连接访问,协议答应（http/SSL/SMB等
- 编码转换（XOR，Base64，Unicode）

1. MSF::Core
   - 提供MSF的核心技术API，是框架的核心能力实现库
2. MSF::Base
   - 提供有好的API接口， 便于模块调用的库
3. MSF::UI (本次课程介绍三种UI接口使用方法)
4. [ ] Driveer
5. [x]  Console
6. [x] CLI
7. [ ] WebUI
8. [ ] GUI
9. [x] Amitage

*Plugin 插件*

- 连接和调用外部扩展功能和系统

#### 2.Kali Linux系统中启动Metasploit

在kali2.0之后的版本，默认安装完之后会在右侧工作任务栏里有MSF启动图标

**当你首次启动MSF时，MSF会自动做数据库初始化连接**完成之后就会启动Metasploit

```shell
# cowsay++
 ____________
< metasploit >
 ------------
       \   ,__,
        \  (oo)____
           (__)    )\
              ||--|| *


       =[ metasploit v4.16.61-dev                         ]
+ -- --=[ 1773 exploits - 1011 auxiliary - 307 post       ]
+ -- --=[ 538 payloads - 41 encoders - 10 nops            ]
+ -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]

msf > 
```

如果在是用MSF中，你觉得数据库被是弄乱了，或者想删掉数据库&重启，我们可以时候`msfdb`命令

```shell
kali@kali:~$ msfdb 

Manage the metasploit framework database

  msfdb init     # start and initialize the database //初始化数据库
  msfdb reinit   # //删除之前已经初始化过的数据库，重新初始化数据库
  msfdb delete   # delete database and stop using it //或者你只想把现有的数据库删掉
  msfdb start    # start the database //手动启动数据库
  msfdb stop     # stop the database //停止
  msfdb status   # check service status
  msfdb run      # start the database and run msfconsole
```

查看下数据库状况

```shell
kali@kali:~$ netstat -pantu | grep 5432
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 ::1:5432                :::*                    LISTEN      -     
```

停止数据库

```shell
kali@kali:~$ msfdb stop
[+] Stopping database
```

重新初始化数据库

```shell
kali@kali:~$ sudo msfdb reinit
[+] Starting database
[+] Deleting configuration file /usr/share/metasploit-framework/config/database.yml
[+] Stopping database
[+] Starting database
[+] Creating database user 'msf'
为新角色输入的口令: 
再输入一遍: 
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

当数据库初始化完成之后数据库会自动启动，侦探在5432端口

```
kali@kali:~$ netstat -pantu | grep 5432
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 ::1:5432                :::*                    LISTEN      -  
```

