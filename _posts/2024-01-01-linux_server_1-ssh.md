---
layout: post
title:  使用SSH服务管理远程主机
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍通过Linux系统服务器的远程登录。

* TOC
{:toc}

<!--more-->

## 概要

SSH（Secure Shell）是一种安全的远程登录协议，因其加密传输账户密码和数据信息而取代了FTP与Telnet等明文传输协议。

### 历史

- 1995年，芬兰研究员Tatu Ylönen设计了SSH协议的第一个版本（SSH 1），并在大学遭受密码嗅探攻击后开发出SSH1。
- SSH 1迅速普及，1995年底已拥有20,000名用户，并成为IETF的标准文档。
- 1995年底，Ylönen成立了SCS公司，SSH从免费软件转为商业软件。
- 1996年推出SSH 2协议，以解决SSH 1的安全漏洞，但官方SSH2软件为专有软件。
- 1999年，OpenSSH项目启动，基于SSH 1.2.12版本开发SSH 2的开源实现，并迅速独立发展。
- OpenSSH随OpenBSD 2.6发布后，移植到其他操作系统，成为主流的SSH实现，广泛应用于Linux发行版。

### SSH架构

- **架构模式**：SSH采用服务器-客户端模式。
- **软件组成**：客户端软件（如`ssh`）发起连接请求；服务器端软件（如`sshd`）接收并处理请求。
- **辅助工具**：包括`ssh-keygen`、`ssh-agent`等用于密钥管理和代理。
- **客户端工具**：如`scp`、`sftp`用于文件传输。

### 安全验证方式

- **基于口令的验证**：使用账户和密码进行身份验证。
- **基于密钥的验证**：通过在本地生成密钥对并将公钥上传到服务器进行验证，更为安全。

## 基本用法

- **登录命令**：`ssh hostname`，其中`hostname`可以是域名、IP地址或局域网内的主机名。
- **指定用户名**：`ssh user@hostname`或`ssh -l username hostname`。
- **指定端口**：`ssh -p portnumber hostname`，其中`portnumber`是服务器上SSH服务监听的端口号，默认为22。

## SSH加密机制

在 SSH 联机过程中，客户端与服务器端会交换公钥，确保数据传输的安全性。

1. **服务器建立公钥档**：每当启动 SSH 服务时，服务会检查 `/etc/ssh/ssh_host*` 文件夹中的公钥文件。如果没有这些文件，服务会自动生成所需的公钥文件和私钥文件。
   
2. **客户端主动联机要求**：客户端使用 ssh 或其他客户端程序发起连接请求。
   
3. **服务器传送公钥档给客户端**：服务器将公钥文件发送给客户端。
   
4. **客户端记录/比对服务器的公钥数据**：首次连接时，客户端会在用户的家目录下 `.ssh/known_hosts` 文件中记录服务器的公钥数据。如果之前有过记录，则会比对新旧公钥数据是否一致。
   
5. **客户端生成公私钥对**：客户端随机生成自己的公私钥对，并将公钥发送给服务器。
   
1. **双向加解密**：服务器和客户端使用对方的公钥加密数据，接收方使用自己的私钥解密数据。

## 名词解释

- **SSH**：SSH 为Secure SHell protocol 的简写 (安全的壳程序协议)，由 IETF 的网络小组（Network Working Group）所制定；它是建立在应用层基础上的安全协议。
- **数据加密**是指将原始电子数据通过特定算法处理成乱码形式，使得即便数据在网络上被截获也无法轻易解读其内容。加密技术的核心是使用“非对称密钥系统”，即公钥和私钥的组合。
- **公钥**：用于加密数据，可以公开共享。
- **私钥**：用于解密数据，必须保密。


## OpenSSH安装

```shell
# OpenSSH 的客户端
[user@c1 ~]$ yum install -y openssh-clients
# OpenSSH 的服务器
[user@c1 ~]$ yum install -y openssh-server
# ssh-keygen创建公钥-私钥对，ssh-copy-id发送公钥
[user@c1 ~]$ yum install -y ssh-keygen ssh-copy-id 
```

## 启动ssh服务

