---
layout: post
title:  Linux系统-CentOS/Rocky系统的安装
categories: [linux]
comments: true
tags: [Linux]
---

> 本文主要介绍CentOS/Rocky系统的安装，通过制作U盘启动工具来安装系统。

* TOC
{:toc}

<!--more-->

## 概要

CentOS 是一个广受欢迎的开源操作系统，它与 Red Hat Enterprise Linux (RHEL) 兼容，但免费提供。CentOS 社区版由 CentOS LLC 维护，并得到了 Red Hat 的支持。然而，在 2020 年底，Red Hat 宣布了一个重要的变化：CentOS Stream 将取代 CentOS Linux 作为 RHEL 的上游开发平台。这意味着 CentOS Linux 8 将不再接收更新和支持，而 CentOS Stream 成为了 RHEL 的滚动发行版，这引起了 CentOS 社区的一些不满。

Greg Kurtzer，CentOS 的联合创始人之一，在得知 CentOS Linux 8 将停止支持的消息后，决定创建一个新的项目，旨在保持 CentOS 社区版的精神，并继续提供一个稳定的企业级 Linux 发行版。因此，他宣布启动 `Rocky Linux` 项目。

自从 2020 年底宣布以来，`Rocky Linux` 快速获得了社区的支持和关注。它在 2021 年 5 月发布了第一个稳定版本 `Rocky Linux` 8.4，此后一直保持着活跃的开发和维护。

`Rocky Linux`的目标是提供一个与 RHEL 完全兼容的、稳定的企业级 Linux 发行版，同时保持社区驱动和开放性的精神。自发布以来，`Rocky Linux` 已经成为了许多企业和个人用户的首选替代方案。

## 整体安装流程

1. 获取安装源：下载`Rocky Linux`系统软件包
2. 制作USB启动盘：将`Rocky Linux`写入到USB中
3. USB启动安装：通过U盘启动的方式安装`Rocky Linux`系统

### 获取安装源

首先，下载ISO镜像文件以用于安装Rocky Linux。

用于此安装的Rocky Linux版本的最新ISO镜像可从以下位置下载：

`https://www.rockylinux.org/download/`

要直接从命令行下载 ISO，请使用 `wget` 命令：

`wget https://download.rockylinux.org/pub/rocky/8.5/isos/x86_64/Rocky-8.5-x86_64-minimal.iso`

Rocky Linux ISO 的命名遵循以下约定：

`Rocky-<主版本号#>.<次版本号#>-<架构>-<变体>.iso`

例如 `Rocky-8.5-x86_64-minimal.iso`

