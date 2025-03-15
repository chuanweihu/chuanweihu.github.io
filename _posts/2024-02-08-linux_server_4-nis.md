---
layout: post
title:  使用NIS服务管理身份信息
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍Linux系统服务器的NIS服务。

* TOC
{:toc}

<!--more-->

## 引言

### 什么是 NIS？

在 UNIX 环境中，NIS表示网络信息服务 (Network Information Services, NIS)，是集中管理身份和身份验证的常用方法。

NIS最初命名为 Yellow Pages (YP)，集中管理身份验证和身份信息，例如：

- 用户和密码
- 主机名和 IP 地址
- POSIX 组

### NIS 的历史与发展

NIS最初由 `Sun Microsystems` 开发，用于 UNIX® (最初是 `SunOS™`) 系统的集中管理。 

目前，它基本上已经成为了业界标准，所有主流的类 `UNIX®` 系统 (`Solaris™`, HP-UX, AIX®, Linux, NetBSD, OpenBSD, FreeBSD, 等等) 都支持 NIS。

NIS 也就是人们所熟知的黄页(Yellow Pages)，但由于商标的问题，Sun 将其改名为现在的名字。 旧的术语 (以及 yp)，仍然经常可以看到，并被广泛使用。

这是一个基于 RPC 的客户机/服务器系统，它允许在一个 NIS 域中的一组机器共享一系列配置文件。 这样，系统管理员就可以配置只包含最基本配置数据的 NIS 客户机系统，并在单点上增加、 删除或修改配置数据。

尽管实现的内部细节截然不同，这和 `Windows NT®`域系统非常类似，以至于可以将两者的基本功能相互类比。

### NIS 的应用场景

- **用户管理**: 在大型企业网络中，NIS 可以集中管理所有用户的账号信息，包括用户名、密码哈希值、用户主目录等，使得管理员可以在一台服务器上更新用户信息，而无需在每个单独的客户端上进行更改。
- **组管理**: NIS 支持集中管理用户组信息，便于控制用户对资源的访问权限。
- **配置管理**: 可以通过 NIS 分发和管理网络中的各种配置信息，如主机名映射、打印队列配置等。
- **自动化管理**: 在数据中心环境中，NIS 可以简化服务器集群中的用户账户同步，保证所有节点上的用户信息一致。
- **负载均衡**: 通过 NIS 集中管理的服务可以更容易地实现负载均衡，确保用户可以从最近或最不繁忙的服务器上获取资源。
- **跨平台兼容性**: NIS 可以在不同的操作系统之间共享用户信息，适用于包含多种操作系统（如 Linux、Unix、macOS）的混合环境。
- **与现代目录服务的集成**: 在一些场景下，NIS 可以与 LDAP（轻量目录访问协议）或 Active Directory 等现代目录服务结合使用，为用户提供统一的身份验证体验。
- **支持旧版系统**: 对于仍在使用旧版 UNIX 或其他遗留系统的组织而言，NIS 可以提供一种成本效益高的用户信息管理解决方案。
- **过渡期使用**: 在从旧系统迁移到新系统的过程中，NIS 可以作为临时解决方案，确保在迁移期间用户信息的一致性和可用性。

## NIS 的工作原理

在 NIS 环境中，有三种类型的主机： 主服务器，从服务器，以及客户机。 服务器的作用是充当主机配置信息的中央数据库。 主服务器上保存着这些信息的权威副本，而从服务器则是保存这些信息的冗余副本。 客户机依赖于服务器向它们提供这些信息。

许多文件的信息可以通过这种方式来共享。 通常情况下，master.passwd、 group，以及 hosts 是通过 NIS 分发的。 无论什么时候，如果客户机上的某个进程请求这些本应在本地的文件中的资料的时候，它都会向所绑定的 NIS 服务器发出请求，而不使用本地的版本。

所有的 NIS 信息的正规版本，都被保存在一台单独的称作 NIS 主服务器的机器上。 用于保存这些信息的数据库，称为 NIS 映射(map)。 在 CentOS 中，这些映射被保存在 `/var/yp/[domainname]` 里，其中 `[domainname]` 是提供服务的 NIS 域的名字。 一台 NIS 服务器，可以同时支持多个域，因此可以建立很多这样的目录，所支撑一个域对应一个。 每一个域都会有一组独立的映射。

NIS 主和从服务器，通过 `ypserv` 服务程序来处理所有的 NIS 请求。 `ypserv` 有责任接收来自 NIS 客户机的请求，翻译请求的域，并将名字映射为相关的数据库文件的路径，然后将来自数据库的数据传回客户机。

- **NIS 主服务器**
  - 这台服务器，和 `Windows NT®` 域控制器类似，会维护所有 NIS 客户机所使用的文件 
  - `passwd`，`group`，以及许多其他 NIS 客户机所使用的文件，都被存放到主服务器上
- **NIS 从服务器**
  - 这一概念，与 `Windows NT®` 的备份域控制器类似
  - NIS 从服务器，用于维护 NIS 主服务器的数据文件副本
  - NIS 从服务器提供了一种冗余，这在许多重要的环境中是必需的
  - 此外，它也帮助减轻了主服务器的负荷： NIS 客户机总是挂接到最先响应它们的 NIS 服务器上，而这也包括来自从服务器的响应。
- **NIS 客户机**
  - NIS 客户机，和多数 `Windows NT®` 工作站类似，通过 NIS 服务器来完成登录时的身份验证过程。

## NIS的工作流程

NIS服务的应用结构分为NIS服务端和NIS客户端两种角色，NIS服务端集中维护用户的帐号信息（数据库）供NIS客户机进行查询，用户登录任何一台NIS客户端设备都会从NIS服务端进行登录认证，可实现用户帐号的集中管理。

1. Nis Master先将帐号密码相关文件制作成数据库文件
2. 若有帐号密码变动时，需要重新制作数据库文件并重新同步Master/Slave
3. NIS client 若有登入需求时，会先查询其本机的 /etc/passwd, /etc/shadow 等档案
4. 若在 NIS Client 本机找不到相关的账号数据，才开始向整个 NIS 网域的主机广播查询
5. 每部 NIS server (不论 master/slave) 都可以响应，基本上是『先响应者优先』

## NIS 环境规划

假定您正在管理大学中的一个小型实验室。 在这个实验室中，有 15 台 CentOS 机器，目前尚没有集中的管理点； 每一台机器上有自己的 /etc/passwd 和 /etc/master.passwd。 这些文件通过人工干预的方法来保持与其他机器上版本的同步； 目前，如果您在实验室中增加一个用户，将不得不在所有 15 台机器上手工执行 adduser 命令。 毋庸置疑，这一现状必须改变，因此您决定将整个实验室转为使用 NIS，并使用两台机器作为服务器。

因此，实验室的配置应该是这样的：

| 机器名         | IP 地址               | 机器的角色    |
|-------------|---------------------|----------|
| `ellington` | `192.168.1.102`     | NIS 主服务器 |
| `coltrane`  | `192.168.1.103`     | NIS 从服务器 |
| `basie`     | `192.168.1.104`     | 教员工作站    |
| `bird`      | `192.168.1.105`     | 客户机      |
| `cli[1-4]`  | `192.168.1.10[6-9]` | 其他客户机    |

### 选择 NIS 域名

这可能不是您过去使用的 “域名(domainname)”。 它的规范的叫法，应该是 “NIS 域名”。 当客户机广播对此信息的请求时，它会将 NIS 域的名字作为请求的一部分发出。 这样，统一网络上的多个服务器，就能够知道谁应该回应请求。 您可以把 NIS 域名想象成以某种方式相关的一组主机的名字。

一些机构会选择使用它们的 Internet 域名来作为 NIS 域名。 并不推荐这样做，因为在调试网络问题时，这可能会导致不必要的困扰。 NIS 域名应该是在您网络上唯一的，并且有助于了解它所描述的到底是哪一组机器。 例如对于 Acme 公司的美工部门，可以考虑使用 “acme-art” 这样的 NIS 域名。 在这个例子中，您使用的域名是 test-domain。