```shell
# 启动服务
$ sudo systemctl start sshd.service

# 停止服务
$ sudo systemctl stop sshd.service

# 重启服务
$ sudo systemctl restart sshd.service

# 自动启动运行
$ sudo systemctl enable sshd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/sshd.service to /usr/lib/systemd/system/sshd.service

# 重新加载 systemd 管理器的配置
$ sudo systemctl daemon-reload
```

## 制作不用密码可立即登入的 ssh 用户

NIS服务器所有主机共享相同的用户和密码，即所有主机的用户信息一致，只要把免密秘钥放到用户自己的配置文件内即可实现节点间免密登录，所以仅需在任意节点每个用户内操作以下一次即可

### 1. ssh-keygen创建公钥-私钥对

```shell
#  -t~密钥类型  -f~密钥文件路径及名称  -C~ 备注信息
[user@c1 ~]$ ssh-keygen -t rsa  -f ~/.ssh/id_rsa -C "root"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):  # 输入密码, 若不输入则直接回车
Enter same passphrase again: # 再次确认密码, 若不输入则直接回车
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
9a:e3:94:b9:69:c8:e9:68:4b:dc:fa:43:25:7f:53:f1 root
The keys randomart image is:
+--[ RSA 2048]----+
|                 |
|          .      |
|           o     |
|    . .   . E    |
|     +  S.       |
| . .. .=o        |
|  oo.oB. .       |
| ..o=o.+         |
| .++oo+          |
+-----------------+

# 切换到.ssh目录
[root@c1 ~]# cd .ssh

# 生成的文件以id_rsa开头, id_rsa是私钥, id_rsa.pub是公钥:
[root@c1 .ssh]# ls
id_rsa  id_rsa.pub

# 通过cat命令查看公钥文件: 
[root@c1 .ssh]# cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2JpLMqgeg9jB9ZztOCw0WMS8hdVpFxthqG1vOQTOji/cp0+8RUZl3P6NtzqfHbs0iTcY0ypIJGgx4eXyipfLvilV2bSxRINCVV73VnydVYl5gLHsrgOx+372Wovlanq7Mxq06qAONjuRD0c64xqdJFKb1OvS/nyKaOr9D8yq/FxfwKqK7TzJM0cVBAG7+YR8lc9tJTCypmNXNngiSlipzjBcnfT+5VtcFSENfuJd60dmZDzrQTxGFSS2J34CuczTQSsItmYF3DyhqmrXL+cJ2vjZWVZRU6IY7BpqJFWwfYY9m8KaL0PZ+JJuaU7ESVBXf6HJcQhYPp2bTUyff+vdV root@c1
```

### 2. ssh-copy-id把A的公钥发送给B

```shell
# 把c1的公钥发送给192.168.1.102
[root@c1 ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.102
```

## 软件教程

### 配置文件

#### 系统配置文件

| 文件                                | 描述                                                                                                             |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------|
| `/etc/ssh/moduli`                 | 包含用于Diffie-Hellman密钥交换的Diffie-Hellman组，这对于构建安全的传输层至关重要。在SSH会话开始时交换密钥时，会创建一个共享的秘密值，该值无法单独由任何一方确定。随后，该值用于提供主机认证。 |
| `/etc/ssh/ssh_config`             | 默认的SSH客户端配置文件。请注意，如果存在`~/.ssh/config`，它将覆盖此文件。                                                                 |
| `/etc/ssh/sshd_config`            | `sshd`守护进程的配置文件。                                                                                               |
| `/etc/ssh/ssh_host_ecdsa_key`     | `sshd`守护进程使用的ECDSA私钥。                                                                                          |
| `/etc/ssh/ssh_host_ecdsa_key.pub` | `sshd`守护进程使用的ECDSA公钥。                                                                                          |
| `/etc/ssh/ssh_host_rsa_key`       | `sshd`守护进程用于SSH协议版本2的RSA私钥。                                                                                    |
| `/etc/ssh/ssh_host_rsa_key.pub`   | `sshd`守护进程用于SSH协议版本2的RSA公钥。                                                                                    |
| `/etc/pam.d/sshd`                 | `sshd`守护进程的PAM配置文件。                                                                                            |
| `/etc/sysconfig/sshd`             | `sshd`服务的配置文件。                                                                                                 |

