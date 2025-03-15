---
layout: post
title:  Linux系统-CentOS/Rocky系统的环境管理module
categories: [study]
comments: true
tags: [Linux]
---

> 本文主要介绍通过module加载和管理已有软件环境。

* TOC
{:toc}

<!--more-->

## 概要

Environment Modules软件包提供了通过modulefile动态修改用户环境的功能，通常，用户在登录时通过设置会话期间将引用的每个应用程序的环境信息来初始化其环境。Environment Modules软件包是一种简化外壳初始化的工具，使用户可以在会话期间使用modulefiles轻松修改其环境。

每个模块文件都包含为应用程序配置外壳所需的信息。初始化Modules包后，可以使用解释模块文件的module命令在每个模块的基础上修改环境。通常，模块文件指示模块命令更改或设置外壳程序环境变量，例如PATH，MANPATH等。模块文件可以由系统上的许多用户共享，并且用户可能拥有自己的集合来补充或替换共享的模块
文件。

可以以一种干净的方式动态地和原子地加载和卸载模块。所有流行的shell都支持，包括bash, ksh, zsh, sh, csh, tcsh, fish,以及一些脚本语言，如Perl中，ruby, tcl, python, cmake 和 R。

模块在管理不同版本的应用程序时很有用。模块也可以捆绑到元模块中，这些元模块将加载一整套不同的应用程序。


## 名词解释

- `module`: module is a user interface to the Modules package. The Modules package provides for the dynamic modification of the user's environment via modulefiles.

- `modulefile`: modulefiles are written in the Tool Command Language, Tcl(n) and are interpreted by the modulecmd.tcl program via the module user interface. modulefiles can be loaded, unloaded, or switched on-the-fly while the user is working; and can be used to implement site policies regarding the access and use of applications.

## 安装

```shell
# 安装后退出，重新打开终端生效
[user@c1 ~]$ yum install -y environment-modules
```

## 使用流程

### 1. 编写module环境变量配置文件

- 编写modulefile文件`intel-2018`

```shell
# 创建/public/modulefiles文件夹
[user@c1 ~]$ mkdir /public/modulefiles

# 切换到/public/modulefiles文件夹
[user@c1 ~]$ cd /public/modulefiles

# 编写modulefile文件intel-2018
[user@c1 modulefiles]$ vi intel-2018
#%Module1.#####################################################################
##
## modules modulefile
##
## modulefiles/modules.  Generated from modules.in by configure.
##
# 显示 module help 主要内容
proc ModulesHelp { } {
        puts stderr "This module sets up access to intel-2018 tool environment" 
}

# 设置module whatis 显示主要内容
module-whatis "sets up access to intel-2018 tool environment"

# module不加载冲突模块cuda
conflict        intel

# 设置modulefile内部变量prefix
set     prefix          /public/app/intel-2018

# 通过prepend-path增加PATH变量
prepend-path    PATH		${prefix}/compilers_and_libraries/linux/intel64/bin
prepend-path    PATH		${prefix}/compilers_and_libraries_2018.1.163/linux/mpi/intel64/bin

# 通过append-path 增加PATH变量
append-path    PATH		${prefix}/parallel_studio_xe_2018.1.038/bin

# 通过prepend-path 增加LD_LIBRARY_PATH变量
prepend-path    LD_LIBRARY_PATH		${prefix}/compilers_and_libraries_2018.1.163/linux/mkl/lib/intel64

# 设置环境变量MKL_ROOT变量
setenv		MKL_ROOT		${prefix}/compilers_and_libraries_2018.1.163/linux/mkl

```

- 编写modulefile文件`cuda-10.0`

```shell
# 切换到/public/modulefiles文件夹
[user@c1 ~]$ cd /public/modulefiles

[user@c1 modulefiles]$ vi cuda-10.0
#%Module1.#####################################################################
##
## modules modulefile
##
## modulefiles/modules.  Generated from modules.in by configure.
##
# 显示 module help 主要内容
proc ModulesHelp { } {
        puts stderr "This module sets up access to cuda-10.0 tool environment" 
}

# 设置module whatis 显示主要内容
module-whatis "sets up access to cuda-10.0 tool environment"

# 设置依赖模块（需要先加载该模块）
prereq          intel-2018

# module不加载冲突模块cuda
conflict        cuda

# 设置modulefile内部变量prefix
set     prefix          /usr/local/cuda-10.0

prepend-path    PATH           		${prefix}/bin
prepend-path    LD_LIBRARY_PATH		${prefix}/lib64
```

### 2. 修改系统默认搜索路径

```shell
# 方法一：设置 MODULEPATH
# 添加/public/modulefiles文件夹到module模块，重启终端后生效
[user@c1 ~]$ sudo echo "/public/modulefiles" >> /usr/share/Modules/init/.modulespath

# 方法二：module 命令
# 添加/public/modulefiles文件夹到profile
[user@c1 ~]$ sudo echo "module use --append /public/modulefiles" >> /etc/profile
```

