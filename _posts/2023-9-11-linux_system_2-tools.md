---
layout: post
title:  Linux系统-CentOS/Rocky系统的工具包
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍CentOS/Rocky系统的系统工具包。

* TOC
{:toc}

<!--more-->

## Rocky group Development Tools

> 一组在 Rocky Linux 中用于软件开发的工具集合。

```shell
[root@localhost ~]# dnf group install "Development Tools"
```

## epel-release

> EPEL 仓库的启用包，为 Rocky Linux 或其他 Red Hat 衍生版提供额外的软件包

```shell
[root@localhost ~]# dnf install epel-release
Last metadata expiration check: 0:38:50 ago on Tue 16 Jan 2024 10:56:01 AM CST.
Dependencies resolved.
================================================================================
 Package               Architecture    Version            Repository       Size
================================================================================
Installing:
 epel-release          noarch          8-18.el8           extras           24 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 24 k
Installed size: 35 k
Is this ok [y/N]: y
Downloading Packages:
epel-release-8-18.el8.noarch.rpm                4.6 kB/s |  24 kB     00:05    
--------------------------------------------------------------------------------
Total                                           1.9 kB/s |  24 kB     00:12     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : epel-release-8-18.el8.noarch                           1/1 
  Running scriptlet: epel-release-8-18.el8.noarch                           1/1 
Many EPEL packages require the CodeReady Builder (CRB) repository.
It is recommended that you run /usr/bin/crb enable to enable the CRB repository.

  Verifying        : epel-release-8-18.el8.noarch                           1/1 

Installed:
  epel-release-8-18.el8.noarch                                                  

Complete!
[user@c1 ~]$ dnf clean all
[user@c1 ~]$ dnf makecache
```

## ntfs-3g

> 一个允许在多种操作系统上读写 NTFS 文件系统的软件，支持挂载和使用 Windows 的硬盘分区。

```shell
[root@localhost ~]# dnf install ntfs-3g
Extra Packages for Enterprise Linux 8 - x86_64  499 kB/s |  16 MB     00:33    
Last metadata expiration check: 0:00:03 ago on Tue 16 Jan 2024 11:35:45 AM CST.
Dependencies resolved.
================================================================================
 Package             Architecture  Version                    Repository   Size
================================================================================
Installing:
 ntfs-3g             x86_64        2:2022.10.3-1.el8          epel        133 k
Installing dependencies:
 ntfs-3g-libs        x86_64        2:2022.10.3-1.el8          epel        187 k

Transaction Summary
================================================================================
Install  2 Packages

Total download size: 320 k
Installed size: 690 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): ntfs-3g-2022.10.3-1.el8.x86_64.rpm        24 kB/s | 133 kB     00:05    
(2/2): ntfs-3g-libs-2022.10.3-1.el8.x86_64.rpm   34 kB/s | 187 kB     00:05    
--------------------------------------------------------------------------------
Total                                            29 kB/s | 320 kB     00:10     
Extra Packages for Enterprise Linux 8 - x86_64  1.6 MB/s | 1.6 kB     00:00    
Importing GPG key 0x2F86D6A1:
 Userid     : "Fedora EPEL (8) <epel@fedoraproject.org>"
 Fingerprint: 94E2 79EB 8D8F 25B2 1810 ADF1 21EA 45AB 2F86 D6A1
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : ntfs-3g-libs-2:2022.10.3-1.el8.x86_64                  1/2 
  Installing       : ntfs-3g-2:2022.10.3-1.el8.x86_64                       2/2 
  Running scriptlet: ntfs-3g-2:2022.10.3-1.el8.x86_64                       2/2 
  Verifying        : ntfs-3g-2:2022.10.3-1.el8.x86_64                       1/2 
  Verifying        : ntfs-3g-libs-2:2022.10.3-1.el8.x86_64                  2/2 

Installed:
  ntfs-3g-2:2022.10.3-1.el8.x86_64     ntfs-3g-libs-2:2022.10.3-1.el8.x86_64    

Complete!
```

挂载ntfs类型的文件系统

```shell
[user@c1 ~]$ mount -t ntfs-3g /dev/sdb1 /mnt/ntfs
```

## Linux man手册

> Linux 系统中的文档和命令参考手册，用于查询命令和程序的使用方法。

```shell
[user@c1 ~]$ yum install -y man -man-pages
```


## ibus中文输入法

> CentOS系统默认的是ibus输入法框架，用于在计算机上输入中文字符的软件工具。

1. 安装中文输入法相关软件

```shell
[user@c1 ~]$ yum install -y ibus ibus-table ibus-libpinyin
```

2. 配置输入法

```shell
[user@c1 ~]$ ibus-setup
```

- 运行后，输入法的配置界面就会弹出，`IBus Preference`设置被打开。
    - 在第一个`General`选项卡中，修改快捷键为自己喜欢的键（默认的更换输入法的按钮是`Windows+Space`）。
    - 在第二个`Input Method`选项卡中，选择`Add`，然后选择`Chinese`，然后选择`Intelligent Pinyin`。

