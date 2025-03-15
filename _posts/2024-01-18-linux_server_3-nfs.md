---
layout: post
title:  使用NFS服务共享文件
categories: [linux]
comments: true
tags: [Linux]
---

> 本文主要介绍Linux系统服务器的NFS服务。

* TOC
{:toc}

<!--more-->

## 引言

### 什么是 NFS？

网络文件系统（NFS）是一种分布式文件系统，允许不同的远程系统访问一个文件共享。我们都了解文件应该存储在中央服务器上，以便于安全管理和备份。NFS 为我们提供了一个易于管理和控制客户端资源访问的文件共享服务。

### NFS 的历史与发展

网络文件系统（NFS, Network File System）自1984年由Sun Microsystems开发以来，已经存在了很长时间。NFS 第三版于1995年发布，至今仍广泛使用，尽管其间进行了许多改进。Sun Microsystems 将 NFS 交给了[IETF](https://ietf.org/)，后者在2000年发布了第四版，并且自发布以来也经历了多次改进。

NFS 的初衷是为了让用户能够从一台计算机访问另一台计算机上的文件。虽然在互联网时代这个概念听起来有些过时，但也许你并不需要互联网，只需要一个可以简单方便地共享文件的中央存储库。

与 CIFS/SMB（Samba）不同，NFS 可以使用类 Unix 的访问控制进行管理。与 FTP 不同，NFS 可以本地挂载并访问，而无需每次需要时重新连接。此外，NFS 第四版支持 `LDAP` 和 `Kerberos`，可以实现集中且安全的身份验证。

### NFS 的应用场景

#### 数据共享

在企业或数据中心环境中，多台服务器需要访问同一组文件时，例如多个Web服务器共享用户上传的图片、文档等资源。
开发团队间共享代码库或者构建工具链，实现跨开发环境的一致性。

#### 虚拟化与云计算

虚拟机（VMs）通过宿主机挂载的NFS共享来存储和读取其操作系统镜像、应用程序数据及用户数据，简化管理和维护。
在云服务中，NFS可以作为云主机与云存储之间的接口，用于存储和共享虚拟机磁盘映像或容器数据卷。

#### 集群计算

HPC（高性能计算）集群中的各个节点可以通过NFS共享大型数据集，以支持并行计算任务的数据访问需求。
大数据处理环境中，分布式计算框架可能依赖于NFS共享目录，使得不同节点能够存取统一的数据源。

#### 备份与恢复

作为备份目标，NFS服务器可被用作集中式备份存储，便于进行数据备份和灾难恢复操作。
数据迁移过程中，NFS可以临时作为中间层存储，简化从旧系统到新系统的数据迁移过程。

#### 媒体制作与渲染农场

在多媒体制作和动画渲染等行业中，大量工作站需要共享大型项目文件，NFS提供了高效且实时的数据访问能力。

#### 容器编排

在Kubernetes或其他容器编排平台中，NFS常用于持久卷（Persistent Volumes）提供动态存储分配，满足容器应用对持久化数据的需求。
测试与部署：

开发人员在本地机器上快速搭建与生产环境一致的配置时，通过挂载NFS共享可以确保代码版本、配置文件以及静态资源的同步更新。

## NFS 的工作原理
   
### NFS 架构概述

NFS（Network File System）工作原理基于网络的客户端-服务器架构，它允许网络中的不同计算机如同访问本地磁盘一样透明地共享和存取远程主机上的文件系统

具体过程如下：

首先，NFS服务器在其操作系统上配置并启动相关的服务进程，如`rpc.nfsd`用于处理来自客户端的数据请求，并通过`rpc.mountd`管理共享目录的挂载权限。服务器在 `/etc/exports` 文件中定义了哪些目录可以被哪些客户端以何种权限访问。

当客户端需要访问服务器端的共享资源时，它会通过Portmapper（或rpcbind）服务查询到NFS服务器所监听的实际端口，并向这些端口发送挂载请求。一旦服务器验证并批准该请求后，客户端就可以将远端的共享目录“挂载”至本机的一个指定路径下。

在数据交换过程中，NFS协议借助于Remote Procedure Call (RPC)机制，使客户端能够执行诸如读、写、打开、关闭等与文件系统相关的操作。这些操作实际上是对服务器上对应文件系统的调用，而结果则通过网络返回给客户端。

为了提高性能，NFS支持缓存技术，即客户端可以对常用文件的部分或全部内容进行缓存。同时，为了确保多用户环境下的数据一致性，NFS使用lock manager服务（如lockd）来管理和同步多个客户端对同一文件的并发访问。

### RPC (Remote Procedure Call) 介绍

远程过程调用（RPC, Remote Procedure Call）是一种软件通信协议，一个程序使用该协议请求另一个位于不同计算机和网络上的程序提供服务，而无需了解网络的具体细节。具体来说，RPC 用于像调用本地系统进程一样调用远程系统上的其他进程。过程调用有时也称为函数调用或子程序调用。

RPC 作为一种低级别的传输协议，用于在通信程序之间传输数据包，它采用客户端-服务器模型。请求程序被称为客户端，而提供服务的程序被称为服务器。如同本地过程调用一样，RPC 是一个同步操作，要求请求程序在远程过程的结果返回之前暂停执行。然而，通过使用共享同一地址空间的轻量级进程或线程，可以并发执行多个 RPC。

所有 NFS 版本都依赖于客户端和服务器之间的 远程过程调用 (RPC)。Red Hat Enterprise Linux 7 中的 RPC 服务由 rpcbind 服务控制。

以下 RPC 进程有助于 NFS 服务：

- `rpcbind`

rpcbind 接受本地 RPC 服务的端口保留。这些端口随后可用（或发布），以便相应的远程 RPC 服务可以访问它们。rpcbind 响应对 RPC 服务的请求，并设置到请求的 RPC 服务的连接。这不能与 NFSv4 一起使用。

- `rpc.mountd`

NFS 服务器使用这个进程来处理来自 NFSv3 客户端的 MOUNT 请求。它检查所请求的 NFS 共享是否目前由 NFS 服务器导出，并且允许客户端访问它。如果允许挂载请求，rpc.mountd 服务器会以 Success 状态回复，并将这个 NFS 共享的文件处理返回给 NFS 客户端。

- `rpc.nfsd`

rpc.nfsd 允许定义服务器公告的显式 NFS 版本和协议。它与 Linux 内核配合使用，以满足 NFS 客户端的动态需求，例如在每次 NFS 客户端连接时提供服务器线程。这个进程对应于 nfs 服务。

- `lockd`

lockd 是一个在客户端和服务器上运行的内核线程。它实现了 网络锁定管理器 (NLM)协议，它允许 NFSv3 客户端锁定服务器上的文件。每当运行 NFS 服务器以及挂载 NFS 文件系统时，它会自动启动。

- `rpc.statd`

这个进程实现 网络状态监控器 (NSM)RPC 协议，该协议在 NFS 服务器没有正常关闭而重新启动时，通知 NFS 客户端。RPC.statd 由 nfslock 服务自动启动，不需要用户配置。这不能与 NFSv4 一起使用。

- `rpc.rquotad`

这个过程为远程用户提供用户配额信息。RPC.rquotad 由 nfs 服务自动启动，不需要用户配置。

- `rpc.idmapd`

rpc.idmapd 提供 NFSv4 客户端和服务器上调用，这些调用在线 NFSv4 名称（以 user@domain形式的字符串）和本地 UID 和 GID 之间进行映射。要使 idmapd 与 NFSv4 正常工作，必须配置 /etc/idmapd.conf 文件。至少应指定"Domain"参数，该参数定义 NFSv4 映射域。如果 NFSv4 映射域与 DNS 域名相同，可以跳过这个参数。客户端和服务器必须同意 NFSv4 映射域才能使 ID 映射正常工作。

### NFS 版本对比

| NFS版本 | 发布时间 | 版本描述 |
| -- | -- | -- |
| NFSv1 | 第一个版本，发布于1984年 | 基于Sun Microsystems的原始设计，存在诸守限制且功能有限，它并未广泛部署 |
| NFSv2 | 发布于1985年，在RFC 1094中定义 | 引入了文件属性、目录操作和其他增强功能，但仍有一些局限性，如32位文件大小限制和缺乏安全模型 |
| NFSv3 | 发布于1995年，在RFC 1813中定义 | 支持异步写入，在错误处理方面比 NFSv2 更可靠。支持 64 位文件大小和偏移量，允许客户端访问大于 2GB 的文件共享。 |
| NFSv4 | NFSv4.0在2000年由RFC 3010定义，NFSv4.1由RFC 5661在2010年定义 | 可通过防火墙和互联网工作，不需要 rpcbind 服务。NFSv4引入了状态保持、会庄概念、RPC流水线化、复制支持、安全性改进〈例如 Kerberos 集成) 以及其他许多新特性。NFSY4.1还增加了并发写入支持、租约管理和复制提升等 |