然而，某些操作系统 (最著名的是 SunOS™) 会使用其 NIS 域名作为 Internet 域名。 如果您的网络上存在包含这类限制的机器，就 必须 使用 Internet 域名来作为您的 NIS 域名。

### 服务器的物理要求

选择 NIS 服务器时，需要时刻牢记一些东西。 

NIS 的一个不太好的特性就是其客户机对于服务器的依赖程度。 如果客户机无法与其 NIS 域的服务器联系，则这台机器通常会陷于不可用的状态。 缺少用户和组信息，会使绝大多数系统进入短暂的冻结状态。 

基于这样的考虑，您需要选择一台不经常重新启动，或用于开发的机器来承担其责任。

如果您的网络不太忙，也可以使用运行着其他服务的机器来安放 NIS 服务，只是需要注意，一旦 NIS 服务器不可用，则 所有 的 NIS 客户机都会受到影响。

## NIS Master服务器的搭建

### 安装 NIS 服务器软件

#### 软件需求

由于 NIS 服务器需要使用 RPC 协议，且 NIS 服务器同时也可以当成客户端，因此它需要的软件就有底下这几个：

- `yp-tools`: 提供 NIS 相关的查寻指令功能
- `ypbind`：“绑定(bind)” NIS 客户机到它的 NIS 服务器上。 这样，它将从系统中获取 NIS 域名，并使用 RPC 连接到服务器上。 ypbind 是 NIS 环境中，客户机-服务器通讯的核心； 如果客户机上的 ypbind 死掉的话，它将无法访问 NIS 服务器。
- `ypserv`：只应在 NIS 服务器上运行它； 这是 NIS 的服务器进程。 如果 ypserv死掉的话，则服务器将不再具有响应 NIS 请求的能力 (此时，如果有从服务器的话，则会接管操作)。 有一些 NIS 的实现的客户机上，如果之前用过一个服务器，而那台服务器死掉的话，并不尝试重新连接到另一个服务器。 通常，发生这种情况时，唯一的办法就是重新启动服务器进程 (或者，甚至重新启动服务器) 或客户机上的 ypbind 进程。
- `rpcbind`：必须运行这个程序，才能够启用 RPC (远程过程调用，NIS 用到的一种网络协议)。 如果没有运行 rpcbind，则没有办法运行 NIS 服务器，或作为 NIS 客户机

#### 软件安装流程

1. 查询相关软件

```shell
# 查询yp开头的软件
[root@c1 root]# rpm -qa | grep '^yp'

# 查询rpc开头的软件
[root@c1 root]# rpm -qa | grep '^rpc'
rpcbind-0.2.0-49.el7.x86_64
```

根据查询结果，系统已经安装rpcbind软件，只需要安装yp-tools，ypbind和ypserv三个软件即可。

2. 安装`yp*`软件

```shell
# 查询安装包
[root@c1 root]# yum list yp*

# 安装相关软件
[root@c1 root]# yum install -y ypserv ypbind yp-tools
```

### 配置 NIS 服务器

#### 服务需求文件

在 NIS 服务器上最重要的就是 ypserv 这个软件了，但是，由于 NIS 设定时还会使用到其他网络参数设定数据，因此在配置文件方面需要有底下这些数据喔：

- `/etc/ypserv.conf`：这是最主要的 ypserv 软件所提供的配置文件，可以规范 NIS 客户端是否可登入的权限。
- `/etc/hosts`：由于 NIS server/client 会用到网络主机名与 IP 的对应，因此这个主机名对应档就显的相当重要！每一部主机名与 IP 都需要记录才行！
- `/etc/sysconfig/network`：可以在这个档案内指定 NIS 的网域 (nisdomainname)。
- `/var/yp/Makefile`：前面不是说账号数据要转成数据库文件吗？ 这就是与建立数据库有关的动作配置文件；

至于 NIS 服务器提供的主要服务方面有底下两个：

- `/usr/sbin/ypserv`：就是 NIS 服务器的主要提供服务；
- `/usr/sbin/rpc.yppasswdd`：提供额外的 NIS 客户端之用户密码修改服务，透过这个服务，NIS 客户端可以直接修改在 NIS 服务器上的密码。相关的使用程序则是 yppasswd 指令；

与账号密码的数据库有关的指令方面有底下几个：

- `/usr/lib64/yp/ypinit`：建立数据库的指令，非常常用 (在 32 位的系统下，文件名则是 /usr/lib/yp/ypinit 喔！)；
- `/usr/bin/yppasswd`：与 NIS 客户端有关，主要在让用户修改服务器上的密码。

#### 网络配置

1. 设定NIS的域名(`/etc/sysconfig/network`)

```shell
# 设置NIS的域名(写入文件不会立即生效)
[root@c1 ~]# nisdomainname centos
[root@c1 ~]# echo 'NISDOMAIN=centos' >> /etc/sysconfig/network

# 设置NIS服务器启动的端口号
[root@c1 ~]# echo 'YPSERV_ARGS=-p 1011' >> /etc/sysconfig/network

# 查看网络设置
[root@c1 ~]# cat /etc/sysconfig/network
NISDOMAIN=centos
YPSERV_ARGS="-p 1011"

# 查看NIS的域名
[root@c1 ~]# nisdomainname 
centos
```

2. 设定主机名和主机名解析(`/etc/hosts`和`/etc/hostname`)

```shell
# 通过命令或文件设置主机名称
[root@c1 ~]# hostnamectl set-hostname c1
[root@c1 ~]# vi /etc/hostname
c1

# 设置主机名与IP的对应
[root@c1 ~]# vi /etc/hosts
192.168.1.101 c1
192.168.1.102 c2
192.168.1.103 c3


# 查看主机名称
[root@c1 ~]# hostname

# 查看和修改主机的名称、ID、状态和版本信息
[root@c1 ~]# hostnamectl
```

#### 配置访问NIS服务器的权限

> `ypserv.conf`文件是逐行解释执行，所以要注意设置顺序

| 参数 | 含义 |
| --- | --- |
| Host | 指定客户端，可以指定具体IP地址，也可以挃定一个网段 |
| Domain | 设置NIS域名，这里的NIS域名和DNS中的域名并没有关系。 |
| Map | 设置可用数据库名称，可以用“\*”代替所有数据库 |
| Security | 安全性设置。主要有none、port和deny三种参数设置。 |
| none | 没有任何安全限制，可以连接NIS服务器。 |
| port | 只允许小于1024以下的端口连接NIS服务器。 |
| deny | 拒绝连接NIS服务器。 |

```shell
[root@c1 ~]# vi /etc/ypserv.conf 
# NIS 服务器使用内部局域网络，只要有/etc/hosts即可，不用 DNS 啦
dns no

# 预设会有 30 个数据库被读入内存当中，其实我们的账号档案并不多，30 个够用了。
files: 30

# 与 master/slave 有关，将同步更新的数据库比对所使用的端口，放置于 <1024 内。
xfr_check_port: yes

# 底下则是设定限制客户端或 slave server 查询的权限，利用冒号隔成四部分：
# [主机名/IP] : [NIS域名] : [可用数据库名称] : [安全限制]
# [主机名/IP]   ：可以使用 network/netmask 如 192.168.100.0/255.255.255.0 
# [NIS域名]   ：例如本案例中的 vbirdnis
# [可用数据库名称]：就是由 NIS 制作出来的数据库名称；
# [安全限制]      ：包括没有限制 (none)、仅能使用 <1024 (port) 及拒绝 (deny)
# 一般来说，你可以依照我们的网域来设定成为底下的模样：

#127.0.0.0/255.255.255.0 :*:*:none
#192.168.1.101/255.255.255.0 :*:*:none
#* :*: * :deny
# 星号 (*) 代表任何数据都接受的意思。上面三行的意思是，开放 lo 内部接口、
# 开放内部 LAN 网域，且杜绝所有其他来源的 NIS 要求的意思。

# 允许所有内网客户端可以连接NIS服务器，除此之外的客户端都拒绝连接。
192.168.1.0/255.255.255.0 :*:*:none
* : *: *: deny
```