- 按Windows键，打开`Settings`(系统设置)
    - 选择`Region&Language`，
    - 选择左下方的加号
    - 选择`Chinese`
    - 点击`Chinese（Intelligent Pinyin）`

- 如果右上角图标消失，可以通过`ibus-daemon -drx`命令，找回消失的IBus图标

3. 用户设置输入法

```shell
[user@c1 ~]$ imsettings-switch ibus
```

4. 重启电脑

```shell
[user@c1 ~]$ reboot
```

## firefox浏览器

> 一款开源的网页浏览器，提供安全且私密的上网体验。

```shell
[user@c1 ~]$ yum install firefox
```

## gedit文本编辑器

> GNOME 桌面环境下的默认文本编辑器，提供基本的文本编辑功能。

```shell
[user@c1 ~]$ yum install gedit
```

## Nvidia Drivers on Linux

> NVIDIA 显卡的官方驱动程序，用于在 Linux 系统上优化图形性能。

### CentOS 7 System

1. 检查系统是否有支持 CUDA 编程的 GPU，使用如下命令查看当前GPU的型号

```shell
# 查看当前GPU的型号
[user@c1 ~]$ lspci | grep -i nvidia
09:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050 Ti] (rev a1)
09:00.1 Audio device: NVIDIA Corporation GP107GL High Definition Audio Controller (rev a1)
```

2. 保证内核版本和源码版本一致

```shell
# 查看内核版本
[user@c1 ~]$ ls /boot/ | grep vmlinuz
vmlinuz-0-rescue-7ff1d9fe9c2f4d438935c3190dc1826d
vmlinuz-5.10.0-153.12.0.92.oe2203sp2.x86_64
# 查看源码版本
[user@c1 ~]$ rpm -aq | grep kernel-devel
kernel-devel-5.10.0-153.12.0.92.oe2203sp2.x86_64
```

3. 下载 NVIDIA Driver

