---
layout: post
title:  使用PXE+Kickstart无人值守安装服务
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍Linux系统服务器的Kickstart服务。

* TOC
{:toc}

<!--more-->

红帽企业版Linux提供了两种基本类型的介质：最小引导映像和完整安装映像（也称为二进制DVD）。

安装来源可以是以下几种形式之一：
- DVD：您可以将二进制DVD ISO映像刻录到DVD上，并配置安装程序从此磁盘安装软件包。
- 硬盘：您可以将二进制DVD ISO映像放置到硬盘上，并从此硬盘安装软件包。
- 网络位置：您可以将二进制DVD ISO映像或安装树（二进制DVD ISO映像提取的内容）复制到安装系统可以访问的网络位置，并使用以下协议通过网络进行安装：
    - NFS：二进制DVD ISO映像被放置在一个网络文件系统（NFS）共享中。
    - HTTPS、HTTP 或 FTP：安装树被放置在一个可通过HTTP、HTTPS或FTP访问的网络位置。

当从最小引导介质启动安装时，您必须始终配置一个额外的安装来源。当从完整的二进制DVD启动安装时，也可以配置另一个安装来源，但这不是必须的——二进制DVD ISO映像本身包含了安装系统所需的所有软件包，并且安装程序会自动将二进制DVD配置为来源。

您可以通过以下任何一种方式指定安装来源：