### 3. 加载自定义模块

```shell
# 查看当前可用模块
[user@c1 ~]$ module avail
------------------------------------------- /public/modulefiles-------------------------------------------
intel-2018  cuda-10.0

------------------------------------------- /usr/share/Modules/modulefiles -------------------------------------------
dot         module-git  module-info modules     null        use.own

# 加载cuda-10.0模块
[user@c1 ~]$ module add intel-2018

# 查看当前加载的模块
[user@c1 ~]$ module list
Currently Loaded Modulefiles:
  1) intel-2018/

# 查看环境变量（动态链接器路径）
[user@c1 ~]$ echo $LD_LIBRARY_PATH
/public/app/intel-2018/compilers_and_libraries_2018.1.163/linux/mkl/lib/intel64

# 查看环境变量（可执行文件路径）
[user@c1 ~]$ echo $PATH
/public/app/intel-2018/compilers_and_libraries_2018.1.163/linux/intel64/bin:/public/app/intel-2018/compilers_and_libraries_2018.1.163/linux/mpi/intel64/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games:/public/app/intel-2018/parallel_studio_xe_2018.1.038/bin

# 查看环境变量（自定义文件路径）
[user@c1 ~]$ echo $MKL_ROOT
/public/app/intel-2018/compilers_and_libraries_2018.1.163/linux/mkl

# 查看模块帮助内容
[user@c1 ~]$ module help intel-2018
-----------Module Specific Help for 'intel-2018'-------------------

This module sets up access to intel-2018 tool environment

# 查看模块主要内容
[user@c1 ~]$ module whatis intel-2018
intel-2018  : sets up access to intel-2018 tool environment
```

### 4. Python中使用

```python
import os

exec(open("/usr/share/Modules/init/python.py").read(), globals())
module("load", "modulefile", "modulefile", "...")
```

## 软件教程

### 命令使用

| 命令 | 作用 |
| --- | --- |
| module avail 或 module av | 查看系统中可用的软件 |
| module add 或 module load | 加载模块 |
| module rm 或 unload | 卸载模块 |
| module list 或 module li | 显示已加载模块 |
| module purge | 卸载所有模块 |
| module show | 显示模块配置文件 |
| module swap 或 module switch | 将模块1 替换为 模块2 |
| module whatis | 查看软件的主要内容 |
| module help | 查看具体软件的信息 |

```shell
# 列出所有可用模块文件
[user@c1 ~]$ module av
------------------------------------------- /public/modulefiles-------------------------------------------
cuda-10.0  intel-2018
------------------------------------------- /usr/share/Modules/modulefiles -------------------------------------------
dot         module-git  module-info modules     null        use.own

# 加载cuda-10.0模块
[user@c1 ~]$ module add intel-2018

# 显示已加载模块
[user@c1 ~]$ module list
Currently Loaded Modulefiles:
  1) intel-2018/

# 卸载intel-2018模块
[user@c1 ~]$ module rm intel-2018

# 卸载所有模块
[user@c1 ~]$ module purge

# 显示模块详细信息
[user@c1 ~]$ module show intel-2018
-------------------------------------------------------------------
/public/modulefile/intel-2018:

module-whatis                   sets up access to intel-2018 tool environment
conflict                        intel
set  prefix                     /public/app/intel-2018
prepend-path    PATH		    ${prefix}/compilers_and_libraries/linux/intel64/bin
prepend-path    PATH		    ${prefix}/compilers_and_libraries_2018.1.163/linux/mpi/intel64/bin
append-path    PATH		        ${prefix}/parallel_studio_xe_2018.1.038/bin
prepend-path   LD_LIBRARY_PATH	${prefix}/compilers_and_libraries_2018.1.163/linux/mkl/lib/intel64
setenv		MKL_ROOT		    ${prefix}/compilers_and_libraries_2018.1.163/linux/mkl
-------------------------------------------------------------------

# 替换模块
[user@c1 ~]$ module swap intel-2018 intel-2023
[user@c1 ~]$ module list
Currently Loaded Modulefiles:
  1) intel-2023/

# 查看模块主要内容
[user@c1 ~]$ module whatis intel-2023
intel-2023  : sets up access to intel-2023 tool environment

# 显示模块
[user@c1 ~]$ module help

  Modules Release 3.2.10 2012-12-21 (Copyright GNU GPL v2 1991):

  Usage: module [ switches ] [ subcommand ] [subcommand-args ]

Switches:
	-H|--help		this usage info
	-V|--version		modules version & configuration options
	-f|--force		force active dependency resolution
	-t|--terse		terse    format avail and list format
	-l|--long		long     format avail and list format
	-h|--human		readable format avail and list format
	-v|--verbose		enable  verbose messages
	-s|--silent		disable verbose messages
	-c|--create		create caches for avail and apropos
	-i|--icase		case insensitive
	-u|--userlvl <lvl>	set user level to (nov[ice],exp[ert],adv[anced])
  Available SubCommands and Args:
	+ add|load		modulefile [modulefile ...]
	+ rm|unload		modulefile [modulefile ...]
	+ switch|swap		[modulefile1] modulefile2
	+ display|show		modulefile [modulefile ...]
	+ avail			[modulefile [modulefile ...]]
	+ use [-a|--append]	dir [dir ...]
	+ unuse			dir [dir ...]
	+ update
	+ refresh
	+ purge
	+ list
	+ clear
	+ help			[modulefile [modulefile ...]]
	+ whatis		[modulefile [modulefile ...]]
	+ apropos|keyword	string
	+ initadd		modulefile [modulefile ...]
	+ initprepend		modulefile [modulefile ...]
	+ initrm		modulefile [modulefile ...]
	+ initswitch		modulefile1 modulefile2
	+ initlist
	+ initclear
```