#### 设置主从模式的配置（**如果无从服务器不需要设置**）

如果是一主多从，修改`/var/yp/Makefile`中 `NOPUSH=false`。表示同步到从服务器
如果只有一台服务端服务器，确认`/var/yp/Makefile`中 `NOPUSH=true`，默认`NOPUSH=true`

```shell
# 查看主机名称
[root@c1 ~]# vi /var/yp/Makefile
...
# 同步到slave servers
NOPUSH=false

# 添加从服务器的主机名
[root@c1 ~]# vi /var/yp/ypservers
c1
c2
```

#### 设置防火墙

```shell
# 设置firewall
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-port=1011/tcp
firewall-cmd --permanent --add-port=1011/udp
firewall-cmd --permanent --add-port=1012/udp
firewall-cmd --reload

# 设置iptables
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 1011 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

# 内网直接关闭防火墙
[root@c1 ~]# systemctl stop firewalld.service
```

### 启动与测试 NIS 服务

#### 启动与观察所有相关的服务

```shell
# 修改yppasswdd服务的端口，该功能与 NIS 客户端有关，允许用户在客户端修改服务器上的密码
[root@c1 ~]# vi /etc/sysconfig/yppasswdd 
YPPASSWDD_ARGS='--port 1012'

# 启动yppasswdd服务并设置开机启动
[root@c1 ~]# systemctl start yppasswdd.service && systemctl enable yppasswdd.service

# 启动rpcbind服务并设置开机启动
[root@c1 ~]# systemctl start rpcbind.service && systemctl enable rpcbind.service

# 启动ypserv服务并设置开机启动
[root@c1 ~]# systemctl start ypserv.service && systemctl enable ypserv.service

# 查看ypserv服务状态
[root@c1 ~]# systemctl status ypserv.service 

# 查看yppasswdd服务状态
[root@c1 ~]# systemctl status yppasswdd.service 

# 查看rpcbind服务状态
[root@c1 ~]# systemctl status rpcbind.service 

# 查看所有注册rpc的服务程序及端口号
[root@c1 ~]# rpcinfo -p localhost

# 查看注册rpc中的ypserv状态
[root@c1 ~]# rpcinfo -u localhost ypserv
```

#### 处理账号并建立数据库

```shell
# 修改添加用户默认参数
[root@c1 ~]# vi /etc/default/useradd 
# useradd defaults file
GROUP=100
# 修改默认家目录
HOME=/public/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes

# 添加3个用户
[root@c1 ~]# useradd -u 1001 nisuser1
[root@c1 ~]# useradd -u 1002 nisuser2
[root@c1 ~]# useradd -u 1003 nisuser3

# 设置3个用户的初始密码
[root@c1 ~]# echo meimima | passwd --stdin nisuser1
[root@c1 ~]# echo meimima | passwd --stdin nisuser2
[root@c1 ~]# echo meimima | passwd --stdin nisuser3

# 转化帐号为数据库（-m 代表初始化主服务器）
[root@c1 ~]# /usr/lib64/yp/ypinit -m

# 查看数据库
[root@c1 ~]# ls /var/yp/centos

# 更新数据库
[root@c1 ~]# cd /var/yp/;make && systemctl restart ypserv
```

- 批量创建账号脚本

```shell
# 编写批量账号脚本
[root@c1 ~]# vi account.sh
#!/bin/bash
# 这支程序用来创建新增账号，功能有：
# 1. 检查 account.txt 是否存在，并将该文件内的账号取出；
# 2. 创建上述文件的账号；
# 3. 将上述账号的口令修订成为『强制第一次进入需要修改口令』的格式。
# 2009/03/04    VBird
export PATH=/bin:/sbin:/usr/bin:/usr/sbin:$PATH

# 检查 account.txt 是否存在`
if [ ! -f account.txt ]; then
        echo "所需要的账号文件不存在，请创建 account.txt ，每行一个账号名称"
        exit 1
fi

usernames=$(cat account.txt)

for username in $usernames
do
        useradd $username                         # <==新增账号
        echo $username | passwd --stdin $username # <==与账号相同的口令
        chage -d 0 $username                      # <==强制登陆修改口令
done

# 编写账号文档
[root@c1 ~]# vi account.txt
nisuser1
nisuser2
nisuser3
nisuser4
nisuser5
nisuser6
nisuser7
nisuser8
nisuser9
```

## NIS Slave服务器的搭建

一般只有在大型环境中，才会配置NIS 服务端为一主多从的架构。

> 需要注意两点，其他跟`NIS Master`服务端的设定一致
>> 1. 主机名设置需要跟服务器不一样
>> 2. 创建数据库时需要加上Master服务端

### 安装 NIS 服务器软件

> 同master服务器一致

```shell
# 安装相关软件
[root@c2 ~]# yum install -y ypserv ypbind yp-tools
```

### 配置 NIS 服务器

> 同master服务器类似

#### 设定NIS的域名(`/etc/sysconfig/network`)

```shell
# 设置NIS的域名(写入文件不会立即生效)
[root@c1 ~]# echo 'NISDOMAIN=centos' >> /etc/sysconfig/network

# 设置NIS服务器启动的端口号
[root@c1 ~]# echo 'YPSERV_ARGS=-p 1011' >> /etc/sysconfig/network

# 查看网络设置
[root@c1 ~]# cat /etc/sysconfig/network
NISDOMAIN=centos
YPSERV_ARGS="-p 1011"

# 查看NIS的域名
[root@c1 ~]# nisdomainname 
centos
```

#### 设定主机名和主机名解析(`/etc/hosts`和`/etc/hostname`)

```shell
# 通过命令或文件设置主机名称
[root@c2 ~]# hostnamectl set-hostname c2
[root@c2 ~]# vi /etc/hostname
c2

# 设置主机名与IP的对应
[root@c2 ~]# vi /etc/hosts
192.168.1.101 c1
192.168.1.102 c2
192.168.1.103 c3


# 查看主机名称
[root@c2 ~]# hostname

# 查看和修改主机的名称、ID、状态和版本信息
[root@c2 ~]# hostnamectl
```

#### 配置访问NIS服务器的权限(`/etc/ypserv.conf`)

```shell
[root@c2 ~]# vi /etc/ypserv.conf 
# ...
# 允许所有内网客户端可以连接NIS服务器，除此之外的客户端都拒绝连接。
192.168.1.0/255.255.255.0 :*:*:none
* : *: *: deny
```

#### 设置主从模式的配置

从服务器需要确认`/var/yp/Makefile`中 `NOPUSH=true`，默认`NOPUSH=true`

```shell
# 查看主机名称
[root@c2 ~]# vi /var/yp/Makefile
...
NOPUSH=true
```

#### 设置防火墙

```shell
# 设置firewall
[root@c2 ~]# firewall-cmd --permanent --add-service=rpc-bind
[root@c2 ~]# firewall-cmd --permanent --add-port=1011/tcp
[root@c2 ~]# firewall-cmd --permanent --add-port=1011/udp
[root@c2 ~]# firewall-cmd --permanent --add-port=1012/udp
[root@c2 ~]# firewall-cmd --reload

# 设置iptables
[root@c2 ~]# iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 1011 -j ACCEPT
[root@c2 ~]# iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

# 内网直接关闭防火墙
[root@c2 ~]# systemctl stop firewalld.service
```

### 启动与测试 NIS 服务

> 同master服务器类似

#### 启动与观察所有相关的服务

```shell
# 修改yppasswdd服务的端口，该功能与 NIS 客户端有关，允许用户在客户端修改服务器上的密码
[root@c2 ~]# vi /etc/sysconfig/yppasswdd 
YPPASSWDD_ARGS='--port 1012'

# 启动yppasswdd服务并设置开机启动
[root@c2 ~]# systemctl start yppasswdd.service && systemctl enable yppasswdd.service

# 启动rpcbind服务并设置开机启动
[root@c2 ~]# systemctl start rpcbind.service && systemctl enable rpcbind.service