#### 用户配置文件

| 文件                       | 描述                                                         |
|--------------------------|------------------------------------------------------------|
| `~/.ssh/authorized_keys` | 存储授权的公钥列表，用于服务器端。当客户端连接到服务器时，服务器通过检查存储在此文件中的签名公钥来验证客户端的身份。 |
| `~/.ssh/id_ecdsa`        | 包含用户的ECDSA私钥。                                              |
| `~/.ssh/id_ecdsa.pub`    | 用户的ECDSA公钥。                                                |
| `~/.ssh/id_rsa`          | `ssh`客户端用于SSH协议版本2的RSA私钥。                                  |
| `~/.ssh/id_rsa.pub`      | `ssh`客户端用于SSH协议版本2的RSA公钥。                                  |
| `~/.ssh/known_hosts`     | 存储用户访问过的SSH服务器的主机密钥。这个文件对于确保SSH客户端连接到正确的SSH服务器非常重要。        |

### 命令使用

| 命令          | 作用         |
|-------------|------------|
| ssh-keygen  | 查看系统中可用的软件 |
| ssh-copy-id | 查看系统中可用的软件 |

### 脚本教程

| 参数                                | 说明                                                               | 备注  |
|-----------------------------------|------------------------------------------------------------------|-----|
| Port 22                           | 默认的 sshd 服务端口                                                    |
| ListenAddress 0.0.0.0             | 设定 sshd 服务器监听的 IP 地址                                             |
| Protocol 2                        | SSH 协议的版本号                                                       |
| HostKey /etc/ssh/ssh_host_key     | SSH 协议版本为 1 时，DES 私钥存放的位置                                        |
| HostKey /etc/ssh/ssh_host_rsa_key | SSH 协议版本为 2 时，RSA 私钥存放的位置                                        |
| HostKey /etc/ssh/ssh_host_dsa_key | SSH 协议版本为 2 时，DSA 私钥存放的位置                                        |
| PermitRootLogin yes               | 设定是否允许 root 管理员直接登录                                              |
| StrictModes yes                   | 当远程用户的私钥改变时直接拒绝连接                                                |
| MaxAuthTries 6                    | 最大密码尝试次数                                                         |
| MaxSessions 10                    | 最大终端数                                                            |
| PasswordAuthentication yes        | 是否允许密码验证                                                         |
| PermitEmptyPasswords no           | 是否允许空密码登录（很不安全） 在 RHEL 7 系统中，已经默认安装并启用了 sshd 服务程序。接下来使用 ssh 命令进行 |

## 问题集合

### bad permissions

1. 通过ssh-copy-id命令拷贝公钥文件之后, 尝试免密登录另一台服务器时发生错误:

```shell
[root@localhost ~]# ssh root@172.16.22.131
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/root/.ssh/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
bad permissions: ignore key: /root/.ssh/id_rsa
root@172.16.22.131's password: 
```

2. 问题原因:

提示信息说明密钥文件不受保护, 具体是说这里的密钥文件权限是0644, 而数字签名机制要求密钥文件不能被其他用户访问(读取), 所以该密钥文件被强制忽略处理了.

3. 问题解决:

只要修改该密钥文件的权限即可:

`chmod 600 /root/.ssh/id_rsa`
这里`/root/.ssh/id_rsa`就是warning里给出的密钥文件名.


## Additional Resources

### Documentation

1. [SSH 教程](https://wangdoc.com/ssh/)
2. [Chapter 12. OpenSSH](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-openssh#s2-ssh-beyondshell-tcpip)
3. [第十一章、远程联机服务器SSH / XDMCP / VNC / RDP](http://cn.linux.vbird.org/linux_server/0310telnetssh_2.php)

### Useful Websites

1. [Linux - 配置SSH免密通信 - “ssh-keygen”的基本用法](https://www.cnblogs.com/shoufeng/p/11022258.html)
2. [/etc/ssh/sshd_config的 PasswordAuthentication PermitRootLogin ClientAliveCountMax ClientAliveInterval](https://blog.csdn.net/kfepiza/article/details/127781345)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
3. [鸟哥的 Linux 私房菜 -- 服务器架设篇 第三版](http://cn.linux.vbird.org/linux_server/)