在官网中输入自己GPU相关信息和OS类型，即可搜索出相应的NVIDIA Driver下载链接
[官网](https://www.nvidia.cn/Download/index.aspx?lang=cn)或者下载工具套件[CUDA Toolkit 12.3 Downloads](https://developer.nvidia.com/cuda-downloads)

可以直接下载或者通过`wget`命令下载

```shell
# 单独的驱动文件
[user@c1 ~]$ wget http://cn.download.nvidia.com/tesla/450.51.06/NVIDIA-Linux-x86_64-450.51.06.run
# 包含驱动文件的工具套件
[user@c1 ~]$ wget https://developer.download.nvidia.com/compute/cuda/12.3.0/local_installers/cuda_12.3.0_545.23.06_linux.runsudo 
sh cuda_12.3.0_545.23.06_linux.run
```

4. 禁用默认的显卡驱动

修改dist-blacklist.conf配置文件

```shell
[user@c1 ~]$ vi /lib/modprobe.d/dist-blacklist.conf
# 将nvidiafb注释掉，屏蔽默认显卡驱动的nouveau
# blacklist nvidiafb 
......
blacklist nouveau
options nouveau modeset-0
......
```

5. 重建initramfs image(文件系统)

```shell
# 备份当前文件系统镜像
[user@c1 ~]$ mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak

# 重建文件系统
[user@c1 ~]$ dracut /boot/initramfs-$(uname -r).img $(uname -r) 
```

6. 修改运行级别为文本模式，并重启

```shell
# 设置系统默认以命令行界面登录
[user@c1 ~]$ systemctl set-default multi-user.target

# 重启电脑
[user@c1 ~]$ reboot
```

7. 查看默认的显卡是否被禁用

```shell
# 查看默认的显卡
[user@c1 ~]$ lsmod | grep nouveau
```

8. 安装驱动和工具CUDA Toolkit

```shell
# 添加执行权限
[user@c1 ~]$ chmod +x cuda_12.3.0_545.23.06_linux.run 

# 启动程序
[user@c1 ~]$ ./cuda_12.3.0_545.23.06_linux.run 
```

### Rocky 8 System

#### 使用 NVIDIA 的官方仓库

一些用户可能更倾向于直接从源头使用驱动程序。他们的使用场景各不相同，但可能包括但不限于：

- 希望 GPU 在桌面或工作站上工作的用户
- 希望 GPU 在服务器上工作的用户
- 希望 GPU 在 Rocky Linux 上的云环境中以任何形式工作的用户

NVIDIA 的仓库可以在 Rocky Linux 上使用，因为它们为 RHEL 提供了驱动程序包，因此通常可以在 Rocky Linux 上正常工作。

注意：如果你的显卡已经过时或不再受 NVIDIA 提供的驱动程序支持，你可能要考虑使用 RPMFusion，因为它可能支持你的显卡。

初始系统设置：

```shell
# rpmfusion-free-release and epel-release are part of extras
% dnf install epel-release

# Get the major version and download the repo file
% curver="rhel$(rpm -E %rhel)"
% wget -O /etc/yum.repos.d/cuda-$curver.repo \
  http://developer.download.nvidia.com/compute/cuda/repos/$curver/$(uname -i)/cuda-$curver.repo

# CRB/PowerTools must be enabled
% crb enable

# Perform a dnf update now
% dnf update -y

# Reboot if you had a kernel update
% init 6
```

安装必要的驱动程序。你可以通过运行 `dnf module list` 来查看有哪些可用的驱动程序。

最简单的路径是使用他们的 dkms 驱动程序。这不是预先编译的，通常应该能正常工作。

`% dnf module install nvidia-driver:latest-dkms`

如果你更喜欢使用预先编译的驱动程序，你可以选择另一个模块流来安装它。

`% dnf module install nvidia-driver:latest`

#### 解决 NVIDIA 仓库驱动程序的问题

如果你发现 nouveau 和 NVIDIA 驱动程序之间存在冲突，你可能需要使用 grubby 添加黑名单。这仅适用于来自 NVIDIA 仓库的 NVIDIA 驱动程序。

```shell
% grubby --update-kernel=ALL --args="rd.driver.blacklist=nouveau modprobe.blacklist=nouveau"
% sed -i -e 's/GRUB_CMDLINE_LINUX="/GRUB_CMDLINE_LINUX="rd.driver.blacklist=nouveau modprobe.blacklist=nouveau /g' /etc/default/grub
```

## Htop

> [Htop在Centos7的安装](https://www.jianshu.com/p/5629e331f58d)

htop是Linux系统下一个基本文本模式的、交互式的进程查看器，主要用于控制台或shell中，可以替代top，或者说是top的高级版。

## lm_sensors

> [Centos 7. 6 Install lm_sensors](https://blog.csdn.net/hanzheng260561728/article/details/97011084)

Linux系统的硬件监控软件，可以获得主板，CPU工作电压、温度、风扇转速等信息。

### 3. hddtemp

> [如何在Linux上检查CPU和硬盘温度](https://www.codenong.com/2-view-check-cpu-hard-disk-temperature-linux/)

hddtemp仅现代硬盘驱动器具有温度传感器。hddtemp支持阅读S.M.A.R.T. 来自SCSI驱动器的信息也是如此。hddtemp可以用作简单的命令行工具或守护程序。 


## docker

> [centos7安装Docker详细步骤（无坑版教程）](https://cloud.tencent.com/developer/article/1701451)
> [CenterOS7搭建docker运行环境](https://blog.csdn.net/jacktree365/article/details/118793613#t9)

Docker是一种开源的应用容器引擎，基于Go语言并遵循Apache 2.0协议开源。docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

## tmux

> [CentOS7下使用tmux终端神器](https://www.jianshu.com/p/753e2ea4e6c7)

tmux是一个优秀的终端复用软件，即使非正常掉线，也能保证当前的任务运行，这一点对于 远程SSH访问特别有用，网络不好的情况下仍然能保证工作现场不丢失!此外，tmux完全使用键盘 控制窗口，实现窗口的切换功能。

## cmake

> [使用yum安装cmake](https://www.cnblogs.com/YingYue/p/6471581.html)

CMake是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的makefile或者project文件，能测试编译器所支持的C++特性,类似UNIX下的automake。

## scl

> [如何在 CentOS 上启用 软件集 Software Collections（SCL） ](https://linux.cn/article-6776-1.html)
> [How to install SCL packages](http://linuxdesktops.soton.ac.uk/softwarecollections.html#how-to-install-scl-packages)

软件集（Software Collections, SCL）源出现了，以帮助解决 RHEL/CentOS 下的这种问题。SCL 的创建就是为了给 RHEL/CentOS 用户提供一种以方便、安全地安装和使用应用程序和运行时环境的多个（而且可能是更新的）版本的方式，同时避免把系统搞乱。与之相对的是第三方源，它们可能会在已安装的包之间引起冲突。

`scl enable devtoolset-8 bash`

## gcc

> [CentOS 7升级gcc版本](https://blog.51cto.com/u_15080022/4375675)
> [gcc](https://gcc.gnu.org/)

GCC stands for GNU Compiler Collections which is used to compile mainly C and C++ language. It can also be used to compile Objective C and Objective C++. 

## Additional Resources

### Documentation

1. [Rocky Linux Repositories](https://wiki.rockylinux.org/rocky/repo/)


### Useful Websites

1. [CentOS下的CUDA安装和使用指南](https://blog.csdn.net/SL_World/article/details/108529209)
2. [TUTORIAL for NVIDIA GPU](https://forums.rockylinux.org/t/tutorial-for-nvidia-gpu/4234)
3. [openEuler 安装 Nvidia 驱动](https://forum.openeuler.org/t/topic/979/7)
4. [openEuler 21.9安装中文输入法](https://blog.csdn.net/weixin_44863237/article/details/122316333)
5. [Linux(centos)下安装man手册](https://blog.csdn.net/qq_40253126/article/details/131022866)
6. [Linux 环境下 NTFS 分区数据读写（ntfs-3g 方案）](https://blog.csdn.net/Nanan_an/article/details/133080183)
7. [centos7中yum安装ntfs-3g](https://www.cnblogs.com/ricksteves/p/11616363.html)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