# 启动ypserv服务并设置开机启动
[root@c2 ~]# systemctl start ypserv.service && systemctl enable ypserv.service

# 查看ypserv服务状态
[root@c2 ~]# systemctl status ypserv.service 

# 查看yppasswdd服务状态
[root@c2 ~]# systemctl status yppasswdd.service 

# 查看rpcbind服务状态
[root@c2 ~]# systemctl status rpcbind.service 

# 查看所有注册rpc的服务程序及端口号
[root@c2 ~]# rpcinfo -p localhost

# 查看注册rpc中的ypserv状态
[root@c2 ~]# rpcinfo -u localhost ypserv
```

#### 建立数据库

```shell
# 根据master的数据库初始化从服务器（-s代表初始化从服务器，后面跟上主服务器的hostname）
[root@c2 ~]# /usr/lib64/yp/ypinit -s c1

# 查看数据库
[root@c2 ~]# ls /var/yp/centos
```

## NIS Client客户端的搭建

### 安装 NIS 客户端

#### 软件需求

由于 NIS 服务器需要使用 RPC 协议，NIS 客户端需要的软件就有底下这几个：

- `yp-tools`: 提供 NIS 相关的查寻指令功能
- `ypbind`：“绑定(bind)” NIS 客户机到它的 NIS 服务器上。 这样，它将从系统中获取 NIS 域名，并使用 RPC 连接到服务器上。 ypbind 是 NIS 环境中，客户机-服务器通讯的核心； 如果客户机上的 ypbind 死掉的话，它将无法访问 NIS 服务器。
- `rpcbind`：必须运行这个程序，才能够启用 RPC (远程过程调用，NIS 用到的一种网络协议)。 如果没有运行 rpcbind，则没有办法运行 NIS 服务器，或作为 NIS 客户机

#### 软件安装流程

1. 查询相关软件

```shell
# 查询yp开头的软件
[root@c3 user]# rpm -qa | grep '^yp'

# 查询rpc开头的软件
[root@c3 user]# rpm -qa | grep '^rpc'
rpcbind-0.2.0-49.el7.x86_64
```

根据查询结果，系统已经安装rpcbind软件，只需要安装yp-tools，ypbind软件即可。

2. 安装`yp*`软件

```shell
# 查询安装包
[root@c3 user]# yum list yp*

# 安装相关软件
[root@c3 user]# yum install -y ypbind yp-tools
```

### 配置 NIS 客户端

#### 服务需求文件

在 CentOS 当中我们还有很多配置文件是与认证有关的，包含 `ypbind` 的配置文件时，在设定 NIS client 你可能需要动到底下的档案：

- `/etc/sysconfig/network` (加入 NISDOMAIN 项目)：就是 NIS 的领域名嘛！
- `/etc/hosts`：至少需要有各个 NIS 服务器的 IP 与主机名对应；
- `/etc/yp.conf`(ypbind的配置文件)：这个则是 ypbind 的主要配置文件，里面主要设定 NIS 服务器所在
- `/etc/sysconfig/authconfig`(CentOS 的认证机制)：规范账号登入时的允许认证机制；
- `/etc/pam.d/system-auth` (许多登入所需要的 PAM 认证过程)：这个最容易忘记！因为账号通常由 PAM 模块所管理，所以你必须要在 PAM 模块内加入 NIS 的支持才行！
- `/etc/nsswitch.conf`（修改许多主机验证功能的顺序）：这个档案可以规范账号密码与相关信息的查询顺序，默认是先找 /etc/passwd 再找 NIS 数据库；

另外，NIS 还提供了几个有趣的程序给 NIS 客户端来进行账号相关参数的修改，例如密码、shell 等等，主要有底下这几个指令：

- `/usr/bin/yppasswd` ：更改你在NIS database (NIS Server 所制作的数据库) 的密码
- `/usr/bin/ypchsh`   ：同上，更改shell
- `/usr/bin/ypchfn`   ：同上，更改一些用户的讯息！

### 网络配置

#### 设定主机名和主机名解析(`/etc/hosts`和`/etc/hostname`)

```shell
# 通过命令或文件设置主机名称
[root@c3 ~]# hostnamectl set-hostname c3
[root@c3 ~]# vi /etc/hostname
c3

# 设置主机名与IP的对应
[root@c3 ~]# vi /etc/hosts
192.168.1.101 c1
192.168.1.102 c2
192.168.1.103 c3

# 查看主机名称
[root@c3 ~]# hostname

# 查看和修改主机的名称、ID、状态和版本信息
[root@c3 ~]# hostnamectl
```

#### 设置防火墙

```shell
# 设置firewall
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-port=1011/tcp
firewall-cmd --permanent --add-port=1011/udp
firewall-cmd --permanent --add-port=1012/udp
firewall-cmd --reload

# 设置iptables
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 1011 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

# 内网直接关闭防火墙
[root@c3 ~]# systemctl stop firewalld.service
```

#### 服务设置

- 通过setup管理工具设置
    - 输入 setup命令
    - 选择『Authentication configuration』的项目
    - 选择『Use NIS』项目
    - 填写NIS域名-(Domain: `centos`) 以及 NIS服务器的IP-(Server: `192.168.1.101`)，按下确认即可
    - 跳回原来界面即成功

```shell
[root@c3 ~]# setup
# Authentication configuration
# Use NIS
# Domain: centos
# Server: 192.168.1.101
```

### 检查服务

#### 检查配置文件

```shell
# 查看网络设置
[root@c3 root]# cat /etc/sysconfig/network
# 查看ypbind 的主要配置文件
[root@c3 root]# cat /etc/yp.conf 
# 查看账号密码与相关信息的查询顺序
[root@c3 root]# vi /etc/nsswitch.conf
# 查看账号登入时的允许认证机制
[root@c3 root]# cat /etc/sysconfig/authconfig 
# 查看账号PAM 认证过程
[root@c3 root]# cat /etc/pam.d/system-auth
```

#### NIS client 端的检验

```shell
# 查看所有注册rpc的服务程序及端口号
[root@c3 root]# rpcinfo -p localhost
# 查看NIS client的账号
[root@c3 root]# id nisuser1
[root@c3 root]# id nisuser2
[root@c3 root]# id nisuser3
# 检验数据库
[root@c3 root]# yptest 
# 检验数据库数量
[root@c3 root]# ypwhich -x
# 读取数据库内容
[root@c3 root]# ypcat passwd.byname
```

#### NIS client 端的使用者参数修改

在 NIS 客户端处理账号密码的订正通过yppasswd, ypchsh, ypchfn 来处理即可。这三个指令的对应是：

- `yppasswd` ：与 passwd 指令相同功能；
- `ypchfn` ：与 chfn 相同功能（chfn (Change finger) 在Linux中用于修改用户的指纹信息，包括全名、办公室号码和电话等）；
- `ypchsh` ：与 chsh 相同功能（chsh (Change shell)用于更改 Linux 中的默认登录 shell。）。

```shell
# 查看nisuser账号
[root@c3 ~]# grep nisuser /etc/passwd

# 切换nisuser1账号
[root@c3 ~]# su - nisuser1

# 在c3客户端修改nisuser1账号的密码
[nisuser1@c3 ~]$ yppasswd
Changing NIS account information for nisuser1 on c3.
Please enter old password:
Changing NIS password for nisuser1 on c3.
Please enter new password:
Please retype new password:

The NIS password has been changed on c3.

# 查看centos域名的数据库是否更新
[root@c3 centos]# ll /var/yp/centos/
total 3152
...
-rw-------. 1 root root 137216 Dec 22 14:52 netid.byname
-rw-------. 1 root root 136704 Dec 22 15:29 passwd.byname
-rw-------. 1 root root 136704 Dec 22 15:29 passwd.byuid
-rw-------. 1 root root 142336 Dec 22 14:52 protocols.byname

