---
layout: post
title:  Linux系统-CentOS/Rocky系统的再生龙备份
categories: [linux]
comments: true
tags: [Linux]
---

> 本文主要介绍CentOS/Rocky系统的再生龙备份。再生龙备份的流程和一般的装机流程有点类似，只是在选择备份源和保存源位置的顺序要注意，首先选择的是`保存镜像的位置`，然后是选择`保存/恢复硬盘的位置`。


* TOC
{:toc}

<!--more-->

## Clonezilla 简介

Clonezilla 是一个很好的系统克隆工具，中文名叫“再生龙”，它可以说是吸取了 Norton Ghost 和 Partition Image 的优点。即不仅支持对整个系统进行克隆，而且也可以克隆单个的分区，这种灵活性可能更能适应备份者的需要 。 

Clonezilla 是一款类似于 True Image® 或 Norton Ghost® 的分区和磁盘映像/克隆程序，主要用于系统部署、裸机备份和恢复。

Clonezilla 提供三种版本：Clonezilla Live（适合单机备份和恢复）、Clonezilla Lite Server（支持大规模部署，包括一对一、一对多和比特流克隆）、以及 Clonezilla SE（服务器版，需配合 DRBL 使用，支持一对一、一对多克隆）。

### 主要特点

- 支持多种文件系统，包括 GNU/Linux 的 ext2/3/4、reiserfs、reiser4、xfs、jfs、btrfs、f2fs、nilfs2；MS Windows 的 FAT12/16/32、exFAT、NTFS；Mac OS 的 HFS+、APFS；FreeBSD、NetBSD、OpenBSD 的 UFS；Minix 的 minix；以及 VMWare ESX 的 VMFS3 和 VMFS5。
- 支持 LVM2（不支持 LVM 版本 1）和 LUKS（Linux 统一密钥设置）。
- 支持重新安装 grub（版本 1 和版本 2）和 syslinux 引导加载程序。
- 支持 MBR 和 GPT 分区格式，并能在 BIOS 或 uEFI 机器上启动。
- 支持无人值守模式，几乎所有步骤可通过命令行完成。
- 支持将单一映像恢复到多个本地设备。
- 支持使用 ecryptfs 加密映像文件。
- 支持通过 PXE 和 Wake-on-LAN 远程使用 Clonezilla SE 进行大规模克隆。
- 支持使用 Bittorrent 模式在 Clonezilla Lite Server 中进行大规模部署。
- 映像文件可存放于本地磁盘、SSH 服务器、Samba 服务器、NFS 服务器或 WebDAV 服务器。
- 支持使用 AES-256 加密保障数据访问、存储和传输的安全。
- 支持使用 Partclone（默认）、Partimage（可选）、ntfsclone（可选）或 dd 进行分区映像或克隆。
- 通过 drbl-winroll 自动更改克隆后的 MS Windows 机器的主机名、组和 SID。

### 最低系统要求

- 处理器：X86 或 x86-64
- 内存：196 MB
- 启动设备：CD/DVD 驱动器、USB 端口、PXE 或硬盘

### 限制

- 目标分区必须等于或大于源分区。
- 尚未实现差异/增量备份。
- 尚未实现在线克隆。待克隆的分区必须卸载。
- 由于映像格式限制，无法浏览或挂载映像文件，无法从中恢复单个文件。
- 尚未实现使用多张 CD 或 DVD 恢复 Clonezilla Live。

### 许可证

Clonezilla 本身遵循 GNU 通用公共许可证第 2 版（GPL v2）。运行 Clonezilla 需要使用许多自由和开源软件，如 Linux 内核和最小化的 GNU/Linux 操作系统。

### 选择合适的 Clonezilla 版本

- Clonezilla Live：使用 CD/DVD 或 USB 闪存驱动器启动并运行 Clonezilla（仅支持一对一克隆）。
- Clonezilla Lite Server：使用 Clonezilla Live 进行大规模克隆（支持一对一、一对多、多对多和比特流克隆）。
- Clonezilla SE：包含在 DRBL 中，需要先设置 DRBL 服务器才能使用 Clonezilla 进行大规模克隆（支持一对一、一对多克隆）。

## 使用Clonezilla（再生龙）

1. 下载镜像

