---
typora-copy-images-to: ../image
typora-root-url: ..
---

## 课时10 基本工具-NETCAT（telnet/banner、传输文本信息）

[TOC]



### 常用工具

> **Nc / ncat**
> **Wireshack**
> **Tcpdump**

$$
经常使用且功能强大安全从业者必不可少的帮手
$$

#### NETCAT-----NC

1. 网络工具中的瑞士军刀——小身材、大智慧
2. 侦听模式/传输模式
3. telnet/获取banner信息
4. 传输文本信息
5. 传输文件目录
6. 加密传输文件
7. 远程控制/木马
8. 加密所有流量
9. 流媒体服务器
10. 远程克隆硬盘

------

```shell
kali@kali:~$ nc -h
[v1.10-41.1]
connect to somewhere:	nc [-options] hostname port[s][ports] ... 
listen for inbound:	nc -l -p port [-options][hostname] [port]
options:
	-c shell commands	as `-e'; use /bin/sh to exec [dangerous!!]
	-e filename		program to exec after connect [dangerous!!]
	-b			allow broadcasts
	-g gateway		source-routing hop point[s], up to 8
	-G num			source-routing pointer: 4, 8, 12, ...
	-h			this cruft
	-i secs			delay interval for lines sent, ports scanned
        -k                      set keepalive option on socket
	-l			listen mode, for inbound connects
	-n			numeric-only IP addresses, no DNS
	-o file			hex dump of traffic
	-p port			local port number
	-r			randomize local and remote ports
	-q secs			quit after EOF on stdin and delay of secs
	-s addr			local source address
	-T tos			set Type Of Service
	-t			answer TELNET negotiation
	-u			UDP mode
	-v			verbose [use twice to be more verbose]
	-w secs			timeout for connects and final net reads
	-C			Send CRLF as line-ending
	-z			zero-I/O mode [used for scanning]
port numbers can be individual or ranges: lo-hi [inclusive];
hyphens in port names must be backslash escaped (e.g. 'ftp\-data')
```

##### 译文

```
参数选项:
	-c shell commands	as `-e'; use /bin/sh to exec [dangerous!!]
	-e filename		program to exec after connect [dangerous!!]
	-b			    是否允许广播
	-g gateway		源路由跳点[s], up to 8
	-G num			源路由指针: 4, 8, 12, ...
	-h			    this cruft
	-i secs			线路延时间隔, 端口扫描
    -k              set keepalive option on socket
	-l			    监听模式, 入站链接
	-n			    只使用 IP 地址不通过 DNS解析
	-o file			使用十六进制
	-p port			本地端口号
	-r			    跟随本地和端口
	-q secs			quit after EOF on stdin and delay of secs
	-s addr			本地源地址
	-T tos			set Type Of Service
	-t			    使用telnet连接
	-u			    UDP 模式
	-v			    verbose [use twice to be more verbose]
	-w secs			连接目标网络超时时间
	-C			    Send CRLF as line-ending
	-z			    zero-I/O mode [used for scanning]
端口号可以是一个或者是一段: lo-hi [inclusive];
在端口名称处必须使用反斜杠 (e.g. 'ftp\-data').
```

我用的是GNU的netcat,比起@stake公司的netcat多了-c 选项,不过这是很有用的一个选项,后面我们会讲到.还有GNU的-L,-t ,-T选项和@stake的-L -t 用途是不一样的,自己琢磨吧!

#### 当NC作为客户端使用常用命令：

> root:~# nc -v               //端口扫描
> root:~# mtr 200.106.0.20    //追踪一下路由
> root:~# nc -vn              //显示详细的终端信息，不会Dns解析（不建议使用nc去域名解析）
> root:~# ping pop3. 163.com    	//连接ip邮箱

`nc -vn 123.125.50.29 110` 连接163`pop3`服务，从服务器返回的信息[Welcome ]可以看出163Email服务器用的是coremail来实现的服务

~~~shell
kali@kali:~$ nc -vn 123.125.50.29 110
(UNKNOWN) [123.125.50.29] 110 (pop3) open
+OK Welcome to coremail Mail Pop3 Server (163coms[b62aaa251425b4be4eaec4ab4744cf47s])
~~~

当连接成功后可以用命令行操作（在`pop3`里面命令行操作登录时需要`base64`）