# 查看日志，密码已经被修改
[root@c3 centos]# tail /var/log/messages
...
Dec 22 15:29:02 c3 rpc.yppasswdd[1483]: update nisuser1 (uid=1001) from host 192.168.1.102 successful.
```

## NIS 的安全性和性能优化

对于现代网络基础架构，NIS 被视为过于不安全，因为它既不提供主机身份验证，也不通过网络发送的数据进行加密。为了临时解决这个问题，NIS 通常与其他协议集成，以增强安全性。

如果您使用身份管理(IdM)，您可以使用 NIS 服务器插件连接无法完全迁移到 IdM 的客户端。IdM 将网络组和其他 NIS 数据集成到 IdM 域中。另外，您可以轻松地将用户和主机身份从 NIS 域迁移到 IdM。

NIS 服务器插件使 IdM 集成的 LDAP 服务器能够充当客户端的 NIS 服务器。在此角色中，目录服务器会根据配置动态生成和更新 NIS 映射。使用插件，IdM 使用 NIS 协议作为 NIS 服务器提供客户端。

在身份管理中启用 NIS：

1. 启用 NIS 侦听器和兼容性插件：

```shell
[root@ipaserver ~]# ipa-nis-manage enable
[root@ipaserver ~]# ipa-compat-manage enable
```

2. 可选 ：为 NIS 远程过程调用(RPC)设置固定端口。

在使用 NIS 时，客户端必须知道要使用的 IdM 服务器上的哪些端口来建立连接。使用默认设置，IdM 在服务器启动时绑定到未使用的随机端口。此端口发送到客户端用于请求端口号的端口映射器服务。

对于更严格的防火墙配置，您可以设置固定端口。例如，要将端口设置为 514 ：

```shell
[root@ipaserver ~]# ldapmodify -x -D 'cn=directory manager' -W
dn: cn=NIS Server,cn=plugins,cn=config
changetype: modify
add: nsslapd-pluginarg0
nsslapd-pluginarg0: 514
```

注意: 您可以在 1024 以下设置任何未使用的端口号。

3. 启用并启动端口映射器服务：

```shell
[root@ipaserver ~]# systemctl enable rpcbind.service
[root@ipaserver ~]# systemctl start rpcbind.service
```

4. 重启 Directory 服务器：

```shell
[root@ipaserver ~]# systemctl restart dirsrv.target
```

## 案例研究

假定有 3 台 CentOS 机器，将整个实验室转为使用 NIS，并使用两台机器作为服务器。

| 机器名  | IP 地址           | 机器的角色    |
|------|-----------------|----------|
| `c1` | `192.168.1.101` | NIS 主服务器 |
| `c2` | `192.168.1.102` | NIS 从服务器 |
| `c3` | `192.168.1.103` | 客户机      |

### NIS Master服务端的设置

#### 软件安装

1. 查询相关软件

```shell
# 查询yp开头的软件
[root@c1 root]# rpm -qa | grep '^yp'

# 查询rpc开头的软件
[root@c1 root]# rpm -qa | grep '^rpc'
rpcbind-0.2.0-49.el7.x86_64
```

根据查询结果，系统已经安装rpcbind软件，只需要安装yp-tools，ypbind和ypserv三个软件即可。

2. 安装`yp*`软件

```shell
# 查询安装包
[root@c1 root]# yum list yp*
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
yp-tools.x86_64       2.14-5.el7            base
ypbind.x86_64         3:1.37.1-9.el7        base
ypserv.x86_64         2.31-12.el7           base

# 安装相关软件
[root@c1 root]# yum install yp*
......
Dependencies Resolved

================================================
 Package      Arch   Version         Repository
                                           Size
================================================
Installing:
 yp-tools     x86_64 2.14-5.el7      base  79 k
 ypbind       x86_64 3:1.37.1-9.el7  base  62 k
 ypserv       x86_64 2.31-12.el7     base 156 k
Installing for dependencies:
 tokyocabinet x86_64 1.4.48-3.el7    base 459 k

Transaction Summary
================================================
Install  3 Packages (+1 Dependent package)

Total download size: 757 k
Installed size: 1.9 M
Is this ok [y/d/N]: y
```

#### 设定主机名和主机名解析

```shell
# 通过命令或文件设置主机名称
[root@c1 ~]# hostnamectl set-hostname c1
[root@c1 ~]# vi /etc/hostname
c1

# 设置主机名与IP的对应
[root@c1 ~]# vi /etc/hosts
192.168.1.101 c1
192.168.1.102 c2
192.168.1.103 c3
192.168.1.103 c4

# 查看主机名称
[root@c1 ~]# hostname
c1

# 查看主机相信息
[root@c1 ~]# hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: e9b05280560a4af4a6af98b31c11abda
           Boot ID: 0130f463c0fa4d429d117169a2e895e3
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1127.el7.x86_64
      Architecture: x86-64
```

#### 设置防火墙

```shell
# 设置firewall
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-port=1011/tcp
firewall-cmd --permanent --add-port=1011/udp
firewall-cmd --reload

# 设置iptables
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 1011 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

# 内网直接关闭防火墙
[root@c1 ~]# systemctl stop firewalld.service
```

#### 设定NIS的域名

```shell
# 设置NIS的域名(写入文件不会立即生效)
[root@c1 ~]# nisdomainname centos
[root@c1 ~]# echo 'NISDOMAIN=centos' >> /etc/sysconfig/network

# 设置NIS服务器启动的端口号
[root@c1 ~]# echo 'YPSERV_ARGS=-p 1011' >> /etc/sysconfig/network

# 查看网络设置
[root@c1 ~]# cat /etc/sysconfig/network
NISDOMAIN=centos
YPSERV_ARGS="-p 1011"

# 查看NIS的域名
[root@c1 ~]# nisdomainname 
centos
```

#### 配置访问NIS服务器的权限

```shell
[root@c1 ~]# vi /etc/ypserv.conf 
# NIS 服务器使用内部局域网络，只要有/etc/hosts即可，不用 DNS 啦
dns no

# 预设会有 30 个数据库被读入内存当中，其实我们的账号档案并不多，30 个够用了。
files: 30

# 与 master/slave 有关，将同步更新的数据库比对所使用的端口，放置于 <1024 内。
xfr_check_port: yes

# 底下则是设定限制客户端或 slave server 查询的权限，利用冒号隔成四部分：
# [主机名/IP] : [NIS域名] : [可用数据库名称] : [安全限制]
# [主机名/IP]   ：可以使用 network/netmask 如 192.168.100.0/255.255.255.0 
# [NIS域名]   ：例如本案例中的 vbirdnis
# [可用数据库名称]：就是由 NIS 制作出来的数据库名称；
# [安全限制]      ：包括没有限制 (none)、仅能使用 <1024 (port) 及拒绝 (deny)
# 一般来说，你可以依照我们的网域来设定成为底下的模样：

#127.0.0.0/255.255.255.0 :*:*:none
#192.168.1.101/255.255.255.0 :*:*:none
#* :*: * :deny
# 星号 (*) 代表任何数据都接受的意思。上面三行的意思是，开放 lo 内部接口、
# 开放内部 LAN 网域，且杜绝所有其他来源的 NIS 要求的意思。

* : *: *: deny
```

#### 设置主从模式的配置

```shell
# 查看主机名称
[root@c1 ~]# vi /var/yp/Makefile
...
# 同步到slave servers
NOPUSH=false

# 添加从服务器的主机名
[root@c1 ~]# vi /var/yp/ypservers
c1
c2
```

#### 启动与观察所有相关的服务

```shell
# 修改yppasswdd服务的端口，该功能与 NIS 客户端有关，允许用户在客户端修改服务器上的密码
[root@c1 ~]# vi /etc/sysconfig/yppasswdd 
YPPASSWDD_ARGS='--port 1012'

# 启动yppasswdd服务并设置开机启动
[root@c1 ~]# systemctl start yppasswdd.service && systemctl enable yppasswdd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/yppasswdd.service to /usr/lib/systemd/system/yppasswdd.service.

# 启动rpcbind服务并设置开机启动
[root@c1 ~]# systemctl start rpcbind.service && systemctl enable rpcbind.service

