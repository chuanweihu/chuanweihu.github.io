---
layout: post
title:  Linux系统-CentOS/Rocky系统的备份策略
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍通过Linux系统本身自带的工具进行备份。

* TOC
{:toc}

<!--more-->

## 备份的由来

- 硬件问题（Hardware Failures）
	- 保持库存硬件
	- 服务合同
		- 服务响应时间
		- 服务响应时效
		- 部件性能状态
		- 可用预算
		- 硬件的保修期限
- 软件问题（Software Failures）
	- 操作系统
		- 崩溃（Crashes）-内核出错
		- 挂起（Hangs）-死锁/活锁
	- 应用软件
		- 软件支持
			- 文档
			- 自我解决（FAQs）
			- Web/Email支持
			- 电话支持
			- 现场服务
- 环境问题（Environmental Failures）
	- 建筑整体
	- 电（Uninterruptible Power Supplies，UPS）
	- 供暖、通风和空调系统
	- 自然天气
- 人为因素（Human Errors）
	- 终端用户
	- 运营商操作员
	- 系统管理员
	- 维修服务人员

## 备份的对象

- 操作系统
	- `/etc`: Linux 默认的重要参数
	- `/home`: 一般用户的家目录
	- `/var/spool/mail`: 邮件信息
	- `/boot`: linux核心
	- `/root`: root账户
	- `/usr/local`: 自定义软件安装位置
	- `/opt`: 自定义软件安装位置
- 网络服务
	- 软件本身的配置文件，例如：/etc/ 整个目录，/usr/local/ 整个目录
	- 软件服务提供的数据，以 WWW 及 MySQL 为例：WWW 数据：/var/www 整个目录或 /srv/www 整个目录，及系统的使用者家目录
	- MySQL ： /var/lib/mysql 整个目录
	- 其他在 Linux 主机上面提供的服务之数据库文件！
- 重要文件
	- /var(注：这个目录当中有些缓存目录则可以不备份！)

## 备份的媒介

- 选择媒介的因素
	- 备份速度
	- 备份容量
	- 备份效益
	- 备份的可靠性

- 选择备份的媒介
	- 光驱： /dev/cdrom (其实应该是 /dev/sdX 或 /dev/hdX)
	- 磁带： /dev/st0 (SCSI 介面), /dev/ht0 (IDE 介面)
	- 软盘： /dev/fd0, /dev/fd1
	- 硬盘： /dev/hd[a-d][1-63] (IDE), /dev/sd[a-p][1-16] (SCSI/SATA)
	- 外接式U盘/硬盘： /dev/sd[a-p][1-16] (与 SCSI 相同)
	- 打印机： /dev/lp[0-2]

## 备份的种类

| Backup types | Data                                            | Backup speed                                                | Storage space                                   | Restoration speed                                                                   |
|--------------|-------------------------------------------------|-------------------------------------------------------------|-------------------------------------------------|-------------------------------------------------------------------------------------|
| Active full  | Copies all data.                                | Slow.                                                       | Substantial.                                    | Fast.                                                                               |
| Incremental  | Copies only the changed data since last backup. | Faster than differential.                                   | Smaller than differential.                      | Slower than differential as it requires a full backup plus all incremental backups. |
| Differential | Copies changed data since last full backup.     | Slower than incremental but faster than active full backup. | Gets larger especially with subsequent backups. | Faster than incremental as it requires just the full and last differential.         |

### 完整备份(Full Backup)

完全备份就是指对某一个时间点上的所有数据或应用进行的一个完全拷贝。实际应用中就是用一盘磁带对整个系统进行完全备份，包括其中的系统和所有数据。这种备份方式最大的好处就是只要用一盘磁带，就可以恢复丢失的数据。因此大大加快了系统或数据的恢复时间。

完整备份就是将根目录 (/) 整个系统通通备份下来的意思！ 不过，在某些场合底下，完整备份也可以是备份一个文件系统 (filesystem)！例如 /dev/sda1 或 /dev/md0 或 /dev/myvg/mylv
之类的文件系统就是了。

### 增量备份（Incremental Backup）

所谓的增量备份，指的是在系统在进行完第一次完整备份后，经过一段时间的运行， 比较系统与备份档之间的差异，仅备份有差异的文件而已。而第二次累积备份则与第一次累积备份的数据比较， 也是仅备份有差异的数据而已。

