---
layout: post
title:  Linux系统-CentOS/Rocky系统的应用程序包
categories: [linux]
comments: true
tags: [Linux]
---

> 本文主要介绍CentOS/Rocky系统的应用程序包。

* TOC
{:toc}

<!--more-->

## VS Code编辑器

> 一款轻量级但功能强大的源代码编辑器，支持多种编程语言。

```shell
# 下载VS Code安装包
[root@localhost vscode]# wget https://vscode.download.prss.microsoft.com/dbazure/download/stable/0ee08df0cf4527e40edc9aa28f4b5bd38bbff2b2/code-1.85.1-1702462241.el7.x86_64.rpm

# 安装VS Code软件
[root@localhost vscode]# rpm -i code-1.80.2-1690491680.el7.x86_64.rpm
```

## PathWave Advanced Design System (ADS)安装

> 一款高级射频和微波设计软件，用于模拟和测试无线通信系统。

1. 官方下载ADS软件

[PathWave Advanced Design System (ADS) Software](https://www.keysight.com/us/en/lib/software-detail/computer-software/pathwave-advanced-design-system-ads-software-2212036.html)

2. 下载Crack文件及license

[linux Crack](http://bbs.eetop.cn/thread-471722-1-1.html)

3. 安装ads所需的软件

```shell
[user@c1 ~]$ yum install motif, motif-devel
```

4. 安装ADS软件

```shell
# 切换到ads软件目录中
[user@c1 ~]$ cd ads2015/

# 查看java程序的安装路径
[user@c1 ~]$  ls -lrt /usr/bin/java
lrwxrwxrwx. 1 root root 22 11月 24 14:21 /usr/bin/java -> /etc/alternatives/java
[user@c1 ~]$  ls -lrt /etc/alternatives/java
lrwxrwxrwx. 1 root root 73 11月 24 14:21 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/bin/java

# 载入java VM运行setup程序
[user@c1 ~]$ ./SETUP.SH LAX_VM /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/bin/java
```

5. 破解ADS软件

修改license文件

```shell
# 查看主机名称
[user@c1 ~]$ hostname
c1

# 查看MAC地址
[user@c1 ~]$ ifconfig
# mac地址为ac:1f:6b:fb:62:ac
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.111  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::8c18:4c0c:35b:f89c  prefixlen 64  scopeid 0x20<link>
        ether ac:1f:6b:fb:62:ac  txqueuelen 1000  (Ethernet)
        RX packets 396  bytes 43132 (42.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 189  bytes 25028 (24.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# 修改License文件
[user@c1 ~]$ gedit License.lic
# 将hostname修改为本机hostname
# 将112233445566修改为本机的mac地址
Server c1 ac1f6bfb62ac
DAEMON agileesofd
USE_SERVER
```

解压Patch文件夹

```shell
# 修改Patch文件的格式为zip
[user@c1 ~]$ mv Patch Patch.zip

# 解压Patch文件夹
[user@c1 ~]$ unzip Patch.zip

# 用Patch文件中的文件替换/opt/ADS2015/arttrans等文件夹
[user@c1 ~]$ sudo cp -r Patch ADS2015/*

# 修改/opt/ADS2015/Licensing/2014.07/linux_x86_64/bin/agileesofd的属性
[user@c1 ~]$ sudo chmod +x Patch /opt/ADS2015/Licensing/2014.07/linux_x86_64/bin/agileesofd

# 运行license管理服务，添加License.lic文件
[user@c1 ~]$ sudo /opt/ADS2015/Licensing/2014.07/linux_x86_64/bin/aglmmgr
```

6. 运行ads软件

```shell
[user@c1 ~]$ /opt/ADS2015_01/bin/ads
```

> ads无法运行
> - lmgrd: /lib64/ld-lsb-x86-64.so.3: bad ELF interpreter: No such file or directory
>> `yum -y install redhat-lsb`
> - Error while loading shared libraries: libXm.so.4 on CentOS 7
>> `yum install motif, motif-devel`

## Anaconda

> 一个包含 Python 版本的集成开发和数据科学平台，用于科学计算和数据分析。

```shell
[root@localhost ~]# wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2022.10-Linux-x86_64.sh
[root@localhost ~]# conda create -n py39 python=3.9
# rename "py39" to "py39_pyqt5"
[root@localhost ~]# conda rename -n py39 py39_pyqt5
```

## Qt5

> 一个跨平台的应用程序框架，用于开发图形用户界面应用程序，版本 5。

```
[root@localhost ~]# wget https://download.qt.io/archive/qt/5.14/5.14.2/qt-opensource-linux-x64-5.14.2.run
```

## PyQt5

> 一套基于 Qt5 的 Python 绑定库，用于创建 GUI 应用程序。

```shell
[root@localhost ~]# pip install PyQt5 PyQt5-tools scikit-rf PyQtchart pyqtgraph PyQtWebEngine statsmodels pyinstaller pygments mkl matplotlib intel-openmp localreg pymoo -i https://pypi.tuna.tsinghua.edu.cn/simple
[root@localhost ~]# pip install cython pycrypto iPython -i https://pypi.tuna.tsinghua.edu.cn/simple
[root@localhost ~]# pip install "PyQt-Fluent-Widgets[full]" -i https://pypi.org/simple/
[root@localhost ~]# conda install mpi4py
[root@localhost ~]# conda update -n base -c defaults conda
```

## PyQt6

> PyQt 的第六代版本，提供了对 Qt6 的支持。

```shell
[root@localhost ~]# pip install PyQt6 PyQt6-tools scikit-rf PyQtchart pyqtgraph PyQtWebEngine statsmodels pyinstaller pygments mkl matplotlib intel-openmp localreg pymoo -i https://pypi.tuna.tsinghua.edu.cn/simple
[root@localhost ~]# pip install "PyQt6-Fluent-Widgets[full]" -i https://pypi.org/simple/
[root@localhost ~]# conda install mpi4py
[root@localhost ~]# conda update -n base -c defaults conda

## Pyside6

> 一套基于 Qt6 的 Python 绑定库，用于创建 GUI 应用程序。

```shell
[root@localhost ~]# pip install Pyside6 scikit-rf pyqtgraph statsmodels pyinstaller pygments mkl matplotlib intel-openmp localreg pymoo -i https://pypi.tuna.tsinghua.edu.cn/simple
[root@localhost ~]# pip install "PySide6-Fluent-Widgets[full]" -i https://pypi.org/simple/
[root@localhost ~]# conda install mpi4py
[root@localhost ~]# conda update -n base -c defaults conda
```

## doxygen

一个流行的文档生成工具，用于从源代码自动生成文档。

```shell
[root@localhost ~]# yum install dnf-plugins-core
Last metadata expiration check: 0:03:54 ago on Thu 18 Jan 2024 03:15:55 PM CST.
Package dnf-plugins-core-4.0.21-23.el8.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[root@localhost ~]# dnf config-manager --set-enabled powertools
[root@localhost ~]# dnf -y install doxygen
```

## PyCharm

- **[PyCharm](https://www.jetbrains.com/pycharm/download/)**  
  PyCharm 是一个强大且多功能的集成开发环境（IDE），支持 Python、Web 开发、科学工具等。

## COMSOL

- **[COMSOL](https://www.comsol.com)**  
  COMSOL 是一个通用的仿真软件，允许您创建和部署各种工程领域的物理模型和应用。

## Ansys HFSS

- **[Ansys HFSS](https://www.ansys.com/products/electronics/ansys-hfss)**  
  Ansys HFSS 是一款用于设计和仿真高频电子产品（如天线、组件、互连等）的软件。

## Anaconda

- **[Anaconda](https://www.anaconda.com/)**  
  Anaconda 是一个免费的 Python 和 R 包及其工具的集合，专为数据科学、人工智能和机器学习设计。

### PyArma

- **[PyArma](https://pyarma.sourceforge.io/index.html)**  
  PyArmadillo 是一个用于 Python 的线性代数和科学计算库。

### SymPy

- **[SymPy](https://docs.sympy.org/latest/index.html)**  
  SymPy 是一个用于符号数学的 Python 库。

### NumPy

- **[NumPy](https://numpy.org/doc/stable/)**  
  NumPy 是 Python 中科学计算的基础包，提供了多维数组对象、各种派生对象（如蒙面数组和矩阵），以及一系列针对数组的操作，包括数学运算、逻辑运算、形状操作、排序、选择、I/O、离散傅立叶变换、基本线性代数、统计运算、随机模拟等。

### Pandas

- **[Pandas](https://pandas.pydata.org/docs/)**  
  Pandas 是一个开源的、基于 BSD 许可的数据结构和数据分析工具库，适用于 Python 编程语言。

### SciPy

- **[SciPy](https://docs.scipy.org/doc/scipy/)**  
  SciPy（发音为“Sigh Pie”）是一个开放源码的数学、科学和工程软件库。

### Matplotlib

- **[Matplotlib](https://matplotlib.org/stable/tutorials/index.html)**  
  Matplotlib 可以在图形窗口（如窗口、Jupyter 小部件等）中绘制您的数据，每个窗口都可以包含一个或多个坐标轴，这些坐标轴可用于指定 x-y 坐标（或极坐标图中的 θ-r，三维图中的 x-y-z 等）。

### Seaborn

- **[Seaborn](https://seaborn.pydata.org/tutorial/introduction.html)**  
  Seaborn 是一个用于 Python 的统计图形库，它构建在 Matplotlib 之上，并与 Pandas 数据结构紧密集成。

### PyAutoGUI

- **[PyAutoGUI](https://pyautogui.readthedocs.io/en/latest/)**  
  PyAutoGUI 允许您的 Python 脚本控制鼠标和键盘，以自动化与其他应用程序的交互。

### PySide6

- **[PySide6](https://doc.qt.io/qtforpython-6/index.html)**  
  Qt for Python 提供官方的 Python 绑定，使您能够使用 Python 编写 Qt 应用程序。项目包含两个主要部分：PySide6，用于在 Python 应用程序中使用 Qt6 API；Shiboken6，一个绑定生成工具，可用于向 Python 曝光 C++ 项目，并提供一些实用函数。

### PyQtGraph

- **[PyQtGraph 文档](https://pyqtgraph.readthedocs.io/en/latest/)**
- **[PyQtGraph 图形](https://www.pyqtgraph.org/)**  
  PyQtGraph 是一个纯 Python 图形和 GUI 库，基于 PyQt5/PySide2 和 NumPy 构建。

### PyInstaller

- **[PyInstaller](https://pyinstaller.org/en/stable/)**  
  PyInstaller 将 Python 应用程序及其所有依赖项打包成单个可执行文件。

### Numba

- **[Numba](https://numba.pydata.org/)**  
  Numba 在运行时将 Python 函数转换为优化的机器代码，使用业界标准的 LLVM 编译器库。使用 Numba 编译的 Python 数学算法可以接近 C 或 FORTRAN 的速度。

### CuPy

- **[CuPy](https://docs.cupy.dev/en/stable/user_guide/kernel.html)**
- **[CuPy 安装文档](https://docs.cupy.dev/en/stable/install.html)**  
  CuPy 是一个基于 NumPy 的开源库，可在使用 GPU 加速的情况下快速进行数组计算和处理。它支持与 Numba、NumPy、PyTorch 之间的数据通信。

  ```shell
  python -m pip install -U setuptools pip
  pip install cupy-cuda12x
  pip install cupy
  ```

## Additional Resources

### Documentation

1. [PathWave Advanced Design System (ADS) Software](https://www.keysight.com/us/en/lib/software-detail/computer-software/pathwave-advanced-design-system-ads-software-2212036.html)

### Useful Websites

1. [ADS 2020 射頻、微波和信号完整性仿真软件下载](https://www.mr-wu.cn/advanced-design-system-2020-free-download-and-crack/)
2. [RHEL7 安装ADS2015.01](http://www.edatop.com/ads/335884.html)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