# 启动ypserv服务并设置开机启动
[root@c1 ~]# systemctl start ypserv.service && systemctl enable ypserv.service
Created symlink from /etc/systemd/system/multi-user.target.wants/ypserv.service to /usr/lib/systemd/system/ypserv.service.

# 查看ypserv服务状态
[root@c1 ~]# systemctl status ypserv.service 
  ypserv.service - NIS/YP (Network Information Service) Server
   Loaded: loaded (/usr/lib/systemd/system/ypserv.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-12-22 14:31:51 CST; 21h ago
 Main PID: 1251 (ypserv)
   Status: "Processing requests..."
    Tasks: 1
   CGroup: /system.slice/ypserv.service
           └─1251 /usr/sbin/ypserv -f -p 1011

Dec 22 14:31:50 c1 systemd[1]: Starting NIS/YP (Network Information Servic.....
Dec 22 14:31:51 c1 ypserv[1251]: WARNING: no securenets file found!
Dec 22 14:31:51 c1 systemd[1]: Started NIS/YP (Network Information Service...r.
Hint: Some lines were ellipsized, use -l to show in full.

# 查看yppasswdd服务状态
[root@c1 ~]# systemctl status yppasswdd.service 
 yppasswdd.service - NIS/YP (Network Information Service) Users Passwords Change Server
   Loaded: loaded (/usr/lib/systemd/system/yppasswdd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-12-22 14:31:51 CST; 21h ago
  Process: 1244 ExecStartPre=/usr/libexec/yppasswdd-pre-setdomain (code=exited, status=0/SUCCESS)
 Main PID: 1483 (rpc.yppasswdd)
   Status: "Processing requests..."
    Tasks: 1
   CGroup: /system.slice/yppasswdd.service
           └─1483 /usr/sbin/rpc.yppasswdd -f --port 1012

Dec 22 14:31:50 c1 systemd[1]: Starting NIS/YP (Network Information Servic.....
Dec 22 14:31:51 c1 systemd[1]: Started NIS/YP (Network Information Service...r.
Dec 22 15:29:02 c1 rpc.yppasswdd[1483]: update nisuser1 (uid=1001) from hos....
Hint: Some lines were ellipsized, use -l to show in full.

# 查看rpcbind服务状态
[root@c1 ~]# systemctl status rpcbind.service 
  rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-12-22 14:31:40 CST; 21h ago
  Process: 814 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 830 (rpcbind)
    Tasks: 1
   CGroup: /system.slice/rpcbind.service
           └─830 /sbin/rpcbind -w

Dec 22 14:31:39 c1 systemd[1]: Starting RPC bind service...
Dec 22 14:31:40 c1 systemd[1]: Started RPC bind service.

# 查看所有注册rpc的服务程序及端口号
[root@c1 ~]# rpcinfo -p localhost
   program vers proto   port  service
...
    100009    1   udp   1012  yppasswdd
    100004    2   udp   1011  ypserv
    100004    1   udp   1011  ypserv
    100004    2   tcp   1011  ypserv
    100004    1   tcp   1011  ypserv
...

# 查看注册rpc中的ypserv状态
[root@c1 ~]# rpcinfo -u localhost ypserv
program 100004 version 1 ready and waiting
program 100004 version 2 ready and waiting
```

#### 处理账号并建立数据库

```shell
# 修改添加用户默认参数
[root@c1 ~]# vi /etc/default/useradd 
# useradd defaults file
GROUP=100
# 修改默认家目录
HOME=/public/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes

# 添加3个用户
[root@c1 ~]# useradd -u 1001 nisuser1
[root@c1 ~]# useradd -u 1002 nisuser2
[root@c1 ~]# useradd -u 1003 nisuser3

# 设置3个用户的初始密码
[root@c1 ~]# echo meimima | passwd --stdin nisuser1
Changing password for user nisuser1.
passwd: all authentication tokens updated successfully.
[root@c1 ~]# echo meimima | passwd --stdin nisuser2
Changing password for user nisuser2.
passwd: all authentication tokens updated successfully.
[root@c1 ~]# echo meimima | passwd --stdin nisuser3
Changing password for user nisuser3.
passwd: all authentication tokens updated successfully.

# 转化帐号为数据库
[root@c1 ~]# /usr/lib64/yp/ypinit -m
At this point, we have to construct a list of the hosts which will run NIS
servers.  c1 is in the list of NIS server hosts.  Please continue to add
the names for the other hosts, one per line.  When you are done with the
list, type a <control D>.
	next host to add:  c1
	next host to add:
The current list of NIS servers looks like this:
c1
Is this correct?  [y/n: y]  y
We need a few minutes to build the databases...
Building /var/yp/centos/ypservers...
Running /var/yp/Makefile...
gmake[1]: Entering directory `/var/yp/centos'
Updating passwd.byname...
Updating passwd.byuid...
Updating group.byname...
Updating group.bygid...
Updating hosts.byname...
Updating hosts.byaddr...
Updating rpc.byname...
Updating rpc.bynumber...
Updating services.byname...
Updating services.byservicename...
Updating netid.byname...
Updating protocols.bynumber...
Updating protocols.byname...
Updating mail.aliases...
gmake[1]: Leaving directory `/var/yp/centos'
c1 has been set up as a NIS master server.
Now you can run ypinit -s c1 on all slave server.
```

### NIS Slave服务端的设置

#### 软件安装

1. 查询相关软件

```shell
# 查询yp开头的软件
[root@c2 ~]# rpm -qa | grep '^yp'

# 查询rpc开头的软件
[root@c2 ~]# rpm -qa | grep '^rpc'
rpcbind-0.2.0-49.el7.x86_64
```

根据查询结果，系统已经安装rpcbind软件，只需要安装yp-tools，ypbind和ypserv三个软件即可。

2. 安装`yp*`软件

```shell
# 查询安装包
[root@c2 ~]# yum list yp*
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
yp-tools.x86_64       2.14-5.el7            base
ypbind.x86_64         3:1.37.1-9.el7        base
ypserv.x86_64         2.31-12.el7           base

# 安装相关软件
[root@c2 ~]# yum install yp*
......
Dependencies Resolved

================================================
 Package      Arch   Version         Repository
                                           Size
================================================
Installing:
 yp-tools     x86_64 2.14-5.el7      base  79 k
 ypbind       x86_64 3:1.37.1-9.el7  base  62 k
 ypserv       x86_64 2.31-12.el7     base 156 k
Installing for dependencies:
 tokyocabinet x86_64 1.4.48-3.el7    base 459 k

Transaction Summary
================================================
Install  3 Packages (+1 Dependent package)

Total download size: 757 k
Installed size: 1.9 M
Is this ok [y/d/N]: y
```

#### 设定主机名和主机名解析

```shell
# 通过命令或文件设置主机名称
[root@c2 ~]# hostnamectl set-hostname c2
[root@c2 ~]# vi /etc/hostname
c2

# 设置主机名与IP的对应
[root@c2 ~]# vi /etc/hosts
192.168.1.101 c1
192.168.1.102 c2
192.168.1.103 c3
192.168.1.103 c4

# 查看主机名称
[root@c2 ~]# hostname
c2

# 查看主机相信息
[root@c2 ~]# hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: e9b05280560a4af4a6af98b31c11abda
           Boot ID: 0130f463c0fa4d429d117169a2e895e3
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1127.el7.x86_64
      Architecture: x86-64
```

#### 设置防火墙

```shell
# 设置firewall
[root@c2 ~]# firewall-cmd --permanent --add-service=rpc-bind
[root@c2 ~]# firewall-cmd --permanent --add-port=1011/tcp
[root@c2 ~]# firewall-cmd --permanent --add-port=1011/udp
[root@c2 ~]# firewall-cmd --permanent --add-port=1012/udp
[root@c2 ~]# firewall-cmd --reload

# 设置iptables
[root@c2 ~]# iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 1011 -j ACCEPT
[root@c2 ~]# iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

# 内网直接关闭防火墙
[root@c2 ~]# systemctl stop firewalld.service
```

#### 设定NIS的域名

```shell
# 设置NIS的域名(写入文件不会立即生效)
[root@c2 ~]# echo 'NISDOMAIN=centos' >> /etc/sysconfig/network

# 设置NIS服务器启动的端口号
[root@c2 ~]# echo 'YPSERV_ARGS=-p 1011' >> /etc/sysconfig/network

# 查看网络设置
[root@c2 ~]# cat /etc/sysconfig/network
NISDOMAIN=centos
YPSERV_ARGS="-p 1011"

# 查看NIS的域名
[root@c2 ~]# nisdomainname 
centos
```

#### 配置访问NIS服务器的权限

```shell
[root@c2 ~]# vi /etc/ypserv.conf 
# NIS 服务器使用内部局域网络，只要有/etc/hosts即可，不用 DNS 啦
dns no

# 预设会有 30 个数据库被读入内存当中，其实我们的账号档案并不多，30 个够用了。
files: 30

# 与 master/slave 有关，将同步更新的数据库比对所使用的端口，放置于 <1024 内。
xfr_check_port: yes

# 底下则是设定限制客户端或 slave server 查询的权限，利用冒号隔成四部分：
# [主机名/IP] : [NIS域名] : [可用数据库名称] : [安全限制]
# [主机名/IP]   ：可以使用 network/netmask 如 192.168.100.0/255.255.255.0 
# [NIS域名]   ：例如本案例中的 vbirdnis
# [可用数据库名称]：就是由 NIS 制作出来的数据库名称；
# [安全限制]      ：包括没有限制 (none)、仅能使用 <1024 (port) 及拒绝 (deny)
# 一般来说，你可以依照我们的网域来设定成为底下的模样：

#127.0.0.0/255.255.255.0 :*:*:none
#192.168.1.101/255.255.255.0 :*:*:none
#* :*: * :deny
# 星号 (*) 代表任何数据都接受的意思。上面三行的意思是，开放 lo 内部接口、
# 开放内部 LAN 网域，且杜绝所有其他来源的 NIS 要求的意思。

* : *: *: deny
```

#### 设置主从模式的配置

```shell
# 查看主机名称
[root@c2 ~]# vi /var/yp/Makefile
NOPUSH=true
```

#### 启动与观察所有相关的服务

```shell
# 修改yppasswdd服务的端口，该功能与 NIS 客户端有关，允许用户在客户端修改服务器上的密码
[root@c2 ~]# vi /etc/sysconfig/yppasswdd 
YPPASSWDD_ARGS='--port 1012'

# 启动yppasswdd服务并设置开机启动
[root@c2 ~]# systemctl start yppasswdd.service && systemctl enable yppasswdd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/yppasswdd.service to /usr/lib/systemd/system/yppasswdd.service.

# 启动rpcbind服务并设置开机启动
[root@c2 ~]# systemctl start rpcbind.service && systemctl enable rpcbind.service

# 启动ypserv服务并设置开机启动
[root@c2 ~]# systemctl start ypserv.service && systemctl enable ypserv.service
Created symlink from /etc/systemd/system/multi-user.target.wants/ypserv.service to /usr/lib/systemd/system/ypserv.service.

# 查看ypserv服务状态
[root@c2 ~]# systemctl status ypserv.service 
  ypserv.service - NIS/YP (Network Information Service) Server
   Loaded: loaded (/usr/lib/systemd/system/ypserv.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-12-22 14:31:51 CST; 21h ago
 Main PID: 1251 (ypserv)
   Status: "Processing requests..."
    Tasks: 1
   CGroup: /system.slice/ypserv.service
           └─1251 /usr/sbin/ypserv -f -p 1011

Dec 22 14:31:50 c2 systemd[1]: Starting NIS/YP (Network Information Servic.....
Dec 22 14:31:51 c2 ypserv[1251]: WARNING: no securenets file found!
Dec 22 14:31:51 c2 systemd[1]: Started NIS/YP (Network Information Service...r.
Hint: Some lines were ellipsized, use -l to show in full.

# 查看yppasswdd服务状态
[root@c2 ~]# systemctl status yppasswdd.service 
 yppasswdd.service - NIS/YP (Network Information Service) Users Passwords Change Server
   Loaded: loaded (/usr/lib/systemd/system/yppasswdd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-12-22 14:31:51 CST; 21h ago
  Process: 1244 ExecStartPre=/usr/libexec/yppasswdd-pre-setdomain (code=exited, status=0/SUCCESS)
 Main PID: 1483 (rpc.yppasswdd)
   Status: "Processing requests..."
    Tasks: 1
   CGroup: /system.slice/yppasswdd.service
           └─1483 /usr/sbin/rpc.yppasswdd -f --port 1012

Dec 22 14:31:50 c2 systemd[1]: Starting NIS/YP (Network Information Servic.....
Dec 22 14:31:51 c2 systemd[1]: Started NIS/YP (Network Information Service...r.
Dec 22 15:29:02 c2 rpc.yppasswdd[1483]: update nisuser1 (uid=1001) from hos....
Hint: Some lines were ellipsized, use -l to show in full.

# 查看rpcbind服务状态
[root@c2 ~]# systemctl status rpcbind.service 
  rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-12-22 14:31:40 CST; 21h ago
  Process: 814 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 830 (rpcbind)
    Tasks: 1
   CGroup: /system.slice/rpcbind.service
           └─830 /sbin/rpcbind -w

Dec 22 14:31:39 c2 systemd[1]: Starting RPC bind service...
Dec 22 14:31:40 c2 systemd[1]: Started RPC bind service.

# 查看所有注册rpc的服务程序及端口号
[root@c2 ~]# rpcinfo -p localhost
   program vers proto   port  service
...
    100009    1   udp   1012  yppasswdd
    100004    2   udp   1011  ypserv
    100004    1   udp   1011  ypserv
    100004    2   tcp   1011  ypserv
    100004    1   tcp   1011  ypserv
...

# 查看注册rpc中的ypserv状态
[root@c2 ~]# rpcinfo -u localhost ypserv
program 100004 version 1 ready and waiting
program 100004 version 2 ready and waiting
```

#### 建立数据库

```shell
# 转化帐号为数据库
[root@c2 ~]# /usr/lib64/yp/ypinit -s c1

# 查看注册rpc中的ypserv状态
[root@c2 ~]# rpcinfo -u localhost ypserv
program 100004 version 1 ready and waiting
program 100004 version 2 ready and waiting
```

### NIS Client客户端的设置

#### 软件安装

1. 查询相关软件

```shell
# 查询yp开头的软件
[root@c3 user]# rpm -qa | grep '^yp'

# 查询rpc开头的软件
[root@c3 user]# rpm -qa | grep '^rpc'
rpcbind-0.2.0-49.el7.x86_64
```

根据查询结果，系统已经安装rpcbind软件，只需要安装yp-tools，ypbind和ypserv三个软件即可。

2. 安装`yp*`软件

```shell
# 查询安装包
[root@c3 user]# yum list yp*
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
yp-tools.x86_64       2.14-5.el7            base
ypbind.x86_64         3:1.37.1-9.el7        base
ypserv.x86_64         2.31-12.el7           base

# 安装相关软件y
[root@c3 user]# yum install -y yp-tools ypbind
......
Dependencies Resolved

================================================
 Package      Arch   Version         Repository
                                           Size
================================================
Installing:
 yp-tools    x86_64    2.14-5.el7          /yp-tools-2.14-5.el7.x86_64    201 k
 ypbind      x86_64    3:1.37.1-9.el7      /ypbind-1.37.1-9.el7.x86_64     98 k

Transaction Summary
================================================
Install  2 Packages

Total size: 299 k
Installed size: 299 k
Is this ok [y/d/N]: y
...
Installed:
  yp-tools.x86_64 0:2.14-5.el7           ypbind.x86_64 3:1.37.1-9.el7

Complete!
```

#### 设置防火墙

```shell
# 设置firewall
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-port=1011/tcp
firewall-cmd --permanent --add-port=1011/udp
firewall-cmd --permanent --add-port=1012/udp
firewall-cmd --reload

# 设置iptables
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 --dport 1011 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 1011,1012 -j ACCEPT

# 内网直接关闭防火墙
[root@c3 ~]# systemctl stop firewalld.service
```

#### 服务设置

- 通过setup管理工具设置
    - 输入 setup命令
    - 选择『Authentication configuration』的项目
    - 选择『Use NIS』项目
    - 填写NIS域名-(Domain: `centos`) 以及 NIS服务器的IP-(Server: `192.168.1.101`)，按下确认即可
    - 跳回原来界面即成功

```shell
[root@c3 ~]# setup
# Authentication configuration
# Use NIS
# Domain: centos
# Server: 192.168.1.101
```

#### 检查服务设置

```shell
[root@c3 root]# cat /etc/sysconfig/network
NISDOMAIN=centos

[root@c3 root]# cat /etc/yp.conf 
...
domain centos server 192.168.1.101

[root@c3 root]# vi /etc/nsswitch.conf
passwd:     files nis sss
shadow:     files nis sss
group:      files nis sss
#initgroups: files sss

#hosts:     db files nisplus nis dns
hosts:      files nis dns myhostname

[root@c3 root]# cat /etc/sysconfig/authconfig 
...
USENIS=yes

[root@c3 root]# cat /etc/pam.d/system-auth
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        sufficient    pam_fprintd.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nis nullok try_first_pass use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
```

#### NIS client 端的检验

```shell
# 查看所有注册rpc的服务程序及端口号
[root@c3 root]# rpcinfo -p localhost
   program vers proto   port  service
    100007    2   udp    725  ypbind
    100007    1   udp    725  ypbind
    100007    2   tcp    728  ypbind
    100007    1   tcp    728  ypbind

# 查看NIS client的账号
[root@c3 root]# id nisuser1
uid=1001(nisuser1) gid=1001(nisuser1) groups=1001(nisuser1)
[root@c3 root]# id nisuser2
uid=1002(nisuser2) gid=1002(nisuser2) groups=1002(nisuser2)
[root@c3 root]# id nisuser3
uid=1003(nisuser3) gid=1003(nisuser3) groups=1003(nisuser3)

# 检验数据库
[root@c3 root]# yptest 
Test 1: domainname
Configured domainname is "centos"

Test 2: ypbind
Used NIS server: c1

Test 3: yp_match
WARNING: No such key in map (Map passwd.byname, key nobody)
...
Test 6: yp_master
c1
...
Test 9: yp_all
nisuser1 nisuser1:$6$RLbmKoa/$yVpIiAMMBnn2dxnn0/l12W1j6LoXPIDXvPYk7HE6IBWOJOVAY2OMY835E63Z6mfz2S.y5KRZ.VTIC6aH3PbfN/:1001:1001::/public/home/nisuser1:/bin/shell
nisuser2 nisuser2:$6$2HtFaftP$DNTVkSyDBN5kLjVo2PYuuP45dVJhJkYRjAbbmrFhsW0jVKXr4K7YSNraWHf7fWKU6X2PdWQJ1yd0SzbmUCpDJ1:1002:1002::/public/home/nisuser2:/bin/shell
nisuser3 nisuser3:$6$sLc3hNJ6$rEwtDhC6ng7.n1mThg2s8U/9FAVEQxYslCBweJ6RF4pwYiy9Z2QN.yw6DQcLV2lowtgAM7f8/XoVpb0z2Hnza/:1003:1003::/public/home/nisuser3:/bin/shell
1 tests failed

# 检验数据库数量
[root@c3 root]# ypwhich -x
Use "ethers"	for map "ethers.byname"
Use "aliases"	for map "mail.aliases"
Use "services"	for map "services.byname"
Use "protocols"	for map "protocols.bynumber"
Use "hosts"	for map "hosts.byname"
Use "networks"	for map "networks.byaddr"
Use "group"	for map "group.byname"
Use "passwd"	for map "passwd.byname"

# 读取数据库内容
[root@c3 root]# ypcat passwd.byname
nisuser1:$6$RLbmKoa/$yVpIiAMMBnn2dxnn0/l12W1j6LoXPIDXvPYk7HE6IBWOJOVAY2OMY835E63Z6mfz2S.y5KRZ.VTIC6aH3PbfN/:1001:1001::/public/home/nisuser1:/bin/shell
nisuser2:$6$2HtFaftP$DNTVkSyDBN5kLjVo2PYuuP45dVJhJkYRjAbbmrFhsW0jVKXr4K7YSNraWHf7fWKU6X2PdWQJ1yd0SzbmUCpDJ1:1002:1002::/public/home/nisuser2:/bin/shell
nisuser3:$6$sLc3hNJ6$rEwtDhC6ng7.n1mThg2s8U/9FAVEQxYslCBweJ6RF4pwYiy9Z2QN.yw6DQcLV2lowtgAM7f8/XoVpb0z2Hnza/:1003:1003::/public/home/nisuser3:/bin/shell
```

#### NIS client 端的使用者参数修改

```shell
# 查看nisuser账号
[root@c3 ~]# grep nisuser /etc/passwd

# 切换nisuser1账号
[root@c3 ~]# su - nisuser1

# 在c3客户端修改nisuser1账号的密码
[nisuser1@c3 ~]$ yppasswd
Changing NIS account information for nisuser1 on c3.
Please enter old password:
Changing NIS password for nisuser1 on c3.
Please enter new password:
Please retype new password:

The NIS password has been changed on c3.

# 查看centos域名的数据库是否更新
[root@c3 centos]# ll /var/yp/centos/
total 3152
...
-rw-------. 1 root root 137216 Dec 22 14:52 netid.byname
-rw-------. 1 root root 136704 Dec 22 15:29 passwd.byname
-rw-------. 1 root root 136704 Dec 22 15:29 passwd.byuid
-rw-------. 1 root root 142336 Dec 22 14:52 protocols.byname

# 查看日志，密码已经被修改
[root@c3 centos]# tail /var/log/messages
...
Dec 22 15:29:02 c3 rpc.yppasswdd[1483]: update nisuser1 (uid=1001) from host 192.168.1.102 successful.
```

## NIS 常见错误与解决方法

### yppasswd:yppasswdd not running on NIS master host

在服务器上运行该命令正常。

这是因为客户端不能解析到NIS服务器的IP地址，只要你在客户端的 `/etc/hosts`文件里面添加IP对应关系即可。

### yppasswd:yppasswdd not running on NIS master host ("hostname").

这时候你ping下hostname看解析的为多少，如果为`127.0.0.1`的话，那就赶紧在`/etc/hosts`中加入IP hostname，如果解析正常，自然不会报错了。

## Additional Resources

### Documentation

1. [Linux Domain Identity, Authentication, and Policy Guide](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/index)
2. [Linux 域身份、身份验证和策略指南](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/index)
3. [28.4. 网络信息服务 (NIS/YP)](https://freebsdchina.github.io/freebsdhandbook/network-nis.html)
4. [第十四章、账号控管： NIS 服务器](http://cn.linux.vbird.org/linux_server/0430nis_1.php)：鸟哥的Linux私房菜-服务器篇介绍关于账号管控的部分，主要是讲解怎么搭建NIS服务器。
5. [第十四章、Linux 账号管理与 ACL 权限配置](http://cn.linux.vbird.org/linux_basic/0410accountmanager.php#account)：鸟哥的Linux私房菜-基础篇介绍管理账号的权限功能。

### Useful Websites

1. [Slurm安装2之NIS服务](https://www.michaelapp.com/posts/2018/2018-09-12-CentOS7.6-slurm%E5%AE%89%E8%A3%852/)：介绍NIS服务器怎么安装
2. [服务器集群（四）——NIS用户同步](https://www.cnblogs.com/linagcheng/p/16195927.html)：介绍NIS服务器怎么安装。

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
3. [鸟哥的 Linux 私房菜 -- 服务器架设篇 第三版](http://cn.linux.vbird.org/linux_server/)