所有版本的 NFS 都支持通过 IP 网络的传输控制协议（TCP）。只有 NFSv4 需要 TCP。NFSv2 和 NFSv3 也可以选择使用用户数据报协议（UDP）通过 IP 网络建立客户端和服务器之间的无状态网络连接。

当使用 NFSv2 或 NFSv3 与 UDP 时，UDP 协议所需的开销通常低于 TCP。这在高正常运行时间和低网络拥塞的情况下可以提升性能。然而，UDP 是无状态的，如果服务器意外崩溃，UDP 客户端会继续向服务器发送请求，导致网络拥塞。

此外，如果客户端丢失了一个数据帧，使用 UDP 时需要重新发送整个 RPC 请求，而使用 TCP 时只需要重传丢失的数据帧。因此，建议在连接到 NFS 服务器时使用 TCP。在大多数实际应用中，推荐使用 TCP。

## NFS 的优势与劣势

### NFS 的优点

NFS（Network File System）的主要优点包括：

跨平台共享：不同操作系统之间能够无缝共享文件和目录。
集中管理：数据存储集中化，便于管理和备份。
空间节约与成本优化：客户端无需重复存储相同数据，降低硬件投资成本。
灵活访问：用户可以从任何授权的系统上透明地读写远程文件。
性能优化：支持缓存机制以提高文件访问速度，减少网络传输开销。
可扩展性好：可以根据需求增加服务器来扩展存储资源。
易用性强：配置简单，使用标准命令即可挂载和操作远程文件系统。

### NFS 的缺点

- 基于本质上不安全的 RPC。RPC 通信应仅限于防火墙后的可信网络。
- NFSv4 和 NFSv4.1 在最大带宽上可能有限制，在网络流量大时可能会变慢。这一情况在版本 4.2 中有所改进。

根据下面的标题补充相关内容

## NFS 服务器的搭建

### 环境准备

在开始搭建 NFS 服务器之前，确保满足以下条件：
- **操作系统**: 确保服务器的操作系统支持 NFS。常见的支持 NFS 的操作系统包括 Linux（如 Ubuntu、CentOS）、FreeBSD 等。
- **硬件**: 硬件资源应该足够运行 NFS 服务器，包括足够的存储空间以及适当的 CPU 和内存。
- **网络**: 服务器和客户端之间应该有一个稳定的网络连接。确认服务器和客户端能够互相 ping 通，并且网络防火墙规则允许 NFS 通信。
- **用户权限**: 确保你有足够的权限安装和配置 NFS 服务器软件。

### 安装 NFS 服务器软件

#### 软件需求

- rpchbind: RPC 主程序, NFS可以被视为一个 RPC 服务，而要启动任何一个 RPC 服务之前，我们都需要做好 port 的对应 (mapping) 的工作才行，这个工作其实就是『 rpcbind 』这个服务所负责的！也就是说， 在启动任何一个 RPC 服务之前，我们都需要启动 rpcbind 才行！
- nfs-utils: NFS主程序，提供 rpc.nfsd 及 rpc.mountd 这两个NFS daemons 与其他相关 documents 与说明文件、执行文件等的软件！

#### 软件安装流程

1. 查询相关软件

```shell
# 查询包含nfs的软件
[root@c1 ~]# rpm -qa | grep nfs 

# 查询rpc开头的软件
[root@c1 ~]# rpm -qa | grep '^rpc'
```

根据查询结果，系统没有安装rpchbind和nfs-utilsh，所以都需要安装。

2. 安装相关软件

```shell
# 查询安装包
[root@c1 ~]# yum list nfs-utils
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
nfs-utils.x86_64         1:1.3.0-0.54.el7        base

[root@c1 ~]# yum list rpchbind
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
rpcbind.x86_64         0.2.0-49.el7        base

# 安装相关软件
[root@c1 ~]# yum install -y nfs-utils rpchbind
......
Dependencies Resolved

================================================
 Package      Arch   Version         Repository
                                           Size
================================================
Installing:
 nfs-utils    x86_64    1:1.3.0-0.54.el7         / nfs-utils-1:1.3.0-0.54.el7.x86_64    1.1 M
 rpcbind      x86_64    0.2.0-49.el7     /rpcbind-0.2.0-49.el7.x86_64     101 k

Transaction Summary
================================================
Install  2 Packages
...
Installed:
  nfs-utils.x86_64 1:1.3.0-0.54.el7           rpcbind.x86_64 0.2.0-49.el7

Complete!
```

### 配置 NFS 服务器

在 NFS 服务器中配置导出的方法有两种：

- 手动编辑 NFS 配置文件，即 /etc/exports，以及
- 通过命令行，即使用 exportfs命令