```shell
kali@kali:~$ base64
baidu@163.com
MTMwOTE5NDA****TYzLmNvbQo=  //输入完按ctrl+d
```

在回到`pop3`

```shell
(UNKNOWN) [123.125.50.29] 110 (pop3) open
+OK Welcome to coremail Mail Pop3 Server (163coms[b62aaa251425b4be4eaec4ab4744cf47s])
USER MTMwOTE5NDA4NDBAMTYzLmNvbQo= //演示虚帐号拟
+OK core mail
PASSWORD NTE4NTE4aHVpCg==  //密码
-ERR Unable to log on //显示无法登录，只是演示具体操作
```

`nc -nv 123.125.20.138 25`用nc连接163`smtp`服务，用`ping`命令解析到IP（不要用nc做域名解析）

````shell
kali@kali:~$ ping smtp.163.com
PING smtp.163.com (220.181.12.14) 56(84) bytes of data.
````

连接到`smtp`服务返回的Banner信息,`Anti-spam GT反垃圾邮件网管`这里就不过多操作了

```shell
kali@kali:~$ nc -nv 220.181.12.13 25
(UNKNOWN) [220.181.12.13] 25 (smtp) open //25端口开放
220 163.com Anti-spam GT for Coremail System (163com[20141201])
ehlo admin 
250-mail
250-PIPELINING
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN PLAIN
250-coremail 1Uxr2xKj7kG0xkI17xGrU7I0s8FY2U3Uj8Cz28x1UUUUU7Ic2I0Y2Urit98DUCa0xDrUUUUj
250-STARTTLS
250 8BITMIME
```

如果你现在有一个http的服务器,也可以用nc去连接服务器的80端口,通过一些html命令去实现某些功能

```html
kali@kali:~$ nc -nv 149.28.36.251 80
(UNKNOWN) [149.28.36.251] 80 (http) open
head /
HTTP/1.1 400 Bad Request
Server: nginx
Date: Mon, 18 Jun 2018 13:31:12 GMT
Content-Type: text/html
Content-Length: 166
Connection: close

<html>
<head><title>400 Bad Request</title></head>
<body bgcolor="white">
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

#### NC-----传输文本信息

实现文本信息传输首先需要两台安装有nc的操作系统,且得有一台先开放一个端口

- 假设A_kali:192.168.31.21
- 假设：B_kali:192.168.31.134

使用A_kali开启一个侦探端口

```shell
root@kali:~# nc -l -p 555
root@kali:~# netstat -pantu | grep 555 //netstat查看端口是否开启
tcp        0      0 0.0.0.0:555             0.0.0.0:*               LISTEN      3588/nc
```

使用B_kali去连接A_kali

```she
root@kali:~# nc -l -p 333       //打开端口333
root@kali:~# netstat -pantu | grep 333     //查看端口33是否打开
```

A_kali

![](/image/2018-06-18-223126_776x92_scrot.png)

B_kali

![](/image/2018-06-18-223008_736x135_scrot.png)

其实用nc文本信息传输跟我们渗透测试有什么关系呢？去实现聊天功能那是说不通的,使用nc去实现文本信息主要是用于电子取证某些应用场景下去实现文本信息传输（后面会有单独-电子取证课）

##### 远程电子取证

- kali 32Bit

```
root@kali:~# nnc -l -p 333       //打开端口333
```

- kali 64Bit


```
root:~# ls -l | nc -nv 10.1.1.12 333
(UNKNOWN) [10.1.1.12] 333 (?) open

root:~# ps aux              //查看可疑的进程
```

- kali 32Bit


root@kali:~# nc -l -p 333 > ps.txt   监听333端口有信息重定向到ps.txt文件里

- kali 64Bit


```
root:~# ps aux | nc -nv 10.1.1.12 333 -q 1
(UNKNOWN) [10.1.1.12] 333 (?) open
```

- kali 32Bit


```
root@kali:~# cat ps.txt         //查看ps.txt文件
```

- kali 32Bit


```
root@kali:~# nc -l -p 333 > lsof.txt
```

- kali 64Bit


```
root:~# lsof | nc -nv 10.1.1.12 333 -q 1
(UNKNOWN) [10.1.1.12] 333 (?)  open
```

- kali 32Bit

```
root@kali:~# more lsof.txt    //这个文件比较多，所以用more查看文件
```

