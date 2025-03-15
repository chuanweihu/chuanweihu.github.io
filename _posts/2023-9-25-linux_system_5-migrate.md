---
layout: post
title:  Linux系统-CentOS 7 迁移到Rocky Linux 8
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍CentOS 7 迁移到Rocky Linux 8。需要CentOS 7先升级到CentOS 8.5，然后再更新到Rocky 8.5即可。

* TOC
{:toc}

<!--more-->

## 更新管理软件包

### 更新yum

`yum update -y`

### 安装epel仓库

`yum install epel-release -y`

### 安装解决包冲突的工具

`yum install rpmconf -y`

### 查看rpm配置

`rpmconf -a`

### 安装yum工具

`yum install yum-utils -y`

### CentOS软件包清理

```shell
#清理所有无用软件包
package-cleanup --leaves
#清理RPM软件包
package-cleanup --orphans
```

### 安装dnf

```shell
#当前系统中安装dnf软件包管理工具（centos 8默认使用）
yum install dnf -y
```

### 卸载yum

```shell
#卸载centos 7中yum软件管理工具（使用dnf卸载 centos 7默认软件管理工具yum）
dnf remove yum yum-metadata-parser -y
#删除yum相关配置文件
rm -Rf /etc/yum
```

### 更新dnf

```shell
dnf update -y
```

## 系统升级（CentOS 7升级到CentOS 8）

### 安装CentOS 8的release软件包

```shell
# 安装CentOS 8的release软件包
dnf install http://vault.centos.org/8.5.2111/BaseOS/x86_64/os/Packages/{centos-linux-repos-8-3.el8.noarch.rpm,centos-linux-release-8.5-1.2111.el8.noarch.rpm,centos-gpg-keys-8-3.el8.noarch.rpm} -y
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```

### 升级EPEL仓库

```shell
# 升级EPEL仓库
dnf upgrade https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y
# 在升级了EPEL仓库后，移除所有临时文件
dnf clean all
```

### 删除CentOS 7的旧内核core

```shell
# 删除CentOS 7的旧内核core
rpm -e `rpm -q kernel`
# 移除冲突的软件包
rpm -e --nodeps sysvinit-tools
```

### 升级CentOS 8

```shell
# 然后执行CentOS 8升级
dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
# 安装内核
dnf install kernel-core -y
```

## 迁移到Rocky Linux 8

根据Rocky Linux的步骤继续迁移： [Migrate to Rocky Linux 8 ](https://docs.rockylinux.org/guides/migrate2rocky/)

### 下载移植程序

```shell
# 下载移植程序
curl https://raw.githubusercontent.com/rocky-linux/rocky-tools/main/migrate2rocky/migrate2rocky.sh -o migrate2rocky.sh
```

### 运行移植程序

```shell
# 添加执行权限
chmod u+x migrate2rocky.sh
# 运行程序
./migrate2rocky.sh -r
```

## 问题

1. kernel >= 3.10.0-384.el7 被 (已安裝) hypervvssd-0-0.34.20180415git.el7.x86_64 需要

用`dnf remove hypervvssd-0-0.34.20180415git.el7.x86_64`或者`rpm -e --nodeps hypervvssd-0-0.34.20180415git.el7.x86_64`命令删除相关包即可

2. UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 4: ordinal not in range(128)

中文编码问题，修改相关的打印代码或者不管即可。

## 参考

## Additional Resources

### Documentation

1. [How to migrate to Rocky Linux from CentOS Stream, CentOS, AlmaLinux, RHEL, or Oracle Linux](https://docs.rockylinux.org/guides/migrate2rocky/)

### Useful Websites

1. [CentOS 7 迁移到Rocky Linux 9](https://cloud.tencent.com/developer/article/2442737)
2. [CentOS EOL应对方案](https://help.aliyun.com/zh/ecs/user-guide/options-for-dealing-with-centos-linux-end-of-life#4276c562c00mi)
3. [【Rocky 9】Step by Step 从 CentOS 7.9 升级到 Rocky Linux 9.2](https://www.modb.pro/db/1717086120041865216)
4. [升级CentOS 7到CentOS 8](https://cloud-atlas.readthedocs.io/zh-cn/latest/linux/redhat_linux/centos/upgrade_centos_7_to_8.html)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