### 脚本教程

| 参数 | 说明 | 备注 |
| --- | --- | --- |
| proc ModulesHelp | 执行`module show 模块名`时候的反馈信息 | |
| set version | 设置版本 | |
| set prefix | 设置安装目录 | 非常关键 |
| setenv CC | 设置环境变量 |  |
| conflict | 设置`conflict`报错的 | 这就是之前遇到`error`的原因，同下 |
| prereq | 用来设置依赖哪些模块的 | |
| prepend-path | 添加一个路径到某个环境变量 | |
| PATH | 将目录添加到`PATH`环境变量中 | 最常用的环境变量就是`PATH`和`LD_LIBRARY_PATH` |
| LD_LIBRARY_PATH | 将目录添加到`LD_LIBRARY_PATH`环境变量中 | |
| LIBRARY_PATH 等 | 与上面类似，通常运行软件命令时不需要，编译其他软件时候可能会用到 |

### MODULEPATH环境变量

通过 /etc/profile.d/modules.sh 调用 /usr/share/Modules/init/bash，加载 /usr/share/Modules/init/.modulespath 配置到环境变量 MODULEPATH 中。

```shell
# CentOS7-添加/public/Modules/modulefiles到MODULEPATH环境变量中
[user@c1 ~]$ echo "/public/Modules/modulefiles" >> /usr/share/Modules/init/.modulespath
# Rocky8-添加/public/Modules/modulefiles到MODULEPATH环境变量中
[user@c1 ~]$ echo "/public/Modules/modulefiles" >> /etc/environment-modules/modulespath
```

## 问题集合

### conflict

对于conflict而言，通常是由于同款软件不能同时加载多个版本，因此建议卸载之前的，再加载新的。提示哪个模块冲突了就卸载哪一个。

```shell
[user@c1 ~]$ module add intel-2023
intel-2023(25):ERROR:150: Module 'intel-2023' 
    conflicts with the currently loaded module(s) 'intel-2018'
intel-2023(25):ERROR:102: Tcl command execution failed: 
    conflict   intel
```

我们要加载的是`intel-2023`，但与`intel-2018`冲突了，那么：

```shell
$ module rm intel-2018
$ module add intel-2023
```

### prereq

对于prereq而言，通常是加载的模块依赖其他默认，因此必须先加载依赖的模块，在加载该模块。

举例：
```shell
$ module add cuda-10.0
module add cuda-10.0(27):ERROR:151: Module 'module add cuda-10.0' 
    depends on one of the module(s) 'intel-2018'
module add cuda-10.0(27):ERROR:102: Tcl command execution failed: 
    prereq                intel-2018
```

我们需要先加载intel-2018，再加载module add cuda-10.0。

```shell
$ module add intel-2018
$ module add cuda-10.0
```

## Additional Resources

### Documentation

1. [Environment Modules open source project](https://modules.sourceforge.net/)
2. [Environment Modules documentation portal](https://modules.readthedocs.io/en/latest/)
3. [modulefile](https://modules.readthedocs.io/en/v4.3.1/modulefile.html)
4. [module](https://modules.readthedocs.io/en/v5.4.0/module.html)

### Useful Websites

1. [超算入门课程5 module命令使用教学](https://nscc.mrzhenggang.com/supercomputer-courses/module/)
2. [环境管理模块：Environment Modules](https://www.xiexianbin.cn/hpc/env-manage/environment-modules/index.html)
3. [软件模块使用方法](https://docs.hpc.sjtu.edu.cn/app/module.html)
4. [通过module加载已有软件环境](https://hpc.pku.edu.cn/_book/guide/soft/module.html)
