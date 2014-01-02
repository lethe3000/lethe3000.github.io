---
layout: post
title: "ssh常用用法"
date: 2014-01-02 21:30
comments: true
categories: 
---

ssh即**secure shell protocol**
------------------------------
命令基本格式为:
{% codeblock lang=sh %}
ssh [-l login_name] [-p port] [user@]hostname
{% endcodeblock %}

其中指定用户的参数`-l`相当于`user@`
{% codeblock lang=sh %}
ssh -l root 192.168.0.11
ssh root@192.168.0.11
{% endcodeblock %}

**ssh协议**会将数据进行加密传输，使用非对称加密算法。
public key可公开，供其他主机加密数据；private key本机持有，用于解密被public key加密的数据。

ssh的连接方式
------------
1. 服务器启动sshd服务时，尝试寻找/etc/ssh_host*(mac),里面是各种公钥私钥数据。
2. 用户端通过ssh等连接到服务器
3. 服务器把公钥传送给用户端，这里公钥是明文传输
4. 用户获取到公钥之后，会查找~/.ssh/known_hosts文件中是否已经存在该公钥，如果ok则传回用户端的公钥
5. 现在连接已经建立，用户端和服务器分别持有自己的私钥和对方的公钥，所以称为非对称加密
6. 双方开始数据通信，公钥加密，私钥解密

重启sshd服务
-----------
{% codeblock lang=sh %}
[root@www ~]# /etc/init.d/sshd restart
[root@www ~]# netstat -tlnp | grep ssh
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address  Foreign Address  State   PID/Program name
tcp        0      0 :::22          :::*             LISTEN  1539/sshd
{% endcodeblock %}

公钥记录**~/.ssh/known_hosts**
-----------------------------
用户端在登录拿到服务器的公钥的时候，会首先在这个文件里去查找是否已经存在。有两种异常情况：
1. 公钥尚未记录，则提示是否记录，确认之后写入known_hosts文件并继续
2. 拿到公钥之后，与known_hosts中记录的不一致，有可能是服务器的域名-ip发生了变化，也可能是受到了攻击，这里会给出warning，用户需要确认之后移除对应的公钥并重新连接。
known_hosts的文件格式为：
`[domain],[ip] ssh-rsa [public-key]`

sshd详细配置
-----------
mac下配置文件位于`/etc/sshd_config`，其他系统可能位于`/etc/ssh/sshd_config`