#### /etc/exports 配置文件 

`/etc/exports` 文件控制哪些文件系统被导出到远程主机，并指定选项。

##### 语法规则

它遵循以下语法规则：

- 空白行将被忽略。
- 要添加注释，以井号(#)开始一行。
- 您可以使用反斜杠(\)换行长行。
- 每个导出的文件系统都应该独立。
- 所有在导出的文件系统后放置的授权主机列表都必须用空格分开。
- 每个主机的选项必须在主机标识符后直接放在括号中，没有空格分离主机和第一个括号。

##### 文件结构

导出的文件系统的每个条目都有以下结构：

`export host(options)`

上述结构使用以下变量：

- export: 导出的目录
- host: 导出要共享的主机或网络
- options: 用于 host 的选项。默认值为(ro,sync,wdelay,root_squash)，每个导出的文件系统的默认值都必须被显式覆盖。例如，如果没有指定 rw 选项，则导出的文件系统将以只读形式共享。

例如：`/public 192.168.1.102(rw,sync,no_root_squash)`

可以指定多个主机，以及每个主机的特定选项。要做到这一点，将它们列在与用空格分隔的列表相同的行中，每个主机名后跟其各自的选项（在括号中）

例如：`export host1(options1) host2(options2) host3(options3)`

```shell
/public    192.168.1.102(rw,sync,no_root_squash) 192.168.1.103(rw,sync,no_root_squash) 192.168.1.104(rw,sync,no_root_squash)
```

要指定 NFS 服务器应该分配给来自特定主机的远程用户的用户 ID 和组 ID，请分别使用 anonuid 和 anongid 选项，如下所示：

例如：`/public 192.168.1.102(anonuid=uid,anongid=gid)`

> - host参数
>>
>> - 单台机器: 完全限定域名（可由服务器解析）、主机名（可由服务器解析）或 IP 地址。
>> - 使用通配符指定的一系列机器: 使用 `*` 或 `?` 字符指定字符串匹配项。通配符不能用于 IP 地址；但是，如果反向 DNS 查找失败，则可能会意外起作用。当在完全限定域名中指定通配符时，点`(.)`不会包含在通配符中。例如： `*.example.com` 包含 `one.example.com`，但不包括 `include one.two.example.com`。
>> - IP 网络: 使用 `a.b.c.d/z`，其中 `a.b.c.d` 是网络，`z` 是子网掩码的位数（如 `192.168.0.0/24`）。另一种可接受的格式是 `a.b.c.d/netmask`，其中 `a.b.c.d` 是网络，`netmask` 是子网掩码（例如 `192.168.100.8/255.255.255.0`）。
>> - Netgroups: 使用格式 `@group-name` ，其中 `group-name` 是 `NIS netgroup` 名称。
>>
> - options参数
>>
>> - `ro`: 导出的文件系统是只读的。远程主机无法更改文件系统中共享的数据。要允许主机更改文件系统（即读写），请指定 rw 选项。
>> - `sync`: 在将之前的请求所做的更改写入磁盘前，NFS 服务器不会回复请求。要启用异步写入，请指定 async 选项。
>> - `wdelay`: 如果 NFS 服务器预期另外一个写入请求即将发生，则 NFS 服务器会延迟写入磁盘。这可以提高性能，因为它可减少不同的写命令访问磁盘的次数，从而减少写开销。要禁用此功能，请指定 no_wdelay。只有指定了默认的 sync 选项时，no_wdelay 才可用。
>> - `root_squash`: 这可防止远程连接的 root 用户（而不是本地）具有 root 权限；相反，NFS 服务器会为他们分配用户 ID nfsnobody。这可以有效地将远程 root 用户的权限"挤压"成最低的本地用户，从而防止在远程服务器上可能的未经授权的写操作。要禁用 root 压缩，请指定 no_root_squash。要对所有远程用户（包括root用户）进行压缩，请使用all_squash。

#### /etc/sysconfig/nfs 配置文件

- 要防止客户端使用 NFSv4，可以通过`/etc/sysconfig/nfs`添加以下设置来将其关闭。

`RPCNFSDARGS= -N 4`

- 将您的 NFS 服务器配置为只支持 NFS 版本 4.0 及更新的版本，可以通过在 `/etc/sysconfig/nfs` 配置文件中添加以下行来禁用 `NFSv2`、`NFSv3` 和 `UDP`：

`RPCNFSDARGS="-N 2 -N 3 -U"`

- 禁用对 RPCBIND、MOUNT 和 NSM 协议调用的监听，这在仅使用 NFSv4 的情况下不需要。禁用这些选项的影响有：尝试使用 NFSv2 或 NFSv3 从服务器挂载共享的客户端变得无响应。NFS 服务器本身无法挂载 NFSv2 和 NFSv3 文件系统。配置文件中添加以下行来禁用。

`RPCMOUNTDOPTS="-N 2 -N 3"`

以上修改都需要重启NFS服务器：`$ systemctl restart nfs`

#### exportfs 命令

使用 NFS 将每个文件系统导出到远程用户，以及这些文件系统的访问级别，均列在 `/etc/exports` 文件中。当 nfs 服务启动时，`/usr/sbin/exportfs` 命令启动并读取此文件，将控制传递给实际挂载进程的 `rpc.mountd` （如果 NFSv3），然后传给 `rpc.nfsd`，然后供远程用户使用。

手动发出时，`/usr/sbin/exportfs` 命令允许 root 用户有选择地导出或取消导出目录，而无需重启 NFS 服务。给定正确的选项后，`/usr/sbin/exportfs` 命令将导出的文件系统写入 `/var/lib/nfs/xtab`。由于 `rpc.mountd` 在决定对文件系统的访问权限时引用 `xtab` 文件，因此对导出的文件系统列表的更改会立即生效。

如果没有将选项传给 exportfs 命令，它将显示当前导出的文件系统列表。有关 `exportfs` 命令的详情，请参考`man exportfs`。

以下是 /usr/sbin/exportfs 常用的选项列表：

- `-r`: 通过在 /var/lib/nfs/etab 中构建新的导出列表，将 /etc/exports 中列出的所有目录导出。如果对 /etc/exports 做了任何更改，这个选项可以有效地刷新导出列表。
- `-a`: 根据将哪些其他选项传给 /usr/sbin/exportfs，导致所有目录被导出或取消导出。如果没有指定其他选项，/usr/sbin/exportfs 会导出 /etc/exports 中指定的所有文件系统。
`-o file-systems`: 指定没有在 /etc/exports 中列出的要导出的目录。将 file-systems 替换为要导出的其它文件系统。这些文件系统的格式化方式必须与 /etc/exports 中指定的方式相同。此选项通常用于在将导出的文件系统永久添加到导出的文件系统列表之前，对其进行测试。有关 /etc/exports 语法的详情，请参考 第 8.6.1 节 “/etc/exports 配置文件”。
- `-i`: 忽略 /etc/exports ；只有命令行上指定的选项才会用于定义导出的文件系统。
- `-u`: 不导出所有共享目录。命令 /usr/sbin/exportfs -ua 可暂停 NFS 文件共享，同时保持所有 NFS 守护进程正常运行。要重新启用 NFS 共享，请使用 exportfs -r。
- `-v`: 详细操作，当执行 exportfs 命令时，更详细地显示正在导出的或取消导出的文件系统。

### 启动与测试 NFS 服务

#### 先决条件

- rpcbind

对于支持 NFSv2 或 NFSv3 连接的服务器，rpcbind服务必须正在运行。要验证 rpcbind 是否处于活动状态，请使用以下命令：

```shell
# 启动rpcbind服务并设置开机启动
[root@c1 ~]`$ systemctl start rpcbind && systemctl enable rpcbind
```

要配置只使用 NFSv4 的服务器，它不需要 rpcbind。

- nfs-lock

在 Red Hat Enterprise Linux 7.0 中，如果您的 NFS 服务器导出 NFSv3 并在引导时启用，则需要手动启动并启用 nfs-lock 服务：

`$ systemctl start nfs-lock`
`$ systemctl enable nfs-lock`

在 Red Hat Enterprise Linux 7.1 及更高版本中，如果需要，nfs-lock 会自动启动，并尝试手动启用它。

- nfs

```shell
# 启动nfs服务并设置开机启动
[root@c1 ~]`$ systemctl start nfs && systemctl enable nfs
```

#### 流程

- 要启动 NFS 服务器，请使用以下命令：

`$ systemctl start nfs`

- 要使 NFS 在引导时启动，请使用以下命令：

`$ systemctl enable nfs`

- 要停止服务器，请使用：

`$ systemctl stop nfs`

- `restart` 选项是停止然后启动 NFS 的简写方式。这是编辑 NFS 配置文件后使配置更改生效的最有效方式。要重启服务器的类型：

`$ systemctl restart nfs`

- 编辑 `/etc/sysconfig/nfs` 文件后，运行以下命令来重启 nfs-config 服务，使新值生效：

`$ systemctl restart nfs-config`

- `try-restart` 命令仅在当前正在运行时启动 nfs。此命令等同于 Red Hat init 脚本中的 condrestart (有条件重启)非常有用，因为它不会在 NFS 未运行时启动守护进程。要有条件地重启服务器，请输入：

`$ systemctl try-restart nfs`

- 要在不重启服务类型的情况下重新载入 NFS 服务器配置文件：

`$ systemctl reload nfs`

#### 查询服务器分享资源

有两种方法可以发现 NFS 服务器导出了哪些文件系统。

在支持 NFSv3 的任何服务器上，使用 showmount 命令：

```shell
$ showmount -e myserver
Export list for mysever
/exports/foo
/exports/bar
```

在支持 NFSv4 的任何服务器上，挂载 根目录并查找。

```shell
# mount myserver:/ /mnt/
# cd /mnt/
exports
# ls exports
foo
bar
```

在同时支持 NFSv4 和 NFSv3 的服务器上，这两种方法都可以工作，并给出同样的结果。

## NFS 客户端的配置

### 安装 NFS 客户端

#### 软件需求

- rpchbind: RPC 主程序, NFS 其实可以被视为一个 RPC 服务，而要启动任何一个 RPC 服务之前，我们都需要做好 port 的对应 (mapping) 的工作才行，这个工作其实就是『 rpcbind 』这个服务所负责的！也就是说， 在启动任何一个 RPC 服务之前，我们都需要启动 rpcbind 才行！
- nfs-utils: NFS 主程序，提供 rpc.nfsd 及 rpc.mountd 这两个 NFS daemons 与其他相关 documents 与说明文件、执行文件等的软件！
- autofs: 这个服务在客户端计算机上面，会持续的侦测某个指定的目录， 并预先设定当使用到该目录下的某个次目录时，将会取得来自服务器端的 NFS 文件系统资源，并进行自动挂载的动作。

#### 软件安装流程

1. 查询相关软件

```shell
# 查询yp开头的软件
[root@c2 ~]# rpm -qa | grep nfs 

# 查询rpc开头的软件
[root@c2 ~]# rpm -qa | grep '^rpc'

# 查询autofs的软件
[root@c2 ~]# rpm -qa | grep autofs
autofs-5.0.1-0.rc2.143.el5
```

根据查询结果，系统没有安装`rpchbind`和`nfs-utils`，所以都需要安装。

2. 安装相关软件

```shell
# 查询安装包
[root@c2 ~]# yum list nfs-utils
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
nfs-utils.x86_64         1:1.3.0-0.54.el7        base

[root@c2 ~]# yum list rpchbind
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: ftp.sjtu.edu.cn
 * extras: ftp.sjtu.edu.cn
 * updates: mirror.lzu.edu.cn
Available Packages
rpcbind.x86_64         0.2.0-49.el7        base

# 安装相关软件
[root@c2 ~]# yum install -y nfs-utils rpchbind
......
Dependencies Resolved

================================================
 Package      Arch   Version         Repository
                                           Size
================================================
Installing:
 nfs-utils    x86_64    1:1.3.0-0.54.el7         / nfs-utils-1:1.3.0-0.54.el7.x86_64    1.1 M
 rpcbind      x86_64    0.2.0-49.el7     /rpcbind-0.2.0-49.el7.x86_64     101 k

Transaction Summary
================================================
Install  2 Packages
...
Installed:
  nfs-utils.x86_64 1:1.3.0-0.54.el7           rpcbind.x86_64 0.2.0-49.el7

Complete!
```

### 挂载远程文件系统

#### 先决条件

- rpcbind

对于支持 NFSv2 或 NFSv3 连接的服务器，rpcbind服务必须正在运行。要验证 rpcbind 是否处于活动状态，请使用以下命令：

```shell
# 启动rpcbind服务并设置开机启动
[root@c1 ~]`$ systemctl start rpcbind && systemctl enable rpcbind
```

要配置只使用 NFSv4 的服务器，它不需要 rpcbind。

- nfs

```shell
# 启动nfs服务并设置开机启动
[root@c1 ~]`$ systemctl start nfs && systemctl enable nfs
```

#### 挂载NFS

mount 命令在客户端上挂载 NFS 共享。其格式如下：

`mount -t nfs -o options server:/remote/export /local/directory`

这个命令使用以下变量：

- options: 以逗号分隔的挂载选项列表
- server: 导出您要挂载的文件系统的服务器的主机名、IP 地址或完全限定域名
- /remote/export: 从 server 导出的文件系统或目录，即您要挂载的目录
- /local/directory: 挂载 /remote/export 的客户端位置

例如

`mount -t nfs -o nfsvers=4 192.168.1.101:/public /public`

> - options参数
>> - `lookupcache=mode`
>> 指定内核应该如何管理给定挂载点的目录条目缓存。模式 的有效参数为 all、none 或 pos/positive。
>> - `nfsvers=version`
>> 指定要使用的 NFS 协议版本，其中 version 为 3 或 4。这对于运行多个 NFS 服务器的主机很有用。如果没有指定版本，NFS 将使用内核和 mount 命令支持的最高版本。
>> 选项 vers 等同于 nfsvers ，出于兼容性的原因包含在此发行版本中。
>> - `noacl`
>> 关闭所有 ACL 处理。当与旧版本的 Red Hat Enterprise Linux、Red Hat Linux 或 Solaris 交互时，可能会需要此功能，因为最新的 ACL 技术与较旧的系统不兼容。
>> - `nolock`
>> 禁用文件锁定。当连接到非常旧的 NFS 服务器时，有时需要这个设置。
>> - `noexec`
>> 防止在挂载的文件系统中执行二进制文件。这在系统挂载不兼容二进制文件的非 Linux 文件系统时有用。
>> - `nosuid`
>> 禁用 set-user-identifier 或 set-group-identifier 位。这可防止远程用户通过运行 setuid 程序获得更高的特权。
>> - `port=num`
>> 指定 NFS 服务器端口的数字值。如果 num 是 0 ( 默认值)，则 mount 会查询远程主机的 rpcbind 服务，以获取要使用的端口号。如果远程主机的 NFS 守护进程没有注册到其 rpcbind 服务，则会使用标准 NFS 端口号 TCP 2049。
>> - `rsize=num` 和 `wsize=num`
>> 这些选项设定在单个 NFS 读取或写入操作中传输的最大字节数。
>> rsize 和 wsize 没有固定的默认值。默认情况下，NFS 使用服务器和客户端都支持的最大的可能值。在 Red Hat Enterprise Linux 7 中，客户端和服务器的最大值为 1,048,576 字节。
>> - `sec=flavors`
>> 用于访问挂载导出上文件的安全类别。flavors 值是以冒号分隔的一个或多个安全类型列表。
>> 默认情况下，客户端会尝试查找客户端和服务器都支持的安全类别。如果服务器不支持任何选定的类别，挂载操作将失败。
>> sec=sys 使用本地 UNIX UID 和 GID。它们使用 AUTH_SYS 验证 NFS 操作。
>> sec=krb5 使用 Kerberos V5 ,而不是本地 UNIX UID 和 GID 来验证用户。
>> sec=krb5i 使用 Kerberos V5 进行用户身份验证，并使用安全校验和执行 NFS 操作的完整性检查，以防止数据被篡改。
>> sec=krb5p 使用 Kerberos V5 进行用户身份验证、完整性检查，并加密 NFS 流量以防止流量嗅探。这是最安全的设置，但它也会涉及最大的性能开销。
>> - `tcp`
>> 指示 NFS 挂载使用 TCP 协议。
>> - `udp`
>> 指示 NFS 挂载使用 UDP 协议。

| 参数 | 参数代表意义 | 系统默认值 |
| --- | --- | --- |
| suid/nosuid | 如果挂载的 partition 上面有任何 SUID 的 binary 程序时， 你只要使用 nosuid 就能够取消 SUID 的功能了！ | suid |
| rw/ro | 你可以指定该文件系统是只读 (ro) 或可擦写喔！服务器可以提供给你可擦写， 但是客户端可以仅允许只读的参数设定值！ | rw |
| dev/nodev | 是否可以保留装置档案的特殊功能？一般来说只有 /dev 这个目录才会有特殊的装置，因此你可以选择 nodev 喔！ | dev |
| exec/noexec | 是否具有执行 binary file 的权限？ 如果你想要挂载的仅是数据区 (例如 /home)，那么可以选择 noexec 啊！ | exec |
| user/nouser | 是否允许使用者进行档案的挂载与卸除功能？ 如果要保护文件系统，最好不要提供使用者进行挂载与卸除吧！ | nouser |
| auto/noauto | 这个 auto 指的是『mount -a』时，会不会被挂载的项目。 如果你不需要这个 partition 随时被挂载，可以设定为 noauto。 | auto |
| fg /bg | 当执行挂载时，该挂载的行为会在前景 (fg) 还是在背景 (bg) 执行？ 若在前景执行时，则 mount 会持续尝试挂载，直到成功或 time out 为止，若为背景执行， 则 mount 会在背景持续多次进行 mount ，而不会影响到前景的程序操作。 如果你的网络联机有点不稳定，或是服务器常常需要开关机，那建议使用 bg 比较妥当。 | fg |
| soft /hard | 如果是 hard 的情况，则当两者之间有任何一部主机脱机，则 RPC 会持续的呼叫，直到对方恢复联机为止。如果是 soft 的话，那 RPC 会在 time out 后『重复』呼叫，而非『持续』呼叫， 因此系统的延迟会比较不这么明显。同上，如果你的服务器可能开开关关，建议用 soft 喔！ | hard |
| intr | 当你使用上头提到的 hard 方式挂载时，若加上 intr 这个参数， 则当 RPC 持续呼叫中，该次的呼叫是可以被中断的 (interrupted)。 | 没有 |
| rsize /wsize | 读出(rsize)与写入(wsize)的区块大小 (block size)。 这个设定值可以影响客户端与服务器端传输数据的缓冲记忆容量。一般来说， 如果在局域网络内 (LAN) ，并且客户端与服务器端都具有足够的内存，那这个值可以设定大一点， 比如说 32768 (bytes) 等，提升缓冲记忆区块将可提升 NFS 文件系统的传输能力！ 但要注意设定的值也不要太大，最好是达到网络能够传输的最大值为限。 | rsize=1024 /wsize=1024 |

### fstab 配置与自动挂载

如果 NFS 共享是手动挂载的，则重新引导后不会自动挂载共享。

Red Hat Enterprise Linux 提供了两种方法来在引导时自动挂载远程文件系统： /etc/fstab 文件和 autofs 服务

#### 使用 /etc/fstab挂载 NFS 文件系统 

从另一台计算机挂载 NFS 共享的另一个方法是向 /etc/fstab 文件中添加一行。行必须指定 NFS 服务器的主机名、要导出的服务器上的目录，以及 NFS 共享要挂载在本地计算机上的目录。您必须是 root 用户才能修改 /etc/fstab 文件。

`/etc/fstab`中行的一般语法如下：

`server:/usr/local/pub    /pub   nfs  defaults 0 0`

在执行此命令之前，客户端计算机上必须存在挂载点 /pub。将这一行添加到客户端系统上的 /etc/fstab 后，使用命令 mount /pub，挂载点 /pub 是从服务器挂载的。

挂载 NFS 导出的有效 /etc/fstab 条目应包含以下信息：

`server:/remote/export /local/directory nfs options 0 0`

变量 `server` 、`/remote/export`、`/local/directory` 和 `options` 与手动挂载 NFS 共享时所用的一样。

例如

`192.168.1.101:/public /public nfs defaults 0 0`

编辑 /etc/fstab 后，重新生成挂载单元，以便您的系统注册新配置：

`systemctl daemon-reload`

#### autofs 服务

使用 /etc/fstab 的一个缺点是，无论用户如何经常访问 NFS 挂载的文件系统，系统都必须指定资源来保持挂载的文件系统。对于一个或两个挂载没有问题，但当系统一次维护多个系统的挂载时，整体的系统性能可能会受到影响。/etc/fstab 的替代方法是使用基于内核的 automount 工具。

自动挂载程序由两个组件组成：

- 实现文件系统的内核模块，以及
- 执行所有其他功能的用户空间守护进程。

automount 工具可以自动挂载和卸载 NFS 文件系统（按需挂载），从而节省了系统资源。它可用于挂载其他文件系统，包括 AFS、SMBFS、CIFS、CIFS 和本地文件系统。

autofs 使用 /etc/auto.master （主映射）作为其默认主配置文件。可以使用 autofs 配置（在 /etc/sysconfig/autofs中）与名称服务交换机(NSS)机制结合使用其他支持的网络源和名称。为主映射中配置的每个挂载点运行一个 autofs 版本 4 守护进程的实例，因此可以在命令行中针对任何给定挂载点手动运行。autofs 版本 5 无法实现，因为它使用单个守护进程来管理所有配置的挂载点；因此，必须在主映射中配置所有自动挂载。这符合其他行业标准自动挂载程序的常规要求。挂载点、主机名、导出的目录和选项都可以在一组文件（或其他支持的网络源）中指定，而不必为每个主机手动配置它们。

##### 配置 autofs 

自动挂载程序的主要配置文件是 /etc/auto.master，也称为主映射。主映射列出了系统上 autofs- 控制的挂载点，以及它们相应的配置文件或网络来源，称为自动挂载映射。

master 映射的格式如下：

`mount-point map-name options`

使用这种格式的变量有：

- `mount-point`: 例如，autofs 挂载点 /home。
- `map-name`: 包含挂载点列表的映射源的名称，以及挂载这些挂载点的文件系统的位置。
- `options`: 如果提供，它们适用于给定映射中的所有条目，只要它们本身没有指定选项。这个行为与 autofs 版本 4 不同，其中选项是累积的。这已被修改来实现混合环境兼容性。

以下是来自 /etc/auto.master 文件的示例行（使用 cat /etc/auto.master显示）：

`/home /etc/auto.misc`

射的常规格式与主映射类似，但"选项"会出现在挂载点和位置之间，而不是在主映射中的条目末尾：

`mount-point   [options]   location`

##### 启动autofs

要启动自动挂载守护进程，请使用以下命令：

`$ systemctl start autofs`

要重启自动挂载守护进程，请使用以下命令：

`$ systemctl restart autofs`

使用给定配置时，如果进程需要访问 autofs 卸载的目录，如 /home/payroll/2006/July.sxc，则自动挂载守护进程会自动挂载该目录。如果指定了超时，则如果在超时时间内没有访问该目录，则目录会被自动卸载。

要查看自动挂载守护进程的状态，请使用以下命令：

`$ systemctl status autofs`

## NFS 的安全性和性能优化

NFS（Network File System）在默认情况下并不强制要求进行用户账号的认证，但可以根据配置文件实现一定程度的安全性控制和身份验证。

### 安全性要求

在传统的NFSv3及更早版本中，NFS通常依赖于客户端主机的信任关系来进行访问控制。这意味着NFS服务器通过检查请求的来源IP地址，并基于/etc/exports文件中的设置来决定哪些客户端可以挂载共享以及使用什么权限。这种机制下，用户的实际身份没有经过严格的身份验证，而是采用了匿名映射或“信任”模式，即客户端用户的UID和GID直接映射到服务器端的相应UID和GID上。

在安全性要求更高的环境中，NFS可以通过多种方式增强安全措施：

root Squashing：默认情况下，为了避免来自客户端的root用户以服务器的root权限操作共享文件系统，NFS服务器会将所有远程root用户映射为一个非特权用户（如nfsnobody），这被称为root squashing。

### 身份映射服务

NIS (Network Information Service) 或者后来的 NIS+ 可以用于集中式账户管理和身份验证。
LDAP 也可以作为身份验证后端，使得NFS能够与一个中心化的用户数据库同步，确保只有授权用户才能访问共享资源。

Kerberos 集成：在NFSv4中引入了对Kerberos身份验证协议的支持，提供了强大的加密和认证机制，确保只有经过身份验证的用户和服务才能进行交互。

虽然NFS本身不强制执行严格的用户账号认证，但在实际部署中，一般都会结合安全技术来实施必要的身份验证和授权控制。

### 防火墙的设定问题与解决方案

NFS 需要 rpcbind 服务，它动态分配 RPC 服务的端口，这可能导致配置防火墙规则时出现问题。为了让客户端能够通过防火墙访问 NFS 共享，你需要编辑 `/etc/sysconfig/nfs` 配置文件来控制所需 RPC 服务运行的端口。

#### 配置 NFS 服务端口

`/etc/sysconfig/nfs` 文件可能不是所有系统默认就有的。如果不存在，创建它并添加如下变量，将 `port` 替换为未使用的端口号（如果文件已存在，则取消注释并按需修改默认条目）：

- `MOUNTD_PORT=port`：控制 mountd (rpc.mountd) 使用的 TCP 和 UDP 端口。
- `STATD_PORT=port`：控制状态守护进程 (rpc.statd) 使用的 TCP 和 UDP 端口。
- `LOCKD_TCPPORT=port`：控制 nlockmgr (lockd) 使用的 TCP 端口。
- `LOCKD_UDPPORT=port`：控制 nlockmgr (lockd) 使用的 UDP 端口。

如果 NFS 启动失败，请检查 `/var/log/messages` 日志文件。通常，如果指定了一个已被占用的端口号，NFS 将无法启动。在编辑完 `/etc/sysconfig/nfs` 后，使用 `service nfs restart` 重启 NFS 服务，并运行 `rpcinfo -p` 命令来确认更改是否生效。

```shell
[root@www ~]$ vim /etc/sysconfig/nfs
STATD_PORT=1001   
LOCKD_TCPPORT=30001 
LOCKD_UDPPORT=30001 
MOUNTD_PORT=1002   

[root@www ~]$ /etc/init.d/nfs restart
[root@www ~]$ rpcinfo -p | grep -E '(mount|nlock)'
    100021    4   udp  30001  nlockmgr
    100021    4   tcp  30001  nlockmgr
    100005    3   udp   1002  mountd
    100005    3   tcp   1002  mountd
````

#### 配置防火墙以允许 NFS

为了配置防火墙允许 NFS 通信，需要执行以下步骤：

1. **允许 TCP 和 UDP 端口 2049 用于 NFS。**
2. **允许 TCP 和 UDP 端口 111（rpcbind/sunrpc）。**
3. **允许由 `MOUNTD_PORT="port"` 指定的 TCP 和 UDP 端口。**
4. **允许由 `STATD_PORT="port"` 指定的 TCP 和 UDP 端口。**
5. **允许由 `LOCKD_TCPPORT="port"` 指定的 TCP 端口。**
6. **允许由 `LOCKD_UDPPORT="port"` 指定的 UDP 端口。**

通过上述步骤，你可以确保防火墙正确配置，从而使 NFS 正常工作。

```shell
[root@www ~]$ vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.101/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.101/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT
[root@www ~]$ /usr/local/virus/iptables/iptables.rule
```

## 案例研究

### NFS服务端设置

1. 配置分享的文件系统（`/etc/exports`)

```shell
# 设置分享文件系统
[root@c1 ~]# vi /etc/exports
# [分享目录]   [第一部主机(权限)]     [可用主机名]    [可用通配符]
# /public：是共享的数据目录
# 192.168.1.102：表示IP(192.168.1.102/192.168.1.103/192.168.1.104)有权限连接
# rw：读写的权限
# sync：表示文件同时写入硬盘和内存
# no_root_squash：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份
/public    192.168.1.102(rw,sync,no_root_squash) 192.168.1.103(rw,sync,no_root_squash) 192.168.1.104(rw,sync,no_root_squash)
```

2. 启动NFS服务

```shell
# 启动rpcbind服务并设置开机启动
[root@c1 ~]# systemctl start rpcbind && systemctl enable rpcbind

# 启动nfs服务并设置开机启动
[root@c1 ~]# systemctl start nfs && systemctl enable nfs

# 查看rpcbind服务启动状态
[root@c1 ~]# systemctl status rpcbind 

  rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-12-22 14:31:40 CST; 21h ago
  Process: 814 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 830 (rpcbind)
    Tasks: 1
   CGroup: /system.slice/rpcbind.service
           └─830 /sbin/rpcbind -w

Dec 22 14:31:39 c1 systemd[1]: Starting RPC bind service...
Dec 22 14:31:40 c1 systemd[1]: Started RPC bind service...

# 查看nfs服务启动状态
[root@c1 ~]# systemctl status nfs
 nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Fri 2023-12-22 14:31:51 CST; 1 day 1h ago
 Main PID: 1678 (code=exited, status=0/SUCCESS)
    Tasks: 0
   CGroup: /system.slice/nfs-server.service
```

3. 检查NFS服务状态

```shell
# 查看日志
[root@c1 ~]# tail /var/log/messages
...
Dec 22 14:31:50 c1 rpc.statd[1256]: Version 1.3.0 starting
Dec 22 14:31:50 c1 rpc.statd[1256]: Flags: TI-RPC

# 查看NFS开放的端口
[root@c1 ~]# netstat -tulnp | grep -E '(rpc|nfs)'
tcp    0  0 0.0.0.0:39455   0.0.0.0:*   LISTEN  1256/rpc.statd
tcp    0  0 0.0.0.0:111     0.0.0.0:*   LISTEN  830/rpcbind
tcp    0  0 0.0.0.0:20048   0.0.0.0:*   LISTEN  1307/rpc.mountd
tcp6   0  0 :::51498    :::*    LISTEN  1256/rpc.statd
tcp6   0  0 :::111  :::*    LISTEN  830/rpcbind
tcp6   0  0 :::20048    :::*    LISTEN  1307/rpc.mountd
udp    0  0 0.0.0.0:990     0.0.0.0:*   830/rpcbind
udp    0  0 127.0.0.1:1008  0.0.0.0:*   1256/rpc.statd
udp    0  0 0.0.0.0:20048   0.0.0.0:*   1307/rpc.mountd
udp    0  0 0.0.0.0:45674   0.0.0.0:*   1256/rpc.statd
udp    0  0 0.0.0.0:111     0.0.0.0:*   830/rpcbind
udp6   0  0 :::990  :::*    830/rpcbind
udp6   0  0 :::20048    :::*    1307/rpc.mountd
udp6   0  0 :::54963    :::*    1256/rpc.statd
udp6   0  0 :::111  :::*    830/rpcbind

# 显示出目前这部主机的 RPC 状态
[root@c1 ~]# rpcinfo -p localhost
   program vers proto   port  service
   ...
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl

# 针对 nfs 这个程序检查其相关的软件版本信息 (仅察看 TCP 封包)
[root@c1 ~]# rpcinfo -t localhost nfs
program 100003 version 3 ready and waiting
program 100003 version 4 ready and waiting
```

4. NFS的联机观察

```shell
# 显示设定好的相关 exports 分享目录信息
[root@c1 ~]# showmount -e localhost
Export list for localhost:
/public 192.168.1.102,192.168.1.103,192.168.1.104

# 显示分享目录的权限设定
[root@c1 ~]# tail /var/lib/nfs/etab 
/public	192.168.1.102(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,no_root_squash,no_all_squash)
/public	192.168.1.103(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,no_root_squash,no_all_squash)
/public	192.168.1.104(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,no_root_squash,no_all_squash)

# *如果需要重新挂载，可以使用下面的命令
# 重新挂载 
[root@c1 ~]# exportfs -arv  
# 重新卸载   
[root@c1 ~]# exportfs -auv 
```

5. NFS的安全性

可以通过`/etc/exports`文件设置，或者通过特定端口的规则设置`/etc/sysconfig/nfs`，由于一般NFS服务仅对内网开放，所以也可以直接关闭防火墙。

```shell
# 设置iptables
[root@c1 ~]# iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.0/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT
[root@c1 ~]# iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.0/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT

# 内网直接关闭防火墙
[root@c1 ~]# systemctl stop firewalld.service
```

### NFS客户端设置

1. 启动NFS服务

```
# 启动rpcbind服务并设置开机启动
[root@c2 ~]# systemctl start rpcbind && systemctl enable rpcbind

# 启动nfs服务并设置开机启动
[root@c2 ~]# systemctl start nfs && systemctl enable nfs

# 查看rpcbind服务启动状态
[root@c2 ~]# systemctl status rpcbind 

  rpcbind.service - RPC bind service
   Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-12-22 14:31:40 CST; 21h ago
  Process: 814 ExecStart=/sbin/rpcbind -w $RPCBIND_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 830 (rpcbind)
    Tasks: 1
   CGroup: /system.slice/rpcbind.service
           └─830 /sbin/rpcbind -w

Dec 22 14:31:39 c2 systemd[1]: Starting RPC bind service...
Dec 22 14:31:40 c2 systemd[1]: Started RPC bind service...

# 查看nfs服务启动状态
[root@c2 ~]# systemctl status nfs
 nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Fri 2023-12-22 14:31:51 CST; 1 day 1h ago
 Main PID: 1678 (code=exited, status=0/SUCCESS)
    Tasks: 0
   CGroup: /system.slice/nfs-server.service
```

2. 手动挂载 NFS 服务器分享的资源

```shell
# 显示设定好的相关 exports 分享目录信息
[root@c2 ~]# showmount -e 192.168.1.101
Export list for 192.168.1.101:
/public 192.168.1.102,192.168.1.103,192.168.1.104

# 建立挂载点，实际挂载
[root@c2 ~]# mkdir -p /public
[root@c2 ~]# mount -t nfs 192.168.1.101:/public /public

# 查看挂载情况
[root@c2 ~]# mount | grep addr
192.168.1.101:/public on /public type nfs (rw,noexec,nosuid,
nodev,vers=4,addr=192.168.1.101,clientaddr=192.168.1.102)

# 如果需要卸载，可以使用以下命令
[root@c2 ~]# umount /public
```

3. 自动挂载 NFS 服务器分享的资源

通过系统配置 /etc/fstab 自动挂载

```shell
[root@c2 ~]# vim /etc/fstab
192.168.1.101:/public /public nfs defaults 0 0
```

autofs 自动挂载

```shell
# 建立主配置文件 /etc/auto.master ，并指定侦测的特定目录
[root@c2 ~]# vim /etc/auto.master
# [持续侦测的目录] [数据对应文件]
/misc /etc/auto.misc
# 添加/public目录，加上超时选项30s，以防挂载时失败导致卡死
/public  /etc/auto.nfs_public --timeout=30

# 建立数据对应文件内 (/etc/auto.nfs_public) 的挂载信息与服务器对应资源
[root@c2 ~]# vim /etc/auto.nfs_public
# 参数部分，只要最前面加个 - 符号即可！
# [本地端次目录]  [-挂载参数]  [服务器所提供的目录]
# 选项与参数：
# [本地端次目录] ：指的就是在 /etc/auto.master 内指定的目录之次目录
# [-挂载参数]    ：就是前一小节提到的 rw,bg,soft 等等的参数啦！nfsvers/vers=n设置使用NFS服务的NFS协议版本号。可有可无；
# [服务器所提供的目录] ：例如 192.168.1.101:/public 等
home  -nfsvers=3 192.168.1.101:/public/&

# 启动autofs服务并设置开机启动
[root@c2 ~]# systemctl start autofs
[root@c2 ~]# systemctl enable autofs

# 查看挂载是否成功
[root@worker2 opt] df -h
```

## NFS 常见错误与解决方法

### 客户端的主机名或 IP 网段不被允许使用

将你的 IP 加入 `/etc/exports` 这个档案中。

```shell
[root@www ~]$ mount -t nfs localhost:/home/test /mnt
mount.nfs: access denied by server while mounting localhost:/home/test
```

### 服务器或客户端某些服务未启动

解决的方法就是去重新启动 `rpcbind` 管理的其他所有服务

```shell
[root@clientlinux ~]# mount -t nfs 192.168.1.101:/home/test /mnt
mount: mount to NFS server '192.168.1.101' failed: System Error: Connection refused.
# 如果你使用 ping 却发现网络与服务器都是好的，那么这个问题就是 rpcbind 没有开啦！

[root@clientlinux ~]# mount -t nfs 192.168.1.101:/home/test /home/nfs
mount: mount to NFS server '192.168.1.101' failed: RPC Error: Program not registered.
# 注意看最后面的数据，确实有连上 RPC ，但是服务器的 RPC 告知我们，该程序无注册
```

### 被防火墙档掉了

当你一直无法顺利的连接 NFS 服务器，请先到服务器端，将客户端的 IP 完全放行，若确定这样就连的上， 那代表就是防火墙有问题啦！

可以通过`/etc/exports`文件设置，或者通过特定端口的规则设置`/etc/sysconfig/nfs`，由于一般NFS服务仅对内网开放，所以也可以直接关闭防火墙。

```shell
# 设置iptables
[root@www ~]$ vim /usr/local/virus/iptables/iptables.allow
iptables -A INPUT -i $EXTIF -p tcp -s 192.168.1.101/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT
iptables -A INPUT -i $EXTIF -p udp -s 192.168.1.101/24 -m multiport \
         --dport 111,2049,1001,1002,30001 -j ACCEPT
[root@www ~]$ /usr/local/virus/iptables/iptables.rule

# 内网直接关闭防火墙
[root@c1 ~]$ systemctl stop firewalld.service
```

## Additional Resources

### Documentation

1. [第十三章、文件服务器之一：NFS 服务器](http://cn.linux.vbird.org/linux_server/0330nfs_1.php)：鸟哥的Linux私房菜-服务器篇介绍关于网络分享文件系统的服务的部分，主要是讲解怎么搭建NFS服务器。
2. [nfsclient_autofs](https://linux.vbird.org/linux_server/centos6/0330nfs.php#nfsclient_autofs)
3. [Chapter 8. Network File System (NFS)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-nfs)

### Useful Websites

1. [Slurm安装3之NFS服务](https://www.michaelapp.com/posts/2018/2018-09-12-CentOS7.6-slurm%E5%AE%89%E8%A3%853/)：介绍NFS服务器怎么安装
2. [服务器集群（三）——NFS网络共享文件](https://www.cnblogs.com/linagcheng/p/16188939.html)：介绍NFS服务器怎么安装。
3. [nfsclient_autofs](https://cloud.tencent.com/developer/article/1902630?from=15425)
4. [nfsclient_autofs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_file_systems/mounting-nfs-shares_managing-file-systems#doc-wrapper)
5. [nfsclient_autofs](https://cloud.tencent.com/developer/article/1902630?from=15425)
6. [Getting started with NFS](https://www.redhat.com/sysadmin/getting-started-nfs)
7. [Learning NFS through server and client configuration ](https://www.redhat.com/sysadmin/nfs-server-client)
8. [分布式文件系统协议：NFS（Network File System）网络文件系统](https://developer.aliyun.com/article/1420265)
9. [Linux 多种方式实现文件共享（三）NFS 6](https://developer.aliyun.com/article/1578077)
10. [Linux NFS: The Basics and How to Run NFS in the Cloud](https://bluexp.netapp.com/blog/azure-anf-blg-linux-nfs-the-basics-and-running-nfs-in-the-cloud)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
3. [鸟哥的 Linux 私房菜 -- 服务器架设篇 第三版](http://cn.linux.vbird.org/linux_server/)