- 在安装程序的图形界面中：在图形安装开始后并且您选择了首选语言之后，会出现“安装摘要”屏幕。导航到“安装来源”屏幕并选择您想配置的来源。
- 使用引导选项：您可以在安装程序启动前指定自定义引导选项来配置安装程序。其中一个选项允许您指定要使用的安装来源。详见第23.1节《在引导菜单配置安装系统》中关于inst.repo=选项的描述。
- 使用Kickstart文件：您可以在Kickstart文件中使用install命令并指定安装来源。详见[第27.3.1节《Kickstart命令和选项》](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax#sect-kickstart-commands)中关于install Kickstart命令的详情，以及[第27章《Kickstart安装》](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations)中关于Kickstart安装的一般信息。

## Kickstart介绍

Kickstart安装提供了一种自动化安装过程的方法，无论是部分还是全部自动化。

Kickstart文件包含了安装程序通常会问到的所有问题的答案，例如系统应该使用哪个时区，硬盘应该如何分区，应该安装哪些软件包等。因此，在安装开始时提供一个预先准备好的Kickstart文件就可以实现自动安装，无需用户的任何干预。这对于一次性在大量系统上部署红帽企业版Linux特别有用。

Kickstart文件可以保存在一个单独的服务器系统上，并在安装过程中被各个计算机读取。这种安装方法可以使用单一的Kickstart文件来安装红帽企业版Linux到多台机器上，使其成为网络管理员和系统管理员的理想选择。

所有的Kickstart脚本及其执行的日志文件都会存储在/tmp目录下，以便于调试安装失败的问题。

需要注意的是，实际情况下日志文件通常不会直接存放在/tmp目录下，而是有特定的日志存放位置，/tmp目录在这里可能是个示例说明或者是针对特定上下文的情况。正确的日志位置取决于具体的Linux发行版和配置。

## 安装步骤

可以通过本地DVD、本地硬盘、NFS、FTP、HTTP或HTTPS来进行Kickstart安装。
要使用Kickstart，你必须：

1. 创建一个Kickstart文件。
2. 将Kickstart文件放到可移动媒体、硬盘或网络位置上，以便能够访问它。
3. 创建引导介质，这将用于启动安装过程。
4. 让安装源可用。
5. 开始Kickstart安装。

## 创建一个Kickstart文件

创建一个Kickstart文件有两种方式

1. 拷贝已安装系统下的`/root/anaconda-ks.cfg`，根据自身需求修改即可
2. 通过`system-config-kickstart`软件导出`ks.cfg`文件

### /root/anaconda-ks.cfg

[root@c1 ~]# cp /root/anaconda-ks.cfg ks.cfg

### system-config-kickstart

#### 下载system-config-kickstart

```shell
[root@c1 ~]# yum install system-config-kickstart -y
Dependencies Resolved

=====================================================================================
 Package                         Arch       Version                   Repository
                                                                                Size
=====================================================================================
Installing:
 system-config-kickstart         noarch     2.9.7-1.el7               base     348 k
Installing for dependencies:
 gnome-python2                   x86_64     2.28.1-14.el7             base      47 k
 gnome-python2-canvas            x86_64     2.28.1-14.el7             base      34 k
 libart_lgpl                     x86_64     2.3.21-10.el7             base      67 k
 libgnomecanvas                  x86_64     2.30.3-8.el7              base     226 k
 rarian                          x86_64     0.8.1-11.el7              base      98 k
 rarian-compat                   x86_64     0.8.1-11.el7              base      66 k
 system-config-date              noarch     1.10.6-3.el7.centos       base     591 k
 system-config-date-docs         noarch     1.0.11-4.el7              base     527 k
 system-config-keyboard          noarch     1.4.0-5.el7               base      33 k
 system-config-keyboard-base     noarch     1.4.0-5.el7               base     103 k
 system-config-language          noarch     1.4.0-9.el7               base     133 k
 usermode-gtk                    x86_64     1.111-6.el7               base     110 k
Updating for dependencies:
 usermode                        x86_64     1.111-6.el7               base     193 k

Transaction Summary
=====================================================================================
Install  1 Package  (+12 Dependent packages)
Upgrade             (  1 Dependent package)

Total size: 2.5 M
Total download size: 226 k
```

#### 设置Kickstart文件

`[root@c1 ~]# system-config-kickstart` 

##### [Basic Configuration]

> - lang （必需）
>> 
>> 语法规则： `lang [options]`
>> 含义：设置安装期间要使用的语言和安装系统上要使用的默认语言。
>> 选项与参数：
>> - `--addsupport=` - 添加对其他语言的支持。格式为使用逗号分开的列表，无空格。
>> 
>> 范例一：将语言设置为英语（美国）：
>> `lang en_US`
>> 范例二：将语言设置为英语并支持捷克语，德语，英语（英国）：
>> `lang en_US --addsupport=cs_CZ,de_DE,en_UK`
>>
> - keyboard （必需）
>>  
>> 语法规则 ： `keyboard [options]`
>> 含义：为系统设置一个或多个可用的键盘布局。
>> 选项与参数：
>> - `--vckeymap=` - 指定应使用的 VConsole 键映射。
>> - `--xlayouts=` - 指定 X 布局列表，该列表应当用作逗号分隔的列表，没有空格。
>> - `--switch=` - 指定布局切换选
项列表（在多个键盘布局之间切换的快捷方式）。必须使用逗号分开多个选项，没有空格。
>> 
>> 范例一：设置了一种键盘布局 （英语(US)）
>> `keyboard --vckeymap=us --xlayouts='us'`
>> 范例二：设置了两种键盘布局 （英语(US) 和集中式(qwerty)），并允许使用 AltShift 在它们之间进行切换：
>> `keyboard --xlayouts=us,'cz (qwerty)' --switch=grp:alt_shift_toggle`
>>
> - timezone （必需）
>> 
>> 语法规则： `timezone timezone [options]`
>> 含义：将系统时区设置为时区。
>> 选项与参数：
>> - `--UTC` - 如果存在，系统假定硬件时钟被设置为 UTC（格林威治 Mean）时间。
>> - `--nontp` - 禁用 NTP 服务自动启动。
>> - `--ntpservers=` - 指定用作没有空格的逗号分隔列表的 NTP 服务器列表。
>> 
>> 范例一：将时区设置为亚洲上海：
>> `timezone Asia/Shanghai`
>>
> - rootpw （必需）
>> 
>> 语法规则： `rootpw [--iscrypted|--plaintext] [--lock] password`
>> 含义：将系统的 root 密码设置为 password 参数。
>> 选项与参数：
>> - `--iscrypted` - 如果给出这个选项，则假设 password 参数已被加密。这个选项与 --plaintext 相互排斥。要创建加密的密码，您可以使用 python:（python -c 'import crypt,getpass;pw=getpass.getpass();print(crypt.crypt(pw) if (pw==getpass.getpass("Confirm: ")) else exit())'）
>> - `--plaintext` - 如果给出这个选项，则假设 password 参数为纯文本。这个选项与 --iscrypted 相互排斥。
>> - `--lock` - 如果给出这个选项，则默认锁定 root 帐户。这意味着 root 用户无法从控制台登录。这个选项还在图形和文本手动安装中禁用 Root 密码 页面。
>> 
>> 范例一：设置系统的root加密的密码：
>> `rootpw --iscrypted $1$EKlvbgYn$BMDc3EbzR0TZPfn7qzbOZ`
>> 范例二：设置系统的root纯文本的密码：
>> `rootpw --plaintext 123456`
>> 
> - reboot （可选）
>> 
>> 语法规则： `reboot [options]`
>> 含义：安装成功完成（无参数）后重新启动。通常，Kickstart 会显示信息并等待用户按任意键来重新引导系统。`reboot` 选项等同于 `shutdown -r` 命令。
>> 选项与参数：
>> - `--eject` - 在重新启动前尝试弹出可引导介质（DVD、USB 或其他介质）。
>> - `--kexec` - 使用 kexec 系统调用而不是执行完全重启，这样可立即将安装的系统加载到内存中，绕过通常由 BIOS 或固件执行的硬件初始化。
>> 
>> 范例一：安装成功完成后重新启动：
>> `reboot`
>> 
> - halt （可选）
>> 
>> 语法规则： `halt`
>> 含义：在成功完成安装后停止系统。这与手动安装类似，Anaconda 会显示一条信息并等待用户按键，然后再重新启动。在 Kickstart 安装过程中，如果没有指定完成方法，将使用这个选项作为默认选项。
>> 
>> 范例一：在成功完成安装后停止系统：
>> `halt`
>> 
> - poweroff （可选）
>> 
>> 语法规则： `poweroff`
>> 含义：在安装成功完成后关闭和关闭系统。poweroff 选项等同于 shutdown -p 命令。
>> 
>> 范例一：在安装成功完成后关闭和关闭系统：
>> `poweroff`
>> 
> - shutdown （可选）
>> 
>> 语法规则： `shutdown `
>> 含义：成功完成安装后关闭系统。shutdown Kickstart 选项等同于 shutdown 命令。
>> 
>> 范例一：在安装成功完成安装后关闭系统：
>> `shutdown`

```shell
# System language
lang en_US
# Keyboard layouts
keyboard 'us'
# System timezone
timezone Asia/Shanghai
# Root password
rootpw --iscrypted $1$EKlvbgYn$BMDc3EbzR0TZPfn7qzbOZ
#platform=x86, AMD64, or Intel EM64T
# Reboot after installation
reboot
```

##### [Installation Method]

> - install （可选）
>> 
>> 语法规则： `install type --url="location"`
>> 含义：默认安装模式。您必须指定来自 cdrom、harddrive、nfs、liveimg 或 url （用于 FTP、HTTP 或 HTTPS 安装）的安装类型。install 命令和安装方法命令必须位于单独的行中。
>> 选项与参数：
>> - `CDROM` - 从系统上的第一个光驱安装。
>> - `硬盘驱动器` - 从红帽安装树安装或本地驱动器中的完整安装 ISO 映像。该驱动器必须包含安装程序可挂载的文件系统： ext2、ext3、ext 4、vfat 或 xfs。
>> - `liveimg` - 从磁盘映像而不是软件包安装.映像可以是来自实时 ISO 映像的 squash fs.img 文件、压缩的 tar 文件，或者安装介质可以挂载的任何文件系统。
>> - `NFS` - 从指定的 NFS 服务器安装。
>> - `URL` - 使用 FTP、HTTP 或者 HTTPS 协议从远程服务器上的安装树进行安装。
>> 
>> 范例一：使用 FTP协议从远程服务器上的安装树进行安装：
>> `url --url="ftp://192.168.5.67/centos7"`

```shell
# Install OS instead of upgrade
install
# Use network installation
url --url="ftp://192.168.5.67/centos7" # 自己主机的ftp链接
```

##### [Boot Loader Options]

> - bootloader （必需）
>> 
>> 语法规则： `bootloader [options]`
>> 含义：指定引导装载程序的安装方式。
>> 选项与参数：
>> - `--append=` - 指定附加内核参数。要指定多个参数，使用空格分隔它们。
>> - `--boot-drive=` - 指定引导装载程序应写入的驱动器，因此要从哪个驱动器引导计算机。
>> - `--driveorder=` - 指定哪个驱动器最先在 BIOS 引导顺序中。
>> - `--location=` - 指定引导记录的写入位置。(有效值如下: MBR - 默认选项.具体要看驱动器是使用主引导记录（MBR）还是 GUID 分区表（GPT）方案；none - 不要安装引导装载程序；partition - 在包含内核的分区的第一个扇区安装引导装载程序。)
>> - `--password=` - 如果使用 GRUB2，则将引导装载程序密码设置为使用这个选项指定的密码。这应该用于限制对可传递任意内核选项的 GRUB2 shell 的访问。
>> - `--iscrypted` - 通常，当您使用 --password= 选项指定引导装载程序密码时，它会以纯文本形式存储在 Kickstart 文件中。
>> - `--timeout=` - 指定引导装载程序在引导默认选项前等待的时间（以秒为单位）。
>> 
>> 范例一：驱动器使用主引导记录：
>> `bootloader --location=mbr`
>> 
>> `bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda`

```shell
# System bootloader configuration
bootloader --location=mbr
```

##### [Partition Information]

> - clearpart （可选）
>> 
>> 语法规则： `clearpart [options]`
>> 含义：在创建新分区之前，从系统中删除分区。默认情况下不会删除任何分区。
>> 选项与参数：
>> - `--all` - 断掉系统中的所有分区。
>> - `--drives=` - 指定从中清除分区的驱动器。
>> - `--initlabel` - 通过在相应架构中为所有磁盘创建默认磁盘标签（已指定为格式化）
>> - `--list=` - 指定要清除哪些分区。如果使用此选项，这个选项将覆盖 --all 和 --linux 选项。可在不同的驱动器间使用。
>> - `--Linux` - 减少所有 Linux 分区。
>> - `--none` （默认）- 请勿删除任何分区。
>> 
>> 范例一：清除磁盘上所有分区并以默认磁盘标签初始化磁盘：
>> `clearpart --all --initlabel`
>> 范例二：保留磁盘上所有分区并以默认磁盘标签初始化磁盘：
>> `clearpart --none --initlabel`
>>
> - part 或 partition （必需）
>> 
>> 语法规则： `part|partition mntpoint --name=name --device=device --rule=rule [options]`
>> 含义：在系统上创建分区。
>> 选项与参数：
>> - `mntpoint` - 挂载分区的位置。该值必须是以下格式之一：`/path`, `swap`,`RAID.id`,`pv.id`,`biosboot`,`/boot/efi`
>> `--asprimary` - 强制将该分区分配为主分区。
>> - `--size=` - 最小分区大小，以 MiB 为单位。
>> - `--fstype=` - 为分区设置文件系统类型。有效值为 `xfs`、`ext2`、`ext3`、`ext4`、`swap`、`vfat`、`efi` 和 `biosboot`。
>> - `--fstype=` - 为分区设置文件系统类型。有效值为 xfs、ext2、ext3、ext4、swap、vfat、efi 和 biosboot。
>> - `--recommended` - 自动确定分区的大小。
>> - `--resize=` - 调整现有分区的大小。使用这个选项时，使用 --size= 选项指定目标大小（以 MiB 为单位），并使用 --onpart= 选项指定目标分区。
>> - `--ondisk=` 或 `--ondrive=` - 在现有磁盘上创建一个分区（由 part 命令指定）。这个命令总是创建分区。例如： --ondisk=sdb 将分区放在系统的第二个 SCSI 磁盘中。
>> 
>> 范例一：在现有sda磁盘上创建一个大小为100000lMiB，格式为ext4的主分区：
>> `part / --asprimary --fstype="ext4" --size=100000l`
>> 范例二：在现有sda磁盘上创建一个大小为405637MiB，格式为xfs的分区：
>> `part / --fstype="xfs" --ondisk=sda --size=405637`
>> 范例三：在现有sda磁盘上创建一个大小为51200MiB，格式为swap的swap分区：
>> `part swap --fstype="swap" --ondisk=sda --size=51200`

```shell
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part / --asprimary --fstype="ext4" --size=100000
part /home --asprimary --fstype="ext4" --size=100000
part /public2 --asprimary --fstype="ext4" --size=800000
```
##### [Network Configuration]

> - Network （可选）
>> 
>> 语法规则： `network [options]`
>> 含义：为目标系统配置网络信息，并在安装环境中激活网络设备。第一个 网络 命令中指定的设备会自动激活。--activate 选项还可明确要求激活设备。
>> 选项与参数：
>> - `--activate` - 在安装环境中激活这个设备。
>> - `--no-activate` - 不要在安装环境中激活这个设备。
>> - `--BOOTPROTO=` - dhcp、boot p、ibft 或 static 中的一个。默认选项为 dhcp ； dhcp 和 bootp 选项的处理方式相同。要禁用设备的 ipv4 配置，可使用 --noipv4 选项。
>> - `--device=` - 使用 网络 命令指定要配置的设备（最终在 Anaconda中激活）。
>> - `--ip=` - 设备的 IP 地址。
>> - `--ipv6=` - 设备的 IPv6 地址，格式为 地址[/前缀 长度] - 例如，3ffe:ffff:0:1::1/128 。如果省略了 前缀，则使用 64。您还可以使用 auto 进行自动配置，或使用 dhcp 进行仅 DHCPv6 配置（无路由器播发）。
>> - `--gateway=` - 作为单一 IPv4 地址的默认网关。
>> - `--ipv6gateway=` - 作为单一 IPv6 地址的默认网关。
>> - `--netmask=` - 安装系统的网络掩码。
>> - `--hostname=` - 安装的系统的主机名。主机名可以是 完全限定域名( FQDN)，格式为 host_name.domainname，也可以是没有域的短主机名。许多网络具有 动态主机配置 协议(DHCP)服务，该服务可自动提供带域名的连接的系统；为了允许 DHCP 分配域名，仅指定简短主机名。
>> 
>> 范例一：在现有sda磁盘上创建一个大小为100000lMiB，格式为ext4的主分区：
>> `part / --asprimary --fstype="ext4" --size=100000l`

```shell
# Network information
network  --bootproto=dhcp --device=localhost.localdomain
```

##### [Authentication Configuration]

> - auth 或 authconfig （可选）
>> 
>> 语法规则： `auth [options]`
>> 含义：使用 authconfig 命令为系统设置身份验证选项，也可以在安装完成后在命令行中运行该命令。
>> 选项与参数：
>> - `--enablenis` - 开启 NIS 支持。默认情况下，-- enablenis 使用它在网络上找到的任何域。域几乎应始终使用 --nisdomain= 选项手动设置。
>> - `--nisdomain=` - NIS 域名用于 NIS 服务.
>> - `--nisserver=` - 服务器要用于 NIS 服务（默认为广播）。
>> - `--useshadow` 或 `--enableshadow` - 使用影子密码。
>> - `--passalgo=` - 指定 sha256 以设置 SHA-256 哈希算法或 sha512 来设置 SHA-512 哈希算法。
>> 
>> 范例一：系统设置身份验证使用影子密码并设置SHA-512哈希算法：
>> `auth  --useshadow  --passalgo=sha512`


```shell
# System authorization information
auth  --useshadow  --passalgo=sha512
```

##### [Firewall Configuration]

> - SELinux （可选）
>> 
>> 语法规则： `selinux [--disabled|--enforcing|--permissive]`
>> 含义：设置已安装系统上 SELinux 的状态。默认 SELinux 策略为 enforcing。
>> 选项与参数：
>> - `--enforcing` - 使用正在强制执行 默认目标策略启用 SELinux。
>> - `--permissive` - 基于 SELinux 策略的输出警告，但并不强制执行该策略。
>> - `--disabled` - 在系统上完全禁用 SELinux。
>> 
>> 范例一：在系统上完全禁用 SELinux：
>> `selinux --disabled`
>> 
> - firewall （可选）
>> 
>> 语法规则： `firewall --enabled|--disabled device [options]`
>> 含义：设置已安装系统上 SELinux 的状态。默认 SELinux 策略为 enforcing。
>> 选项与参数：
>> - `--enabled` 或 `--enable` - 拒绝不是响应出站请求（如 DNS 回复或 DHCP 请求）的传入连接。如果需要访问在这个机器中运行的服务，您可以选择允许指定的服务通过防火墙。
>> - `--remove-service` - 不允许服务穿过防火墙。
>> - `--disabled` 或 `--disable` - 不配置任何 iptables 规则。
>> - `--trust=` - 在此处列出设备，如 em1，允许进出该设备的所有流量通过防火墙。要列出多个设备，请使用 --trust em1 --trust em2。不要使用逗号分隔的格式，如 --trust em1、em2。
>> - `incoming` - 使用以下一个或多个命令(--ssh, --smtp, --http, --ftp)替换，以允许指定服务穿过防火墙。
>> - `--port=` - 您可以使用 port:protocol 格式指定允许通过防火墙的端口。例如，要允许 IMAP 通过您的防火墙，可指定 imap:tcp。数字端口也可以明确指定；例如，要允许 UDP 数据包在端口 1234 到，请指定 1234:udp。要指定多个端口，用逗号将它们隔开。
>> - `--service=` - 此选项提供允许服务穿过防火墙的更高级别方法。有些服务（如 cups 、 vahi 等）需要打开多个端口或其他特殊配置才能使服务正常工作。
>> 
>> 范例一：在系统上完全禁用 SELinux：
>> `firewall --disabled`


```shell
# SELinux configuration
selinux --disabled
# Firewall configuration
firewall --disabled
```

##### [Display Configuration]

> - text （可选）
>> 
>> 语法规则： `text`
>> 含义：在文本模式下执行 Kickstart 安装.Kickstart 安装默认是以图形模式执行的。
>> 
>> 范例一：在文本模式下执行 Kickstart 安装：
>> `text`
>>
> - graphical （可选）
>> 
>> 语法规则： `graphical`
>> 含义：在图形模式模式下执行 Kickstart 安装.
>> 
>> 范例一：在图形模式下执行 Kickstart 安装：
>> `graphical`

```shell
# Use text mode install
text
firstboot --disable
```

##### [Package Selection]

> - %packages （可选）
>> 
>> 语法规则： `%packages [@package] %end`
>> 含义：设置已安装系统上 SELinux 的状态。默认 SELinux 策略为 enforcing。
>> 选项与参数：
>> - Specifying an environment(指定一个环境): 以 @^ 符号开头的行形式指定要安装的整个环境。
>> - Specifying groups(指定组) - 指定组，每个条目一行，以 @ 符号开头，然后是 *-comps-repository.architecture.xml 文件中给出的完整组群名称或者组群 ID。Core 组总是被选择 - 不需要在 %packages 部分指定它。
>> - Specifying individual packages(指定单独的软件包) - 根据名称指定单个软件包，每个条目对应一行。您可以在软件包名称中使用星号字符 (*) 作为通配符。
>> - Specifying profiles of module streams(指定模块流的配置集) - 使用配置集语法为模块流指定配置集（一个条目为一行）。
>> - Excluding environments, groups, or packages(排除环境、组群或者软件包) - 使用前导短划线 (-) 指定安装中排除的软件包或组。
>> 
>> 范例一：安装属于gnome-desktop-environment环境一部分的所有软件包：
>> `%packages`
>> `@^gnome-desktop-environment`
>> `%end`
>> 范例二：排除autofs的软件包：
>> `%packages`
>> `-autofs`
>> `%end`
>> 
> - %addon com_redhat_kdump （可选）
>> 
>> 语法规则： `%addon com_redhat_kdump [OPTIONS]`
>> 含义：`%addon com_redhat_kdump Kickstart`命令是可选的。这个命令配置 kdump 内核崩溃转储机制。
>> 选项与参数：
>> - `--enable` - 在安装的系统中启用 kdump。
>> - `--disable` - 在安装的系统中禁用 kdump。
>> - `--reserve-mb=` - 要为 kdump 保留的内存量，单位为 MiB。您还可以指定 auto 而不是数字值。
>> - `--enablefadump` - 在允许它的系统中（特别是 IBM Power Systems 服务器）启用固件辅助转储。
>> 
>> 范例一：在安装的系统中禁用 kdump：
>> `%addon com_redhat_kdump --enable --reserve-mb='auto'`
>> `%end`

##### [Pre-Installation Script]

> - %pre （可选）
>> 
>> 语法规则： `%pre [OPTIONS]`
>> 含义：`%pre` 脚本可用于激活和配置联网和存储设备。还可以使用安装环境中可用的脚本来运行脚本。如果您在继续安装之前有需要特殊配置的联网和存储，或者具有设置其他日志参数或环境变量的脚本，则添加 `%pre` 脚本非常有用。
>> 选项与参数：
>> - `--interpreter=` - 允许指定不同的脚本语言，如 Python
>> - `--erroronfail` - 显示错误并在脚本失败时暂停安装。
>> - `--log=` - 将脚本的输出记录到指定的日志文件中。请注意，无论您是否使用 --nochroot 选项，日志文件的路径都必须考虑。
>> 
>> 范例一：将脚本的输出记录到指定的日志文件中。：
>> `%pre --log=/tmp/ks-pre.log`
>> `%end`

##### [Post-Installation Script]

> - %post （可选）
>> 
>> 语法规则： `%post [OPTIONS]`
>> 含义：`%post` 脚本是安装后脚本，可在安装完成后运行，但在第一次重启系统前运行。您可以使用这部分来运行任务，比如系统订阅。
>> 选项与参数：
>> - `--interpreter=` - 允许指定不同的脚本语言，如 Python
>> - `--nochroot` - 允许您指定在 chroot 环境之外运行的命令。
>> - `--erroronfail` - 显示错误并在脚本失败时暂停安装。
>> - `--log=` - 将脚本的输出记录到指定的日志文件中。请注意，无论您是否使用 --nochroot 选项，日志文件的路径都必须考虑。
>> 
>> 范例一：将脚本的输出记录到指定的日志文件中：
>> `%post --nochroot --log=/mnt/sysroot/root/ks-post.log`
>> `%end`

#### 保存Kickstart文件

`[File] -> [Save] -> [ks.cfg]`

##### 文本界面的Kickstart配置

```shell
[root@c1 ~]# cat ks.cfg
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --iscrypted $1$EKlvbgYn$BMDc3EbzR0TZPfn7qzbOZ.
# Use network installation
url --url="ftp://192.168.5.67/centos7"
# System language
lang en_US
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use text mode install
text
firstboot --disable
# SELinux configuration
selinux --disabled

# Firewall configuration
firewall --disabled
# Network information
network  --bootproto=dhcp --device=c1
# Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai
# System bootloader configuration
bootloader --location=mbr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part / --asprimary --fstype="ext4" --size=10000
part /home --asprimary --fstype="ext4" --size=100000

%pre
echo "Beginning install CentOS 7...."
%end

%post
echo "Installed CentOS 7...."
%end
```

##### 图形界面的Kickstart配置

```shell
[root@c1 ~]# cat ks.cfg
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Use network installation
url --url="ftp://192.168.5.67/centos7"
# Use graphical install
graphical

# SELinux configuration
selinux --disabled
# Firewall configuration
firewall --disabled

# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8 --addsupport=zh_CN.UTF-8

# Network information
network --bootproto=dhcp --device=localhost.localdomain

# poweroff after installation
poweroff
# Root password
# rootpw --iscrypted $6$fL5F0BThvKgIear2$XWmZC5Hj3PaEPLfYBtSDz1SQF2QmPX1Cbk5PsyG6nwvRj4GH14KWYLcmM.F1aW6CUhagAb0ziPyAZaBurOIv/.
rootpw --plaintext meiyoumima
# System services
services --enabled="chronyd"
# System timezone
timezone Asia/Shanghai
user --groups=wheel --name=server --password=meimima
# X Window System configuration information
xconfig  --startxonboot
# System bootloader configuration
bootloader --location=mbr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part swap --fstype="swap" --ondisk=sda --size=5200
part /boot --fstype="xfs" --ondisk=sda --size=2048
part / --fstype="xfs" --ondisk=sda --size=115637


%packages
@^gnome-desktop-environment
@base
@compat-libraries
@core
@debugging
@desktop-debugging
@deveLopment
@dial-up
@directory-client
@file-server
@fonts
@gnome-apps
@gnome-desktop
@guest-desktop-agents
@input-methods
@internet-applications
@internet-browser
@java-platform
@multimedia
@network-file-system-client
@performance
@perl-runtime
@print-client
@ruby-runtime
@virtualization-client
@virtualization-hypervisor
@virtualization-tools
@web-server
@x11
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end
```

## 访问Kickstart文件

### 验证Kickstart文件

```shell
# 安装pykickstart
[root@c1 ~]# yum install pykickstart -y
Dependencies Resolved

=====================================================================================
 Package              Arch            Version                    Repository     Size
=====================================================================================
Updating:
 pykickstart          noarch          1.99.66.22-1.el7           base          367 k

Transaction Summary
=====================================================================================
Upgrade  1 Package

Total size: 367 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : pykickstart-1.99.66.22-1.el7.noarch                               1/2 
  Cleanup    : pykickstart-1.99.66.18-1.el7.noarch                               2/2 
  Verifying  : pykickstart-1.99.66.22-1.el7.noarch                               1/2 
  Verifying  : pykickstart-1.99.66.18-1.el7.noarch                               2/2 

Updated:
  pykickstart.noarch 0:1.99.66.22-1.el7                                              

Complete!

# 验证ks.cfg
[root@c1 ~]# ksvalidator ks.cfg 
```

#### 检查Kickstart语法

```shell
[root@c1 ~]# ksverdiff -f RHEL6 -t RHEL7
The following commands were removed in RHEL7:
monitor key interactive 
The following commands were deprecated in RHEL7:
upgrade 
The following commands were added in RHEL7:
eula liveimg sshkey btrfs ostreesetup snapshot reqpart nvdimm realm mount hmc 

The following options were added to the logvol command in RHEL7:
--metadatasize --poolname --thin --cachemode --thinpool --chunksize --mkfsoptions --profile --label --cachesize --resize --cachepvs 
The following options were removed from the logvol command in RHEL7:
--bytes-per-inode 

The following options were added to the firewall command in RHEL7:
--use-system-defaults --remove-service 
The following options were removed from the firewall command in RHEL7:
--telnet 

The following options were added to the clearpart command in RHEL7:
--cdl --list --disklabel 

The following options were added to the poweroff command in RHEL7:
--kexec 

The following options were added to the shutdown command in RHEL7:
--kexec 

The following options were added to the keyboard command in RHEL7:
--vckeymap --switch --xlayouts 

The following options were added to the timezone command in RHEL7:
--nontp --ntpservers 

The following options were added to the network command in RHEL7:
--bridgeslaves --teamconfig --bindto --interfacename --teamslaves --no-activate --ipv6gateway --bridgeopts --wpakey 

The following options were added to the autopart command in RHEL7:
--nohome --fstype --type --nolvm 

The following options were added to the raid command in RHEL7:
--label --chunksize --mkfsoptions 
The following options were removed from the raid command in RHEL7:
--bytes-per-inode 

The following options were added to the reboot command in RHEL7:
--kexec 

The following options were added to the method command in RHEL7:
--sslclientcert --mirrorlist --sslclientkey --sslcacert 

The following options were added to the repo command in RHEL7:
--sslclientcert --sslclientkey --sslcacert --install 

The following options were added to the part command in RHEL7:
--resize --mkfsoptions 
The following options were removed from the part command in RHEL7:
--end --start --bytes-per-inode 

The following options were added to the user command in RHEL7:
--gid 

The following options were added to the sshpw command in RHEL7:
--sshkey 

The following options were added to the bootloader command in RHEL7:
--disabled --boot-drive --nombr --extlinux --leavebootorder 
The following options were removed from the bootloader command in RHEL7:
--lba32 

The following options were added to the lang command in RHEL7:
--addsupport 

The following options were removed from the zfcp command in RHEL7:
--scsilun --scsiid 

The following options were removed from the xconfig command in RHEL7:
--driver --resolution --depth --videoram 

The following options were added to the url command in RHEL7:
--sslclientcert --mirrorlist --sslclientkey --sslcacert 

The following options were added to the partition command in RHEL7:
--resize --mkfsoptions 
The following options were removed from the partition command in RHEL7:
--end --start --bytes-per-inode 

The following options were added to the halt command in RHEL7:
--kexec 

The following options were removed from the driverdisk command in RHEL7:
--type 

The following options were added to the fcoe command in RHEL7:
--autovlan 
```

## Kickstart文件存放位置

Kickstart文件必须放置在以下位置之一：

1. 在可移动媒体上，如DVD或USB闪存盘。
2. 在连接到安装系统的硬盘上。
3. 在可以从安装系统访问的网络共享中。

通常，Kickstart文件会被复制到可移动媒体或硬盘上，或者在网络中提供访问。

将文件放置在网络位置补充了通常基于网络的Kickstart安装方式：系统通过`PXE`服务器引导，Kickstart文件从网络共享下载，文件中指定的软件包则从远程仓库下载。

使Kickstart文件能够被安装系统访问的过程与使安装源可用的过程是一样的，只是用Kickstart文件代替了安装的ISO镜像或树。

## 准备安装源

网络安装至少需要两个系统：

- 一台服务器 —— 运行DHCP服务器、TFTP服务器以提供引导文件，以及HTTP、FTP或NFS服务器来托管安装映像的系统。理论上，每个服务器可以运行在不同的物理系统上；为了简化起见，本节中的程序假设所有这些服务都运行在同一系统上。
- 客户端 —— 即您要安装红帽企业版Linux的系统。当安装开始时，客户端会查询DHCP服务器，从TFTP服务器获取引导文件，并从HTTP、FTP或NFS服务器下载安装映像。

为了准备网络安装，必须执行以下步骤：

1. 配置网络服务器（NFS、HTTPS、HTTP或FTP）以导出安装树或安装ISO映像。有关配置的详细步骤，请参阅[第3.3.3节，“网络上的安装源”](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/sect-making-media-additional-sources#sect-making-media-sources-network)。
2. 配置用于网络引导的TFTP服务器上的文件，配置DHCP，并在PXE服务器上启动TFTP服务。详情请参阅[第24.1节，“配置网络引导服务”](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/chap-installation-server-setup#sect-network-boot-setup)。
3. 引导客户端（即您想要安装红帽企业版Linux的系统），并开始安装。

### 配置文件传输的服务器（FTP）

```shell
# 安装FTP软件（Very Secure FTP Daemon）
[root@c1 ~]# yum -y install vsftpd
Dependencies Resolved

===================================================
 Package Arch    Version            Repository
                                              Size
===================================================
Installing:
 vsftpd  x86_64  3.0.2-29.el7_9     updates  173 k

Transaction Summary
===================================================
Install  1 Package

Total download size: 173 k
Installed size: 353 k
Downloading packages:
vsftpd-3.0.2-29.el7_9.x86_64. | 173 kB   00:07     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : vsftpd-3.0.2-29.el7_9.x86_64    1/1 
  Verifying  : vsftpd-3.0.2-29.el7_9.x86_64    1/1 

Installed:
  vsftpd.x86_64 0:3.0.2-29.el7_9                   

Complete!

# 挂载CentOS系统
[root@c1 ~]# mkdir /mnt/iso
[root@c1 ~]# mount -t iso9660 -o loop CentOS-7-x86_64-Minimal-2009.iso /mnt/iso
# 新建centos7目录，并将光盘镜像下的文件全部复制到centos7目录下
[root@c1 ~]# mkdir /var/ftp/centos7
[root@c1 ~]# cp -rf /mnt/iso/* /var/ftp/centos7/
# 复制kickstart配置文件到ftp目录下
[root@c1 ~]# cp ks.cfg /var/ftp/
# 开启ftp服务
[root@c1 ~]# systemctl start vsftpd
[root@c1 ~]# systemctl enable vsftpd
Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service to /usr/lib/systemd/system/vsftpd.service.
```

### 配置文件传输的服务器（TFTP）

```shell
# 安装TFTP服务端和xinetd网络守护进程服务程序
[root@c1 ~]# yum install tftp-server xinetd -y
Dependencies Resolved

=======================================================================
 Package           Arch         Version               Repository  Size
=======================================================================
Installing:
 tftp-server       x86_64       5.2-22.el7            base        47 k
 xinetd            x86_64       2:2.3.15-14.el7       base       128 k

Transaction Summary
=======================================================================
Install  2 Packages

Total download size: 175 k
Installed size: 325 k
Downloading packages:
(1/2): tftp-server-5.2-22.el7.x86_64.rpm          |  47 kB   00:06     
(2/2): xinetd-2.3.15-14.el7.x86_64.rpm            | 128 kB   00:06     
-----------------------------------------------------------------------
Total                                     9.7 kB/s | 175 kB  00:18     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 2:xinetd-2.3.15-14.el7.x86_64                       1/2 
  Installing : tftp-server-5.2-22.el7.x86_64                       2/2 
  Verifying  : tftp-server-5.2-22.el7.x86_64                       1/2 
  Verifying  : 2:xinetd-2.3.15-14.el7.x86_64                       2/2 

Installed:
  tftp-server.x86_64 0:5.2-22.el7     xinetd.x86_64 2:2.3.15-14.el7    

Complete!

# 安装PXE引导程序syslinux
[root@c1 ~]# yum -y install syslinux
Dependencies Resolved

===================================================
 Package    Arch     Version          Repository
                                              Size
===================================================
Installing:
 syslinux   x86_64   4.05-15.el7      base   990 k

Transaction Summary
===================================================
Install  1 Package

Total download size: 990 k
Installed size: 2.3 M
Downloading packages:
syslinux-4.05-15.el7.x86_64.r | 990 kB   00:16     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : syslinux-4.05-15.el7.x86_64     1/1 
  Verifying  : syslinux-4.05-15.el7.x86_64     1/1 

Installed:
  syslinux.x86_64 0:4.05-15.el7                    

Complete!

# 编辑tftp配置文件
[root@c1 ~]# vi /etc/xinetd.d/tftp 
service tftp
{
        socket_type             = dgram
        protocol                = udp # TFTP默认使用UDP协议
        wait                    = no # no表示客户机可以多台一起连接，yes表示客户机只能一台一台连接
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot # 指定TFTP根目录（引导文件的存储路径）
        disable                 = no # no表示开启TFTP服务
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}

# 复制 Linux系统的内核文件 到TFTP根目录下
[root@c1 ~]# cp /mnt/iso/isolinux/{vesamenu.c32,boot.msg} /var/lib/tftpboot/
[root@c1 ~]# cp /mnt/iso/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/

# 复制 PXE引导程序 到TFTP根目录下
[root@c1 ~]# cp /mnt/iso/Packages/syslinux-4.05-13.el7.x86_64.rpm .
[root@c1 ~]# rpm2cpio syslinux-4.05-13.el7.x86_64.rpm | cpio -dimv
[root@c1 ~]# mkdir /var/lib/tftpboot/pxelinux
[root@c1 ~]# cp ./usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/pxelinux

# 配置启动菜单文件
# 在ftfpboot目录下创建pxelinux.cfg
[root@c1 ~]# mkdir /var/lib/tftpboot/pxelinux.cfg
# 编辑TFTP预启动配置文件
[root@c1 ~]# vi /var/lib/tftpboot/pxelinux.cfg/default
default auto
prompt 0
timeout 600

label auto
  kernel vmlinuz
  append initrd=initrd.img method=ftp://192.168.5.67/centos7 ks=ftp://192.168.5.67/ks.cfg # ftp服务器地址

label linux text
  kernel vmlinuz
  append text initrd=initrd.img method=ftp://192.168.5.67/centos7

label linux rescue
  kernel vmlinuz
  append rescue initrd=initrd.img method=ftp://192.168.5.67/centos7

# 启动tftp服务（简单文件传输协议）
[root@c1 ~]# systemctl start tftp
[root@c1 ~]# systemctl enable tftp
Created symlink from /etc/systemd/system/sockets.target.wants/tftp.socket to /usr/lib/systemd/system/tftp.socket.
# 启动xinetd服务（网络守护进程服务程序）
[root@c1 ~]# systemctl start xinetd
[root@c1 ~]# systemctl enable xinetd
# 防火墙添加tftp服务
[root@c1 ~]# firewall-cmd --add-service=tftp --permanent
```

### 配置地址分配的服务器（DHCP）

```shell
# 下载dhcp
[root@c1 ~]# yum install dhcp -y

Dependencies Resolved

=====================================================================================
 Package               Arch        Version                        Repository    Size
=====================================================================================
Installing:
 dhcp                  x86_64      12:4.2.5-83.el7.centos.2       updates      515 k
Installing for dependencies:
 bind-export-libs      x86_64      32:9.11.4-26.P2.el7_9.16       updates      1.1 M
Updating for dependencies:
 dhclient              x86_64      12:4.2.5-83.el7.centos.2       updates      286 k
 dhcp-common           x86_64      12:4.2.5-83.el7.centos.2       updates      177 k
 dhcp-libs             x86_64      12:4.2.5-83.el7.centos.2       updates      133 k

Transaction Summary
=====================================================================================
Install  1 Package  (+1 Dependent package)
Upgrade             ( 3 Dependent packages)

Total download size: 2.2 M
Downloading packages:
(1/5): dhclient-4.2.5-83.el7.centos.2.x86_64.rpm              | 286 kB  00:00:08     
(2/5): dhcp-4.2.5-83.el7.centos.2.x86_64.rpm                  | 515 kB  00:00:06     
(3/5): dhcp-common-4.2.5-83.el7.centos.2.x86_64.rpm           | 177 kB  00:00:03     
(4/5): dhcp-libs-4.2.5-83.el7.centos.2.x86_64.rpm             | 133 kB  00:00:01     
(5/5): bind-export-libs-9.11.4-26.P2.el7_9.16.x86_64.rpm      | 1.1 MB  00:00:15     
-------------------------------------------------------------------------------------
Total                                                    83 kB/s | 2.2 MB  00:26    

# 修改DHCP服务的配置文件
[root@c1 ~]# vim /etc/dhcp/dhcpd.conf 
subnet 192.168.5.0 netmask 255.255.255.0 { # 指定为那个网段分配网络参数
  range 192.168.5.67 192.168.5.1; # 设置准备为客户端分配的IP地址范围
  option domain-name-servers 192.168.5.67; # 设置分配给客户端的服务器地址
  option routers 192.168.5.1; # 设置分配给客户端的网关地址
  default-lease-time 600; # 地址租赁时间 600秒后失效
  max-lease-time 7200;
  next-server 192.168.5.67; # 下一个要访问的地址，就是tftp地址，也是服务器地址
  filename "pxelinux.0";
}

# 启动DHCP服务
[root@c1 ~]# systemctl start dhcpd
[root@c1 ~]# systemctl enable dhcpd
Created symlink from /etc/systemd/system/multi-user.target.wants/dhcpd.service to /usr/lib/systemd/system/dhcpd.service.
# 查看DHCP服务
[root@c1 ~]# ss -tulanp | grep -w 67
udp    UNCONN     0      0         *:67                    *:*                   users:(("dhcpd",pid=32242,fd=7))
udp    UNCONN     0      0      *%virbr0:67                    *:*                   users:(("dnsmasq",pid=1674,fd=3))
```

### 防火墙设置

| Protocol used | Ports to open          |
|---------------|------------------------|
| `FTP`         | `21`                   |
| `HTTP`        | `80`                   |
| `HTTPS`       | `443`                  |
| `NFS`         | `2049`, `111`, `20048` |
| `TFTP`        | `69`                   |

#### 开启相关的端口

```shell
# 防火墙添加tftp服务
[root@c1 ~]# firewall-cmd --add-service=tftp --permanent
# 防火墙添加tftp服务端口
[root@c1 ~]# firewall-cmd --zone=public --add-port=69/tcp --permanent  
```

#### 直接关闭防火墙

```shell
# 关闭防火墙
[root@c1 ftp]# systemctl stop firewalld.service
# 临时关闭SELinux
[root@c1 ~]# setenforce 0
```

### 启动项更改

#### 安装efibootmgr工具

efibootmgr工具来管理EFI启动项

```shell
[root@localhost-localdomain ~]# yum -y install efibootmgr
================================================================================
 Package              Arch            Version               Repository     Size
================================================================================
Updating:
 efibootmgr           x86_64          17-2.el7              base           34 k
Updating for dependencies:
 efivar-libs          x86_64          36-12.el7             base           88 k

Transaction Summary
================================================================================
Upgrade  1 Package (+1 Dependent package)

Total size: 123 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : efivar-libs-36-12.el7.x86_64                                 1/4 
  Updating   : efibootmgr-17-2.el7.x86_64                                   2/4 
  Cleanup    : efibootmgr-15-2.el7.x86_64                                   3/4 
  Cleanup    : efivar-libs-31-4.el7.x86_64                                  4/4 
  Verifying  : efivar-libs-36-12.el7.x86_64                                 1/4 
  Verifying  : efibootmgr-17-2.el7.x86_64                                   2/4 
  Verifying  : efibootmgr-15-2.el7.x86_64                                   3/4 
  Verifying  : efivar-libs-31-4.el7.x86_64                                  4/4 

Updated:
  efibootmgr.x86_64 0:17-2.el7                                                  

Dependency Updated:
  efivar-libs.x86_64 0:36-12.el7                                                

Complete!
```

#### 调整顺序

```shell
# 列出 UEFI 启动项条目
[root@localhost-localdomain ~]# efibootmgr
BootCurrent: 0000
Timeout: 1 seconds
BootOrder: 0000,000B,0002,0001
Boot0000* CentOS
Boot0001* Network Card
Boot0002* Hard Drive
Boot000B* CentOS

# 更改 UEFI 启动项顺序, 网卡启动变为第一项
[root@localhost-localdomain ~]# efibootmgr -o 0001,000B,0002,0000
BootCurrent: 0000
Timeout: 1 seconds
BootOrder: 0001,000B,0002,0000
Boot0000* CentOS
Boot0001* Network Card
Boot0002* Hard Drive
Boot000B* CentOS

# 更改 UEFI 启动项顺序, 网卡启动变为第一项
[root@localhost-localdomain ~]# efibootmgr -o 0000,0001,000B,0002
BootCurrent: 0000
Timeout: 1 seconds
BootOrder: 0001,000B,0002,0000
Boot0000* CentOS
Boot0001* Network Card
Boot0002* Hard Drive
Boot000B* CentOS
```

## 问题

### TFTP prefix: Unable to locate configuration file

- 问题现象

```
PAELINUX 4.05 0x581e199a Copyright (C) 1994-2011 H. Peter Anvin et al
!PXE entry point found (we hope) at 98C7:0100 via plan A 
UNDI code segment at 98C? len 46CC 
UNDI data segment at 8C7C len C4B0
Getting cached packet 01 02 03
My IP address seens to be C0A80502 192.168.5.2
ip=192.168.5.2:192.168.5.67:192.168.5.1:255.255.255.0
BOOTIF=01-d8-bb-c1-15-b7-75
SYSUUID=aa68ed5e-0ee9-19ab-a4a1-d8bbc 115b775
TFTP prefix: /
Unable to locate configuration file
```

- 原因分析

找不到`default`配置文件以及SELinux阻止访问相关配置文件。

[root@c1 ~]# vi /var/log/message

```
...
Sep  9 17:29:35 c1 setroubleshoot: failed to retrieve rpm info for /var/lib/tftpboot/pxelinux.cfg/default
Sep  9 17:29:35 c1 setroubleshoot: SELinux is preventing /usr/sbin/in.tftpd from read access on the file /var/lib/tftpboot/pxelinux.cfg/default.
...
```

- 解决办法

```shell
# 编辑default文件
[root@c1 ~]# mkdir -p /var/lib/tftpboot/pxelinux.cfg
[root@c1 ~]# vi /var/lib/tftpboot/pxelinux.cfg/default
default auto
prompt 0
timeout 600

label auto
  kernel vmlinuz
  append initrd=initrd.img method=ftp://192.168.5.67/centos7 ks=ftp://192.168.5.67/ks.cfg # ftp服务器地址

label linux text
  kernel vmlinuz
  append text initrd=initrd.img method=ftp://192.168.5.67/centos7

label linux rescue
  kernel vmlinuz
  append rescue initrd=initrd.img method=ftp://192.168.5.67/centos7

# 临时关闭SELinux
[root@c1 ~]# setenforce 0
```

### TFTP Error - File not found

- 问题现象

```shell
Intel UNDI, PXE-2.1 (build 083)
Copyright (C) 1997-2000 Intel Corporation

This Product is covered by one or more of the following patents:
US6,578,884, US6,115,776 and US6,327,625

Realtek PCle GBE Family Controller Series v2.66 (05/26/17)

CLIENT MAC ADDR: D8 BB C1 15 B7 75 GUID: AA68ED5E-0EE9-19AB-A4A1-D8BBC115B775
CLIENT IP: 192.168.5.2 MASK: 255.255.0.0 DHCP IP: 192.168.5.67
GATEWAY IP: 192.168.5.1

TFTP.
PXE-T01: File not found
PXE-E3B: TFTP Error - File Not found
PXE-MOF: Exiting PXE ROM.
```

- 原因分析

在TFTP工作目录/var/lib/tftpboot/下没有找到/pxelinux.0文件。

- 解决办法

[root@c1 ~]# yum -y install syslinux
[root@c1 ~]# cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/

### RHEL7 Kickstart installation throws "No disk selected" message

将kickstart.cfg文件中的`clearpart --none`（保留现有的分区结构不变）改为`#clearpart --all --initlabel`（删除系统中的所有分区）

## 项目配置文件

### 服务器自动配置脚本(autoinstall.sh)

```shell
#!/bin/bash -x

# local resources
# resources files: `CentOS-7-x86_64-DVD-1804.iso`,`default`,`ks.cfg`
resources_dir="./resources"
# ftp location
ftp_loc="/var/ftp"
ftp_loc_os="$ftp_loc/centos7"
# mount location
iso_mnt="/mnt/iso"

# tftpboot pxelinux file
pxelinux_loc=/var/lib/tftpboot
pxelinux_cfg_loc=/var/lib/tftpboot/pxelinux.cfg

# dhcp configure file
dhcp_cfg_loc=/etc/dhcp/dhcpd.conf 

function check_linux_system(){
  system_version=`cat /etc/redhat-release`
  if [[ ${system_version} =~ "CentOS Linux release 7" ]]; then
    echo -e "\033[1;32m current system is ${system_version} \033[00m \n"
  else
    echo -e "\033[0;31m system is not CentOS; Only support CentOS\033[00m \n"
  fi
}

function check_pre_resources(){
  # check resources directory
  if [ ! -d $resources_dir ]; then
    mkdir $resources_dir
    echo -e "\033[0;31m can't find resources directory. We already create it, please put source file in it.\033[00m \n"
    exit 1
  fi
  
  # check centOS iso file
  if [ ! -f ./resources/*.iso ]; then
    echo -e "\033[0;31m can't find centOS system iso file; Please input *.iso in [resources] directory\033[00m \n"
  else
    centos_iso=$(ls ./resources/*.iso | awk -F '/' '{print $NF}')
    echo -e "\033[1;32m CentOS filename: ${centos_iso}\033[0m"
  fi
 
  # check kickstart configure file
  if [ ! -f ./resources/*.cfg ]; then
     echo -e "\033[0;31m can't find kickstart configure file; Please input ks.cfg in [resources] directory\033[00m \n"
  else
    ks_conf=$(ls ./resources/*.cfg | awk -F '/' '{print $NF}')
    echo -e "\033[1;32m kickstart configure filename: ${ks_conf}\033[0m"
  fi

  # check tftp default file
  if [ ! -f ./resources/default ]; then
     echo -e "\033[0;31m can't find tftp configure file; Please input default in [resources] directory\033[00m \n"
  else
    tftp_conf="default"
    echo -e "\033[1;32m tftp configure filename: ${tftp_conf}\033[0m"
  fi
}

function check_network(){
  ping -w 1 -c 1 mirrors.aliyun.com &> /dev/null
  if [ $? -eq 0 ]; then
    echo -e "\033[1;32m network is working.\033[00m"
    ip_addr=$(ifconfig  | egrep cast | awk '{print $2}' | head -n 1)
    echo -e "\033[1;32m ip_addr=$ip_addr.\033[00m"
    netmask_addr=$(ifconfig | egrep cast | awk '{print $4}' | head -n 1)
    echo -e "\033[1;32m netmask_addr=$netmask_addr.\033[00m"
    ip_cidr=$(echo ${ip_addr} | awk -F '.' '{$NF=0}{print $1"."$2"."$3"."$4}')
    echo -e "\033[1;32m ip_cidr=$ip_cidr.\033[00m"
    gateway_addr=$(ip route show  | awk '/default via/{print $3}')
    echo -e "\033[1;32m gateway_addr=$gateway_addr.\033[00m"
  else
    echo -e "\033[0;31m unable to connect to public network, please check network settings.\033[00m"
    exit 1
  fi
}


function change_yum_repo(){
  yum_backup=/etc/yum.repos.d/bak
  [[ ! -d $yum_backup ]] || mkdir /etc/yum.repos.d/bak
  [[ -d $yum_backup ]] && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak/
  curl -s http://mirrors.aliyun.com/repo/Centos-7.repo -o /etc/yum.repos.d/Centos-7.repo
  sed -i '/aliyuncs/d' /etc/yum.repos.d/Centos-7.repo
  curl -s http://mirrors.aliyun.com/repo/epel-7.repo -o /etc/yum.repos.d/epel-7.repo
  yum clean all && yum makecache
}

function install_resources_deploy(){
  # Install ftp software
  yum install vsftpd -y
  # Create ftp file transform location
  [ -d $ftp_loc_os ] || mkdir $ftp_loc_os
  
  # copy centOS install file to ftp transform location
  [ -d $iso_mnt ] || mkdir $iso_mnt
  mount -t iso9660 -o loop "$resources_dir/$centos_iso" $iso_mnt
  cp -rf $iso_mnt/* $ftp_loc_os
  umount -f $iso_mnt
  
  # copy kickstart configure file to ftp
  cp -rf "$resources_dir/$ks_conf" $ftp_loc
  
  # activate vsftpd service
  systemctl start vsftpd
  systemctl enable vsftpd
}

function tftp_deploy(){
  # Install tftp software
  yum install tftp-server xinetd -y

  # Install pxe syslinux
  yum install syslinux -y

  # copy kernel file to tftpboot directory
  cp -a $ftp_loc_os/isolinux/{vesamenu.c32,boot.msg} $pxelinux_loc
  cp -a $ftp_loc_os/images/pxeboot/{vmlinuz,initrd.img} $pxelinux_loc  

  # copy pxelinux file to tftpboot directory
  cp -a $ftp_loc_os/Packages/syslinux-4.05-13.el7.x86_64.rpm .
  rpm2cpio syslinux-4.05-13.el7.x86_64.rpm | cpio -dimv
  [ -d $pxelinux_loc ] || mkdir $pxelinux_loc
  cp ./usr/share/syslinux/pxelinux.0 $pxelinux_loc
  
  # set kernel start file
  [ -d $pxelinux_cfg_loc ] || mkdir $pxelinux_cfg_loc
  if [ -f "$resources_dir/default" ]; then
    cp -a $resources_dir/default $pxelinux_cfg_loc/default
    echo -e "copying $resources_dir/default file"
  else
cat << _PXECFG_ > $pxelinux_cfg_loc/default
default auto
prompt 0
timeout 600

label auto
  kernel vmlinuz
  append initrd=initrd.img method=ftp://$ip_addr/centos7 ks=ftp://$ip_addr/ks.cfg # ftp服务器地址

label linux text
  kernel vmlinuz
  append text initrd=initrd.img method=ftp://$ip_addr/centos7

label linux rescue
  kernel vmlinuz
  append rescue initrd=initrd.img method=ftp://$ip_addr/centos7
_PXECFG_
fi   

  # set tftp configure file
cat << _TFTP_ > /etc/xinetd.d/tftp
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = no
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
_TFTP_

  # activate tftp service
  systemctl start tftp && systemctl enable tftp
  systemctl start xinetd && systemctl enable xinetd
  
  # close firewall temp
  systemctl stop firewalld.service 
  # close selinux temp
  setenforce 0
  #firewall-cmd --add-service=tftp --permanent
  #firewall-cmd --zone=public --add-port=69/tcp --permanent  
  #firewall-cmd --zone=public --add-port=69/udp --permanent
}

function dhcp_deploy(){
  # install dhcp
  yum install dhcp -y
  # set dhcp configure file
cat << _DHCP_DEPLOY_ > $dhcp_cfg_loc
subnet $ip_cidr netmask $netmask_addr {
range $ip_addr $gateway_addr;
option domain-name-servers $ip_addr;
option routers $gateway_addr;
default-lease-time 600;
max-lease-time 7200;
next-server $ip_addr;
filename "pxelinux.0";
}
_DHCP_DEPLOY_

  systemctl start dhcpd ; systemctl enable dhcpd
}

check_linux_system
change_yum_repo
check_network
check_pre_resources
install_resources_deploy
tftp_deploy
dhcp_deploy

echo -e "\033[1;32m == KickStart complete！==\033[0m"
```

### kickstart配置脚本（ks.cfg）

```shell
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Use network installation
url --url="ftp://192.168.5.67/centos7"
# Use graphical install
graphical

# SELinux configuration
selinux --disabled
# Firewall configuration
firewall --disabled

# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8 --addsupport=zh_CN.UTF-8

# Network information
network --bootproto=dhcp --device=localhost.localdomain

# poweroff after installation
poweroff
# Root password
# rootpw --iscrypted $6$fL5F0BThvKgIear2$XWmZC5Hj3PaEPLfYBtSDz1SQF2QmPX1Cbk5PsyG6nwvRj4GH14KWYLcmM.F1aW6CUhagAb0ziPyAZaBurOIv/.
rootpw --plaintext meiyoumima
# System services
services --enabled="chronyd"
# System timezone
timezone Asia/Shanghai
user --groups=wheel --name=server --password=meimima
# X Window System configuration information
xconfig  --startxonboot
# System bootloader configuration
bootloader --location=mbr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part swap --fstype="swap" --ondisk=sda --size=5200
part /boot --fstype="xfs" --ondisk=sda --size=2048
part / --fstype="xfs" --ondisk=sda --size=115637


%packages
@^gnome-desktop-environment
@base
@compat-libraries
@core
@debugging
@desktop-debugging
@deveLopment
@dial-up
@directory-client
@file-server
@fonts
@gnome-apps
@gnome-desktop
@guest-desktop-agents
@input-methods
@internet-applications
@internet-browser
@java-platform
@multimedia
@network-file-system-client
@performance
@perl-runtime
@print-client
@ruby-runtime
@virtualization-client
@virtualization-hypervisor
@virtualization-tools
@web-server
@x11
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end
```

### ftp配置脚本（default）

```shell
default auto
prompt 0
timeout 600

label auto
  kernel vmlinuz
  append initrd=initrd.img method=ftp://192.168.5.67/centos7 ks=ftp://192.168.5.67/ks.cfg

label linux text
  kernel vmlinuz
  append text initrd=initrd.img method=ftp://192.168.5.67/centos7

label linux rescue
  kernel vmlinuz
  append rescue initrd=initrd.img method=ftp://192.168.5.67/centos7
```

### dhcp配置脚本（dhcpd.conf）

```shell
subnet 192.168.5.0 netmask 255.255.255.0 {
range 192.168.5.67 192.168.5.1;
option domain-name-servers 192.168.5.67;
option routers 192.168.5.1;
default-lease-time 600;
max-lease-time 7200;
next-server 192.168.5.67;
filename "pxelinux.0";
}
```

## Additional Resources

### Documentation

1. [27.2. How Do You Perform a Kickstart Installation?](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-howto#sect-kickstart-syntax-changes)
2. [3.3. Preparing Installation Sources](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/sect-making-media-additional-sources#sect-making-media-additional-sources)
3. [Chapter 24. Preparing for a Network Installation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/installation_guide/chap-installation-server-setup)
4. [27.3. Kickstart 语法参考](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax)
5. [A.3. Kickstart 文件中的脚本](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_8_installation/scripts-in-kickstart-file_kickstart-script-file-format-reference)
6. [Kickstart 中的软件包选择](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/9/html/automatically_installing_rhel/package-selection-in-kickstart_kickstart-script-file-format-reference)
7. [B.6. RHEL 安装程序提供的附加组件的 Kickstart 命令](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/automatically_installing_rhel/kickstart-commands-for-addons-supplied-with-the-rhel-installation-program_kickstart-commands-and-options-reference)

### Useful Websites

1. [CentOS KickStart 无人值守安装及自动部署ks脚本](https://www.cnblogs.com/hukey/p/14919346.html)
2. [PXE全自动批量安装linux系统【全程干货详解-保姆级教程】](https://www.zhihu.com/tardis/zm/art/374054171?source_id=1005)
3. [Sample kickstart configuration file for RHEL 7 / CentOS 7](https://www.golinuxhub.com/2017/07/sample-kickstart-configuration-file-for/)
4. [自动化部署(kickstart/cobbler)问题总结](https://blog.leonshadow.cn/763482/225.html)
5. [PXE boot -- kernel not found on TFTP server](https://serverfault.com/questions/237782/pxe-boot-kernel-not-found-on-tftp-server)
6. [RHEL7 Kickstart installation throws "No disk selected" message even after deleting the partitions and selecting the disk again](https://access.redhat.com/solutions/1296133)
7. [CentOS7中使用efibootmgr管理UEFI启动项](https://www.linuxprobe.com/centos-efibootmgr-uefi.html)

### Related Books

1. 鸟哥. 鸟哥的 Linux 私房菜: 基础学习篇. 人民邮电出版社, 2018.
2. 鸟哥. 鸟哥的 Linux 私房菜: 服务器架设篇. 机械工业出版社, 2008.