> 说明: Rocky项目网页列出了几个镜像，它们位于全球各地。 选择地理位置离您最近的镜像。 官方镜像列表可在 [这里](https://mirrors.rockylinux.org/mirrormanager/mirrors) 找到。

### 验证安装的 ISO 文件

如果您已经在现有的 GNU/Linux 发行版上下载了 Rocky Linux ISO，那么可以使用 sha256sum 程序验证下载的文件没有损坏。 接下来通过一个示例，演示如何验证 `Rocky-8.5-x86_64-minimal.iso`。

首先下载包含可用于 ISO 的官方校验和的文件。 在包含已下载的Rocky Linux ISO的文件夹中下载ISO的校验和文件时，键入：

`wget https://download.rockylinux.org/pub/rocky/8.7/isos/x86_64/CHECKSUM`

使用 `sha256sum` 工具来验证ISO文件的完整性，防止损坏或者被篡改。

`sha256sum -c CHECKSUM --ignore-missing`

这将检查先前下载的ISO文件的完整性，前提是该文件位于同一目录中。 输出应显示：

`Rocky-8.7-x86_64-minimal.iso: OK`

### 制作USB启动盘

#### 通过ultraiso制作启动盘

1. 安装并运行UltraISO软件[免费下载UltraISO软碟通官方中文版](https://cn.ultraiso.net/xiazai.html)

2. 用UltraISO软碟通打开ISO文件(`Rocky-8.5-x86_64-minimal.iso`)

3. 插入U盘，`选择启动`--->`写入硬盘镜像`

4. 选择相应的硬盘驱动器（U盘），然后选择`USB-HDD+`写入方式，`点击写入`

5.  待写入完成后，选择便捷启动选择`USB-HDD+`，`点击写入`

6. 修改卷标为`Rocky`。（如果没有修改系统安装时会因为找不到安装源，需要自己定义安装位置。）

#### 通过Linux命令制作启动盘

1. 将USB盘连接到该系统中，并执行 dmesg 命令查看相关的日志信息。在该日志的最后可以看到刚刚连接的USB盘所生成的一组信息，应类似如下：

```shell
[user@localhost ~]$ sudo dmesg | grep sd
[14421.647859] sd 15:0:0:0: [sdc] 241660916 512-byte logical blocks: (124 GB/115 GiB)
```

2. 切换为root用户。使用su命令，需要输入相应的密码。

```shell
[user@localhost ~]$ su - root
```

3. 确保USB盘没有被挂载。使用如下命令进行查询：

```shell
# 查看U盘盘符
[user@localhost ~]$ ls /dev/sd* | grep sd
/dev/sda
/dev/sda1
/dev/sda2
/dev/sda3
/dev/sda4
/dev/sda5
/dev/sda6
/dev/sdc
/dev/sdc1
# 查看U盘挂在情况
[user@localhost ~]$ findmnt /dev/sdc1
# 如果挂在则卸载
[user@localhost ~]$ findmnt /mnt/ntfs 
TARGET    SOURCE    FSTYPE OPTIONS
/mnt/ntfs /dev/sdc1 ntfs   ro,relatime,uid=0,gid=0,fmask=0177,dmask=077,nls=utf8
# umount  /mnt/ntfs
```

4. 使用dd命令将ISO安装镜像直接写入USB盘：

```shell
[user@localhost ~]$ dd if=/path/to/image.iso of=/dev/device bs=blocksize

# 使用您下载的ISO镜像文件的完整路径替换 /path/to/image.iso，使用之前由 dmesg 命令给出的设备名称替换device，同时设置合理的块大小（例如：512k）替换 blocksize，这样可以加快写入进度。

# 例如：如果该ISO镜像文件位于 /home/testuser/Downloads/Rocky-21.09-aarch64-dvd.iso，同时探测到的设备名称为sdb，则该命令如下：

[user@localhost ~]$ dd if=/home/testuser/Downloads/Rocky-21.09-aarch64-dvd.iso of=/dev/sdb bs=512k
```

5. 等待镜像写入完成，安全退出USB盘并拔掉。
镜像写入过程中不会有进度显示，当#号再次出现时，执行命令`sync`将数据同步写入磁盘。退出root帐户，拔掉USB盘。此时，您可以使用该USB盘作为系统的安装源。

### USB启动安装

1. 通过BIOS（『Boot』菜单中）将机器设置为U盘启动
2. 选择`Install Rocky Linux 8`引导安装
3. 选择英文安装
4. 自定义与系统区域设置相关的项目，包括键盘、语言支持、时间和日期。
5. 选择安装源以及要安装的其他软件包。
6. 自定义和更改目标系统的底层硬件，包括创建硬盘分区或LVM、指定要使用的文件系统，以及指定网络配置。
7. 为 root 用户帐户创建密码，或创建新的管理员或非管理员帐户
8. 单击主界面_Installation Summary_上的"开始安装"按钮。 安装将开始，安装程序将显示安装进度。
9. 完成所有必需的子任务且安装程序运行完毕后，单击Reboot System按钮来完成整个过程，系统将重启。

> 安装过程内容分为以下几部分：
> - Localization：(键盘、语言支持以及时间和日期)
> - Software：(安装源和软件选择)
> - System：(安装目的地以及网络和主机名)
> - User Settings：(设定root密码和创建用户)

## 名词解释

- `CentOS`: 一个免费且开源的 Linux 发行版，与 Red Hat Enterprise Linux (RHEL) 高度兼容。
- `Rocky Linux`: 一个社区驱动的 Linux 发行版，旨在与 RHEL 保持完全兼容，由 `CentOS` 的联合创始人 Greg Kurtzer 创建。
- `校验码`: 通过对文件块进行哈希算法形成的哈希值进行对比，以保证文件块的完整性。

## 问题集合

### 1. dracut-initqueue[2761]:Warning

#### 项目场景

通过USB安装Linux系统（Rocky系统）

#### 问题描述

通过Install Rocky引导加载时出现错误，进入不了安装界面，等待超时后提示信息（`ERROR: dracut-initqueue[2761]:Warning :dracut-initqueue timeout - starting timeout scripts`）。

#### 原因分析

制作U盘启动盘时，U盘的标签修改后无法在电脑启动过程中识别。

#### 解决办法

##### 解法一：修改标签名称

将U盘启动盘的卷标修改为`Rocky`，U盘的标签能够正确显示后就能安装系统。

##### 解法二：找出U盘的盘符

插入启动U盘，重新启动电脑，将光标移至`Test this media & install Rocky-8.5-x86_64`，按`e`进行编辑
修改掉默认的信息： 
将`vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=Rocky\x207\x20x86_64 rd.live.check` 改成`vmlinuz initrd=initrd.img linux dd`后，回车

通过查看列表中的硬盘信息，找出U盘的信息（`/dev/sdb1`），再重新启动，将光标移至`Test this media & install Rocky-8.5-x86_64`，按e键进行编辑，这次将`vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=Rocky\x207\x20x86_64 quiet`
改成 
`vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdb1   quiet`后，回车

##### 解法三：刻录光盘启动

直接找个光盘做成系统盘进行安装即可。

### 2. 无图形化界面

#### 项目场景

安装Linux系统（Rocky系统）

#### 问题描述

进入系统后，无图形界面显示。

#### 原因分析

系统安装进行界面时，选择的是最小安装方式，没有安装界面。

#### 解决办法

##### 解法一：CentOS方式安装桌面

1. 安装`X Window System`系统

```shell
# 安装X Window System系统
[root@localhost ~]$ yum groupinstall -y "X Window System"
```

2. 安装`GNOME Desktop`桌面

```shell
# 安装GNOME Desktop桌面
[root@localhost ~]$ yum groupinstall -y "GNOME Desktop"

# 查看是否安装完成
[root@localhost ~]$ yum grouplist
```

3. 修改启动运行级别

```shell
# 设置系统默认以图形界面登录
[root@localhost ~]$ systemctl set-default graphical.target
```

4. 切换运行级别

```shell
# 重启验证
[root@localhost ~]$ reboot

# 如果仍未出现图形界面，切换Linux系统的运行级别为图形模式
[root@localhost ~]$ init 5
```

##### 解法二：Rocky方式安装桌面

1. 下载Rocky ISO镜像并安装系统，更新软件源

```shell
# 更新软件源
[user@localhost ~]$ sudo dnf update
```

2. 安装字库

```shell
[user@localhost ~]$ sudo dnf install dejavu-fonts liberation-fonts gnu-*-fonts google-*-fonts
```

3. 安装Xorg

```shell
# 安装xorg相关软件包
[user@localhost ~]$ sudo dnf install xorg-*

# *必要软件包
[user@localhost ~]$ sudo dnf install xorg-x11-apps xorg-x11-drivers xorg-x11-drv-ati \
	xorg-x11-drv-dummy xorg-x11-drv-evdev xorg-x11-drv-fbdev xorg-x11-drv-intel \
	xorg-x11-drv-libinput xorg-x11-drv-nouveau xorg-x11-drv-qxl \
	xorg-x11-drv-synaptics-legacy xorg-x11-drv-v4l xorg-x11-drv-vesa \
	xorg-x11-drv-vmware xorg-x11-drv-wacom xorg-x11-fonts xorg-x11-fonts-others \
	xorg-x11-font-utils xorg-x11-server xorg-x11-server-utils xorg-x11-server-Xephyr \
	xorg-x11-server-Xspice xorg-x11-util-macros xorg-x11-utils xorg-x11-xauth \
	xorg-x11-xbitmaps xorg-x11-xinit xorg-x11-xkb-utils
```

4. 安装GNOME及组件

```shell
# GNOME及组件
[user@localhost ~]$ sudo dnf install adwaita-icon-theme atk atkmm at-spi2-atk at-spi2-core baobab \
	abattis-cantarell-fonts cheese clutter clutter-gst3 clutter-gtk cogl dconf \
	dconf-editor devhelp eog epiphany evince evolution-data-server file-roller folks \
	gcab gcr gdk-pixbuf2 gdm gedit geocode-glib gfbgraph gjs glib2 glibmm24 \
	glib-networking gmime30 gnome-autoar gnome-backgrounds gnome-bluetooth \
	gnome-builder gnome-calculator gnome-calendar gnome-characters \
	gnome-clocks gnome-color-manager gnome-contacts gnome-control-center \
	gnome-desktop3 gnome-disk-utility gnome-font-viewer gnome-getting-started-docs \
	gnome-initial-setup gnome-keyring gnome-logs gnome-menus gnome-music \
	gnome-online-accounts gnome-online-miners gnome-photos gnome-remote-desktop \
	gnome-screenshot gnome-session gnome-settings-daemon gnome-shell \
	gnome-shell-extensions gnome-software gnome-system-monitor gnome-terminal \
	gnome-tour gnome-user-docs gnome-user-share gnome-video-effects \
	gnome-weather gobject-introspection gom grilo grilo-plugins \
	gsettings-desktop-schemas gsound gspell gssdp gtk3 gtk4 gtk-doc gtkmm30 \
	gtksourceview4 gtk-vnc2 gupnp gupnp-av gupnp-dlna gvfs json-glib libchamplain \
	libdazzle libgdata libgee libgnomekbd libgsf libgtop2 libgweather libgxps libhandy \
	libmediaart libnma libnotify libpeas librsvg2 libsecret libsigc++20 libsoup \
	mm-common mutter nautilus orca pango pangomm libphodav python3-pyatspi \
	python3-gobject rest rygel simple-scan sushi sysprof tepl totem totem-pl-parser \
	tracker3 tracker3-miners vala vte291 yelp yelp-tools \
	yelp-xsl zenity
```

5. 启动gdm显示管理器

```shell
[user@localhost ~]$ sudo systemctl enable gdm
```

6. 设置系统默认以图形界面登录

```shell
# 设置系统默认
[user@localhost ~]$ sudo systemctl set-default graphical.target
# 重启验证
[user@localhost ~]$ sudo reboot
```

7. 当gdm不能工作

```shell
# 如果默认安装了gdm，则停用gdm
[user@localhost ~]$ sudo systemctl disable gdm

# 安装lightdm显示管理器替代
[user@localhost ~]$ sudo dnf install lightdm lightdm-gtk

# 设置默认桌面为GNOME，通过root权限用户设置
[user@localhost ~]$ echo 'user-session=gnome' >> /etc/lightdm/lightdm.conf.d/60-lightdm-gtk-greeter.conf

# 启动lightdm显示管理器
[user@localhost ~]$ sudo systemctl enable lightdm

# 设置系统默认以图形界面登录
[user@localhost ~]$ sudo systemctl set-default graphical.target

# 重启验证
[user@localhost ~]$ sudo reboot
```

### 3. su: Permission denied

#### 项目场景

Linux系统切换root用户

#### 问题描述

普通用户切换root用户时出现`Permission denied`

#### 原因分析

首先需要排除密码错误、密码过期、用户锁定等问题。

其次是去查看PAM（Pluggable Authentication Modules）模块，因为PAM（Pluggable Authentication Modules）负责系统中很多应用程序的登录认证，包括sshd、vsftpd、su等。

```shell
# 查看su的PAM认证配置
[root@c1 pam.d]# cat /etc/pam.d/su
...
# Uncomment the following line to require a user to be in the "wheel" group.
auth           required        pam_wheel.so use_uid
auth            substack        system-auth
```

通过查看`su`的PAM配置文件中有`auth required pam_wheel.so use_uid`，使用su命令则该用户必须在wheel用户组中，而普通用户没有在wheel用户组中。

#### 解决办法

##### 解法一：将普通用户加入wheel组

```shell
[user@localhost ~]$ sudo usermod -a -G wheel user
```

##### 解法二：注释该行

```shell
# 修改su的PAM认证配置
[root@c1 pam.d]# vi /etc/pam.d/su
...
# Uncomment the following line to require a user to be in the "wheel" group.
# auth           required        pam_wheel.so use_uid
auth            substack        system-auth
```

## Additional Resources

### Documentation

1. [Installing Rocky Linux 8](https://docs.rockylinux.org/guides/8_6_installation/)
2. [kde_installation](https://docs.rockylinux.org/guides/desktop/kde_installation/)

### Useful Websites

1. [第四章、安装 CentOS 5.x 与多重开机小技巧](http://cn.linux.vbird.org/linux_basic/0157installcentos5_2.php)
2. [UltraISO制作启动盘安装CentOS7](https://www.cnblogs.com/xuanbjut/p/13092891.html)
3. [UltraISO刻录CentOS 7安装指南](https://www.cnblogs.com/larry-luo/p/11102321.html)
4. [Linux磁盘与启动程序](https://blog.csdn.net/qq_50824019/article/details/124311133)
5. [centos7安装图形化界面图文详解](https://cloud.tencent.com/developer/article/2069955)
6. [su 命令报错 su: Permission denied](https://www.cnblogs.com/xzongblogs/p/15216661.html)
7. [安装GNOME1](https://docs.openeuler.org/zh/docs/22.03_LTS_SP2/docs/desktop/Install_GNOME.html#)
8. [安装GNOME2](https://blog.csdn.net/ZXW_NUDT/article/details/128593216)
9. [安装图形化界面ukui](https://blog.csdn.net/m0_51378263/article/details/115282335)
10. [Rocky Linux 9 从入门到精通001](https://www.rockylinux.cn/notes/rocky-linux-system-introduction-and-system-burning.html)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