![Incremental Backups"](https://www.ubackup.com/screenshot/en/others/incremental-backup.png)

完整备份常用的工具有 dd, cpio, dump/restore，tar等等。

- 备份磁盘

```
# 1. 用 dd 来将 /dev/sda 备份到完全一模一样的 /dev/sdb 硬盘上：
[root@www ~]# dd if=/dev/sda of=/dev/sdb
# 由於 dd 是读取磁区，所以 /dev/sdb 这颗磁碟可以不必格式化！非常的方便！
# 只是你会等非常非常久！因为 dd 的速度比较慢！

# 2. 使用 cpio 来备份与还原整个系统，假设储存媒体为 SATA 磁带机：
[root@www ~]# find / -print | cpio -covB > /dev/st0  <==备份到
```

- 备份文件

```
# 1. 完整备份
[root@www ~]# dump -0u -f /backupdata/home.dump /home

# 2. 第一次进行累积备份
[root@www ~]# dump -1u -f /backupdata/home.dump.1 /home

# tar完整备份
[root@www ~]# tar --exclude /proc --exclude /mnt --exclude /tmp \
> --exclude /backupdata -jcvp -f /backupdata/system.tar.bz2 /
```

### 差异备份（Differential Backup）

差异备份与累积备份有点类似，也是需要进行第一次的完整备份后才能够进行。只是差异备份指的是：每次的备份都是与原始的完整备份比较的结果。所以系统运行的越久，离完整备份时间越长， 那么该次的差异备份数据可能就会越大！

![Differential Backups"](https://www.ubackup.com/screenshot/en/others/differential-backup.png)

差异备份常用的工具与累积备份差不多！

```
# 1. 完整备份
[root@www ~]# tar -N '2009-06-01' -jpcv -f /backupdata/home.tar.bz2 /home
# 只有在比 2009-06-01 还要新的文件，在 /home 底下的文件才会被打包进 home.bz2 中！
# 有点奇怪的是，目录还是会被记录下来，只是目录内的旧文件就不会备份。

# 2. rsync 来进行镜像备份
[root@www ~]# rsync -av 来源目录 目标目录
# 将 /home/ 镜像到 /backupdata/home/ 去
[root@www ~]# rsync -av /home /backupdata/
# 此时会在 /backupdata 底下产生 home 这个目录来！
[root@www ~]# rsync -av /home /backupdata/
# 再次进行会快很多！如果数据没有更动，几乎不会进行任何动作！
```

差异增量备份所使用的磁碟容量可能会比累积增量备份备份来的大，但是差异备份的还原较快， 因为只需要还原完整备份与最近一次的差异备份即可。

### 关键数据备份

如果仅是备份关键数据而已，那么由於系统的绝大部分运行档都可以后来重新安装，因此若你的系统不是因为硬件问题， 而是因为软件问题而导致系统被攻破或损毁时，直接捉取最新的 Linux distribution ，然后重新安装，
然后再将系统数据 (如帐号/口令与家目录等等) 与服务数据 (如 www/email/crontab/ftp 等等) 一个一个的填回去！

```
# tar备份
[root@www ~]# tar -jpcvf mysql.`date +%Y-%m-%d`.tar.bz2 /var/lib/mysql
```

## 备份的策略

### 每日备份（自动）

目前仅备份 MySQL 数据库；利用 /etc/crontab 来自动提供备份的进行

```
[root@www ~]# vi /backup/backupday.sh
#!/bin/bash
# =========================================================
# 请输入，你想让备份数据放置到那个独立的目录去
basedir=/backup/daily/  <==你只要改这里就可以了！

# =========================================================
PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH
export LANG=C
basefile1=$basedir/mysql.$(date +%Y-%m-%d).tar.bz2
basefile2=$basedir/cgi-bin.$(date +%Y-%m-%d).tar.bz2
[ ! -d "$basedir" ] && mkdir $basedir

# 1. MysQL (数据库目录在 /var/lib/mysql)
cd /var/lib
  tar -jpc -f $basefile1 mysql

# 2. WWW 的 CGI 程序 (如果有使用 CGI 程序的话)
cd /var/www
  tar -jpc -f $basefile2 cgi-bin

[root@www ~]# chmod 700 /backup/backupday.sh
[root@www ~]# /backup/backupday.sh  <==记得自己试跑看看！

[root@www ~]# vi /etc/crontab
# 每个星期日的 3:30 进行重要文件的备份
30 3 * * 0 root /backup/backupwk.sh
# 每天的 2:30 进行 MySQL 的备份
30 2 * * * root /backup/backupday.sh
```

### 每周备份（自动）

每周备份包括 /home, /var, /etc, /boot, /usr/local 等目录与特殊服务的目录，利用 /etc/crontab 来自动提供备份的进行

```
[root@www ~]# vi /backup/backupwk.sh
#!/bin/bash
# ====================================================================
# 使用者参数输入位置：
# basedir=你用来储存此脚本所预计备份的数据之目录(请独立文件系统)
basedir=/backup/weekly  <==您只要改这里就好了！

# ====================================================================
# 底下请不要修改了！用默认值即可！
PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH
export LANG=C

# 配置要备份的服务的配置档，以及备份的目录
named=$basedir/named
postfixd=$basedir/postfix
vsftpd=$basedir/vsftp
sshd=$basedir/ssh
sambad=$basedir/samba
wwwd=$basedir/www
others=$basedir/others
userinfod=$basedir/userinfo
# 判断目录是否存在，若不存在则予以创建。
for dirs in $named $postfixd $vsftpd $sshd $sambad $wwwd $others $userinfod
do
	[ ! -d "$dirs" ] && mkdir -p $dirs
done

# 1. 将系统主要的服务之配置档分别备份下来，同时也备份 /etc 全部。
cp -a /var/named/chroot/{etc,var}	$named
cp -a /etc/postfix /etc/dovecot.conf	$postfixd
cp -a /etc/vsftpd/*			$vsftpd
cp -a /etc/ssh/*			$sshd
cp -a /etc/samba/*			$sambad
cp -a /etc/{my.cnf,php.ini,httpd}	$wwwd
cd /var/lib
  tar -jpc -f $wwwd/mysql.tar.bz2 	mysql
cd /var/www
  tar -jpc -f $wwwd/html.tar.bz2 	html cgi-bin
cd /
  tar -jpc -f $others/etc.tar.bz2	etc
cd /usr/
  tar -jpc -f $others/local.tar.bz2	local

# 2. 关於使用者参数方面
cp -a /etc/{passwd,shadow,group}	$userinfod
cd /var/spool
  tar -jpc -f $userinfod/mail.tar.bz2	mail
cd /
  tar -jpc -f $userinfod/home.tar.bz2	home
cd /var/spool
  tar -jpc -f $userinfod/cron.tar.bz2	cron at

[root@www ~]# chmod 700 /backup/backupwk.sh
[root@www ~]# /backup/backupwk.sh

[root@www ~]# vi /etc/crontab
# 每个星期日的 3:30 进行重要文件的备份
30 3 * * 0 root /backup/backupwk.sh
```

### 每月备份（手动）

/backup/ 当中的数据 copy 出来才行耶！否则整部系统死掉的时候...那可不是闹著玩的！ 大约一个月到两个月之间，会将 /backup 目录内的数据使用 DVD 复制一下，然后将 DVD 放置在家中保存。

### 远程备份（自动）

主机已经提供了 FTP 与 sshd 这两个网络服务， 同时你已经做好了 FTP 的帐号，sshd 帐号的免口令登陆功能等

#### 使用 FTP 上传备份数据

假设你要上传的数据是将 /backup/weekly/ 目录内的文件统整为一个 /backup/weekly.tar.bz2 ， 并且上传到服务器端的 /home/backup/ 底下，使用的帐号是 dmtsai ，口令是
dmtsai.pass 。

```
[root@www ~]# vi /backup/ftp.sh
#!/bin/bash
# ===========================================
# 先输入系统所需要的数据
host="192.168.1.100"		# 远程主机
id="dmtsai"			# 远程主机的 FTP 帐号
pw='dmtsai.pass'		# 该帐号的口令
basedir="/backup/weekly"	# 本地端的欲被备份的目录
remotedir="/home/backup"	# 备份到远程的何处？

# ===========================================
backupfile=weekly.tar.bz2
cd $basedir/..
  tar -jpc -f $backupfile $(basename $basedir)

ftp -n "$host" > ${basedir}/../ftp.log 2>&1 <<EOF
user $id $pw
binary
cd $remotedir
put $backupfile
bye
EOF
```

#### 使用 rsync 上传备份数据

假设你已经配置好 dmtsai 这个帐号可以不用口令即可登陆远程服务器，而同样的你要让 /backup/weekly/ 整个备份到 /home/backup/weekly 底下时

```
[root@www ~]# vi /backup/rsync.sh
#!/bin/bash
remotedir=/home/backup/
basedir=/backup/weekly
host=127.0.0.1
id=dmtsai

# 底下为程序阶段！不需要修改喔！
rsync -av -e ssh $basedir ${id}@${host}:${remotedir}
```

## dump filesystem

```
[root@localhost ~]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    4.0M     0  4.0M   0% /dev
tmpfs                        16G     0   16G   0% /dev/shm
tmpfs                       6.2G   34M  6.2G   1% /run
tmpfs                       4.0M     0  4.0M   0% /sys/fs/cgroup
/dev/mapper/openeuler-root  590G   61G  499G  11% /
tmpfs                        16G     0   16G   0% /tmp
/dev/sda5                    20G  245M   19G   2% /boot
/dev/sda4                    20G  6.3M   20G   1% /boot/efi
/dev/mapper/openeuler-home   98G  884M   92G   1% /home
/dev/mapper/openeuler-var   9.8G  1.2G  8.1G  13% /var
tmpfs                       3.1G   60K  3.1G   1% /run/user/1001

[root@localhost ~]# dump -S /dev/mapper/openeuler-home 
930177024

[root@localhost ~]# dump -0u -f /backupdata/home.dump /dev/mapper/openeuler-home 
  DUMP: Date of this level 0 dump: Mon Jan  8 09:10:42 2024
  DUMP: Dumping /dev/mapper/openeuler-home (/home) to /backupdata/home.dump
  DUMP: Label: none
  DUMP: Writing 10 Kilobyte records
  DUMP: mapping (Pass I) [regular files]
  DUMP: mapping (Pass II) [directories]
  DUMP: estimated 908376 blocks.
  DUMP: Volume 1 started with block 1 at: Mon Jan  8 09:10:43 2024
  DUMP: dumping (Pass III) [directories]
  DUMP: dumping (Pass IV) [regular files]
  DUMP: Closing /backupdata/home.dump
  DUMP: Volume 1 completed at: Mon Jan  8 09:10:51 2024
  DUMP: Volume 1 908370 blocks (887.08MB)
  DUMP: Volume 1 took 0:00:08
  DUMP: Volume 1 transfer rate: 113546 kB/s
  DUMP: 908370 blocks (887.08MB) on 1 volume(s)
  DUMP: finished in 8 seconds, throughput 113546 kBytes/sec
  DUMP: Date of this level 0 dump: Mon Jan  8 09:10:42 2024
  DUMP: Date this dump completed:  Mon Jan  8 09:10:51 2024
  DUMP: Average transfer rate: 113546 kB/s
  DUMP: DUMP IS DONE
  
[root@localhost ~]# ls -lh /backupdata/home.dump /etc/dumpdates 
-rw-r--r--. 1 root root 888M Jan  8 09:10 /backupdata/home.dump
-rw-rw-r--. 1 root disk   60 Jan  8 09:10 /etc/dumpdates

[root@localhost ~]# cat /etc/dumpdates 
/dev/mapper/openeuler-home 0 Mon Jan  8 09:10:42 2024 +0800

[root@localhost ~]# dd if=/dev/zero of=/home/testing.img bs=1M count=10
10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.00341576 s, 3.1 GB/s

[root@localhost ~]# dump -1u -f /backupdata/home.dump.1 /dev/mapper/openeuler-home 
  DUMP: Date of this level 1 dump: Mon Jan  8 09:15:45 2024
  DUMP: Date of last level 0 dump: Mon Jan  8 09:10:42 2024
  DUMP: Dumping /dev/mapper/openeuler-home (/home) to /backupdata/home.dump.1
  DUMP: Label: none
  DUMP: Writing 10 Kilobyte records
  DUMP: mapping (Pass I) [regular files]
  DUMP: mapping (Pass II) [directories]
  DUMP: estimated 11879 blocks.
  DUMP: Volume 1 started with block 1 at: Mon Jan  8 09:15:45 2024
  DUMP: dumping (Pass III) [directories]
  DUMP: dumping (Pass IV) [regular files]
  DUMP: Closing /backupdata/home.dump.1
  DUMP: Volume 1 completed at: Mon Jan  8 09:15:45 2024
  DUMP: Volume 1 11880 blocks (11.60MB)
  DUMP: 11880 blocks (11.60MB) on 1 volume(s)
  DUMP: finished in less than a second
  DUMP: Date of this level 1 dump: Mon Jan  8 09:15:45 2024
  DUMP: Date this dump completed:  Mon Jan  8 09:15:45 2024
  DUMP: Average transfer rate: 0 kB/s
  DUMP: DUMP IS DONE
  
[root@localhost ~]# ls -hl /backupdata/home.dump*
-rw-r--r--. 1 root root 888M Jan  8 09:10 /backupdata/home.dump
-rw-r--r--. 1 root root  12M Jan  8 09:15 /backupdata/home.dump.1

[root@localhost ~]# dump -W
Last dump(s) done (Dump '>' file systems):
> /dev/mapper/openeuler-root	(     /) Last dump: never
> /dev/sda5	( /boot) Last dump: never
  /dev/mapper/openeuler-home	( /home) Last dump: Level 1, Date Mon Jan  8 09:15:45 2024
> /dev/mapper/openeuler-var	(  /var) Last dump: never
```

### dump one file

```
[root@localhost ~]# dump -0j -f /backupdata/etc.dump.bz2 /etc/
  DUMP: Date of this level 0 dump: Mon Jan  8 09:18:39 2024
  DUMP: Dumping /dev/mapper/openeuler-root (/ (dir etc)) to /backupdata/etc.dump.bz2
  DUMP: Label: none
  DUMP: Writing 10 Kilobyte records
  DUMP: Compressing output at transformation level 2 (bzlib)
  DUMP: mapping (Pass I) [regular files]
  DUMP: mapping (Pass II) [directories]
  DUMP: estimated 87780 blocks.
  DUMP: Volume 1 started with block 1 at: Mon Jan  8 09:18:39 2024
  DUMP: dumping (Pass III) [directories]
  DUMP: dumping (Pass IV) [regular files]
  DUMP: Closing /backupdata/etc.dump.bz2
  DUMP: Volume 1 completed at: Mon Jan  8 09:18:43 2024
  DUMP: Volume 1 took 0:00:04
  DUMP: Volume 1 transfer rate: 6004 kB/s
  DUMP: Volume 1 105300kB uncompressed, 24018kB compressed, 4.385:1
  DUMP: 105300 blocks (102.83MB) on 1 volume(s)
  DUMP: finished in 3 seconds, throughput 35100 kBytes/sec
  DUMP: Date of this level 0 dump: Mon Jan  8 09:18:39 2024
  DUMP: Date this dump completed:  Mon Jan  8 09:18:43 2024
  DUMP: Average transfer rate: 6004 kB/s
  DUMP: Wrote 105300kB uncompressed, 24018kB compressed, 4.385:1
  DUMP: DUMP IS DONE
```

### restore system

```
[root@localhost ~]# restore -t -f /backupdata/home.dump
Dump   date: Mon Jan  8 09:10:42 2024
Dumped from: the epoch
Level 0 dump of /home on localhost.localdomain:/dev/mapper/openeuler-home
Label: none
         2	.
        11	./lib.tar
        12	./messages

[root@localhost home]# cd /home/
[root@localhost home]# ls -alh
total 894M
drwxr-xr-x.  2 root root 4.0K Jan  8 09:22 .
dr-xr-xr-x. 22 root root 4.0K Jan  8 09:00 ..
-rw-r--r--.  1 root root 882M Jan  8 09:07 lib.tar
-rw-------.  1 root root 1.9M Jan  8 09:08 messages-back
-rw-r--r--.  1 root root  10M Jan  8 09:14 testing.img
[root@localhost home]# mv messages messages-back
[root@localhost home]# restore -C -f /backupdata/home.dump
Dump   date: Mon Jan  8 09:10:42 2024
Dumped from: the epoch
Level 0 dump of /home on localhost.localdomain:/dev/mapper/openeuler-home
Label: none
filesys = /home
restore: unable to stat ./messages: No such file or directory
Some files were modified!  1 compare errors

[root@localhost mnt]# fdisk /dev/mapper/openeuler-home 

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.

The device contains 'ext4' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xa26f516c.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-209715199, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199, default 209715199): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): p
Disk /dev/mapper/openeuler-home: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xa26f516c

Device                           Boot Start     End Sectors Size Id Type
/dev/mapper/openeuler-home-part1       2048 4196351 4194304   2G 83 Linux

Command (m for help): w
The partition table has been altered.
Failed to add partition 1 to system: Invalid argument

The kernel still uses the old partitions. The new table will be used at the next reboot. 
Syncing disks.

[root@localhost mnt]# partprobe 

[root@localhost mnt]# mkfs -t ext4 /dev/mapper/openeuler-home 
mke2fs 1.46.4 (18-Aug-2021)
/dev/mapper/openeuler-home is mounted; will not make a filesystem here!

[root@localhost mnt]# mount /dev/mapper/openeuler-home /mnt
mount: /mnt: /dev/mapper/openeuler-home already mounted on /home.

[root@localhost home]# restore -r -f /backupdata/home.dump
```

- 交互模式

```
[root@localhost home]# restore -i -f /backupdata/home.dump
restore > help
Available commands are:
	ls [arg] - list directory
	cd arg - change directory
	pwd - print current directory
	add [arg] - add `arg' to list of files to be extracted
	delete [arg] - delete `arg' from list of files to be extracted
	extract - extract requested files
	setmodes - set modes of requested directories
	quit - immediately exit program
	what - list dump header information
	verbose - toggle verbose flag (useful with ``ls'')
	prompt - toggle the prompt display
	help or `?' - print this list
If no `arg' is supplied, the current directory is used
restore > ls
.:
lib.tar  messages

restore > pwd
/
restore > add messages  
restore > add lib.tar  
restore > delete lib.tar  
restore > ls 
.:
 lib.tar  *messages

restore > extract
You have not read any volumes yet.
Unless you know which volume your file(s) are on you should start
with the last volume and work towards the first.
Specify next volume # (none if no more volumes): 1
set owner/mode for '.'? [yn] n
restore > quit
```

### restore file

```
[root@localhost home]# restore -t -f /backupdata/etc.dump.bz2 
Dump tape is compressed.
Dump   date: Mon Jan  8 09:18:39 2024
Dumped from: the epoch
Level 0 dump of / (dir etc) on localhost.localdomain:/dev/mapper/openeuler-root
Label: none
         2	.
  23330817	./etc
  23330820	./etc/crypttab
  23330821	./etc/dnf
  23330822	./etc/dnf/modules.d
  23330855	./etc/dnf/aliases.d
  23330857	./etc/dnf/modules.defaults.d
  23330858	./etc/dnf/plugins
  23335160	./etc/dnf/plugins/copr.d
  23335159	./etc/dnf/plugins/copr.conf
  23335161	./etc/dnf/plugins/debuginfo-install.conf
  23330859	./etc/dnf/protected.d
  23332375	./etc/dnf/protected.d/systemd.conf
  23330860	./etc/dnf/protected.d/dnf.conf
  23335523	./etc/dnf/protected.d/grub2-efi-x64.conf
  23335280	./etc/dnf/protected.d/yum.conf
  23332202	./etc/dnf/protected.d/grub2-tools-minimal.conf
  23334208	./etc/dnf/protected.d/sudo.conf
  23336194	./etc/dnf/protected.d/grub2-pc.conf
```

### 打包关键数据

```
[root@localhost docs]# tar -jpcvf cuda.`date +%Y-%m-%d`.tar.bz2 /public/docs/cuda
tar: Removing leading `/' from member names
/public/docs/cuda/
/public/docs/cuda/CUDA_C_Programming_Guide.pdf
/public/docs/cuda/CUDA_C_Best_Practices_Guide.pdf
```

## Additional Resources

### Documentation

1. [Linux 备份策略](http://cn.linux.vbird.org/linux_basic/0580backup.php#VBird_strategy)
2. [RMAN Backup Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-backup-concepts.html)
3. [Chapter 8. Planning for Disaster](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/4/html/introduction_to_system_administration/ch-disaster)
4. [s1-disaster-backups](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/4/html-single/introduction_to_system_administration/index#s1-disaster-backups)
5. [第九章、文件与文件系统的压缩与打包](http://cn.linux.vbird.org/linux_basic/0240tarcompress.php#dump_restore)
6. [第二十五章、 Linux 备份策略](http://cn.linux.vbird.org/linux_basic/0580backup.php)

### Useful Websites

1. [Full vs. incremental vs. differential backup](https://aws.amazon.com/compare/the-difference-between-incremental-differential-and-other-backups/)
2. [Incremental vs Differential Backup: Which Is Better?](https://www.ubackup.com/articles/incremental-vs-differential-backup.html)
3. [浪潮英信服务器标准保修服务政策（B版）V1.2](https://www.inspur.com/lcjtww/2317452/2367100/2367109/2519500/index.html)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