再生龙的镜像网站[下载](http://clonezilla.nchc.org.tw/clonezilla-live/download/download.php) Clonezilla（再生龙）

2. UltraISO 工具

启动盘的制作有很多种方式，这里选择 UltraISO 工具。

在工具栏选择【文件】->【打开】，然后下载好的镜像文件：

![Link](https://upload-images.jianshu.io/upload_images/25633953-1957cf87b05e5aba?imageMogr2/auto-orient/strip|imageView2/2/w/782/format/webp)

在工具栏选择【启动】->【写入硬映像】

![Link](https://img2022.cnblogs.com/blog/1664108/202203/1664108-20220322081654234-1450769574.png)

选择被制作启动盘的U盘，写入方式有zip和hdd两种，一般我们选择hdd或hdd+，选择了写入方式之后要先格式化，然后【格式化】、【写入】。

![Link](https://img2022.cnblogs.com/blog/1664108/202203/1664108-20220322081702632-682043243.png)

点击【写入】后需要等待一段时间，完成后拔掉U盘即可，此时再生龙的启动盘就制作完成了。

## 备份系统到U盘(Save disk image)

**描述**：逐步说明如何将第一块磁盘（sda）保存为第二块磁盘（sdb）上的映像。

### 启动 Clonezilla Live


从U盘启动，选择Clonezilla live (default settings)，进入再生龙。

![Clonezilla live](http://iqotom.com/wp-content/uploads/2019/09/clonezilla_01-1024x768.png)

### 选择分辨率

在 Clonezilla Live 的启动菜单中选择`800x600`分辨率。

![ocs-01-bootmenu](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-01-bootmenu.png)

### 选择语言

选择语言设置。在使用语言菜单中选择`zh_CN.UTF-8Chinese(Simplified)|简体中文`。

![ocs-03-lang](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-03-lang.png)

### 选择键盘布局

选择键盘布局，`不修改键盘映射`。

![ocs-04-keymap](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-04-keymap.png)

### 开始 Clonezilla

选择`Start_Clonezilla 使用再生龙`选项。

![ocs-05-start-clonezilla](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-05-start-clonezilla.png)

### 选择“device-image”选项

进入 Clonezilla 后，选择`device-image硬盘/分区[存到/来自]镜像文件`选项。

![ocs-06-dev-img](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-06-dev-img.png)

### 选择“local_dev”选项

选择`local_dev`选项，将 sdb1 设为映像存储位置。

![ocs-07-1-img-repo](https://clonezilla.org///clonezilla-live/doc/02_Restore_disk_image/images/ocs-07-2-plug-and-play-dev-prompt.png)

注意：如果要将备份的镜像存储到再生龙工具U盘或是准备恢复的镜像存储在再生龙工具U盘中，此时再生龙会提示如果你的镜像存储盘是U盘


请将U盘连接到主板上并且等待5 秒钟后按回车键，再生龙会挂载U盘如果存储镜像的存储盘是已经连接到主板上的硬盘，可以直接按回车键

![ocs-07-3-dev-scan](https://clonezilla.org///clonezilla-live/doc/02_Restore_disk_image/images/ocs-07-3-dev-scan.png)

### 选择映像存储位置

选择 sdb1 作为映像存储位置，然后选择“savedisk”选项。
![ocs-08-1-sdb1-as-img-repo](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-08-1-sdb1-as-img-repo.png)

![ocs-08-1-sdb1-as-img-repo-fs-check](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-08-1-sdb1-as-img-repo-fs-check.png)

选择“初学者模式”

![ocs-08-4-beginner-expert-mode](https://clonezilla.org////clonezilla-live/doc/01_Save_disk_image/images/ocs-08-4-beginner-expert-mode.png)

选择备份/恢复的模式，这里菜单中有两种模式，"savedisk 存储本机硬盘为镜像文件"，此模式是将目标盘进行整盘；"saveparts 存储本机分区为镜像文件"，此模式是对目标盘的某个分区进行备份

![ocs-08-5-save-img](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-08-5-save-img.png)

### 输入映像名称和选择源磁盘

输入映像名称，并选择 sda 作为源磁盘。

![ocs-10-1-img-name](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-10-1-img-name.png)

### 保存磁盘映像

Clonezilla 正在将 sda 的磁盘映像保存到第二块磁盘 sdb1 的分区中。

![ocs-10-2-disk-selection](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-10-2-disk-selection.png)

选择压缩选项，在 Clonezilla 中保存磁盘映像时，可以选择压缩选项 -z1p（使用并行 gzip）或 -z9p（并行 zstd，速度更快且文件大小略小于 gzip）：

![ocs-10-3-param-z1p-z9p](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-10-3-param-z1p-z9p.png)

选择是否检查源文件系统

![ocs-10-4-check-source-fs](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-10-4-check-source-fs.png)

选择保存映像后的模式，操作完成后，选择重启/关机/其他

![ocs-10-7-reboot-poweroff](https://clonezilla.org//clonezilla-live/doc/01_Save_disk_image/images/ocs-10-7-reboot-poweroff.png)

按提示操作，按y继续

![clonezilla backup](http://iqotom.com/wp-content/uploads/2019/09/clonezilla_15-1024x768.png)

### 系统备份过程

系统备份过程

![clone disk](http://iqotom.com/wp-content/uploads/2019/09/clonezilla_16-1024x768.png)

备份完成，按Enter键进行关机或重启

![clone complete](http://iqotom.com/wp-content/uploads/2019/09/clonezilla_17-1024x687.png)

## 还原系统到硬盘

描述：逐步说明如何将第二块磁盘（sdb）上的映像恢复到第一块磁盘（sda）。

### 启动 Clonezilla Live

通过 Clonezilla Live 启动机器。

### 选择分辨率

在 Clonezilla Live 的启动菜单中选择 800x600 分辨率。

### 选择语言

选择语言设置。

### 选择键盘布局

选择键盘布局。

### 开始 Clonezilla

选择“开始 Clonezilla”选项。

### 选择“device-image”选项

进入 Clonezilla 后，选择“device-image”选项。

### 选择“local_dev”选项

选择“local_dev”选项，将 sdb1 设为映像存储位置。

### 选择映像存储位置

选择 sdb1 作为映像存储位置，然后选择“restoredisk”选项。

![ocs-08-1-sdb1-as-img-repo](https://clonezilla.org///clonezilla-live/doc/02_Restore_disk_image/images/ocs-08-1-sdb1-as-img-repo.png)

![ocs-08-5-restoredisk](https://clonezilla.org///clonezilla-live/doc/02_Restore_disk_image/images/ocs-08-5-restoredisk.png)

### 选择映像名称和目标磁盘

选择映像名称，并选择 sda 作为目标磁盘。

![ocs-10-1-img-name](https://clonezilla.org///clonezilla-live/doc/02_Restore_disk_image/images/ocs-10-1-img-name.png)

### 恢复磁盘映像

Clonezilla 正在将 sdb 上的磁盘映像恢复到 sda 上。


## Additional Resources

### Documentation

1. [Clonezilla live doc](https://clonezilla.org/clonezilla-live-doc.php)

### Useful Websites

1. [Clonezilla(再生龙)备份还原系统](http://iqotom.com/?p=582)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
