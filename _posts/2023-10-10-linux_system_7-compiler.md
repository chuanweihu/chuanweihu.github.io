---
layout: post
title:  Linux系统-CentOS/Rocky系统的gcc编译
categories: [linux]
comments: true
tags: [Linux]
---

> 本文主要介绍通过gcc介绍C和C++语言的编译过程。

* TOC
{:toc}

<!--more-->

## 代码类型

使用 C 和 C++ 语言时，有三种类型的代码：

- 使用 C 或 C++ 语言编写的`源代码`，以纯文本文件形式存在。 
  - 文件通常使用扩展名，如 `.c,.cc,.cpp,.h,.hpp,.i,.inc`。有关支持的扩展及其解释的完整列表，请查看 gcc 手册页：
  - `$ man gcc`
- `对象代码`，通过使用 编译器 编译 源代码来创建。这是一种中间形式。 
  - 对象代码文件使用 .o 扩展名。
- `可执行代码`,通过带有一个 linker 的 linking 对象代码来创建。 
  - Linux 应用程序可执行文件不使用任何文件名扩展名。共享对象(library)可执行文件使用 .so 文件名扩展名。 
  - 也存在用于静态链接的库存档文件。这是使用 .a 文件名扩展名的对象代码变体。不建议使用静态链接。

## GCC编译步骤

> 从源代码生成可执行代码需要两个步骤，它们需要不同的应用程序或工具。GCC 可用作编译器和链路器的智能驱动程序。这可让您对任何所需操作使用单个命令 gcc。
> GCC 自动选择所需操作（编译和链接），以及它们的顺序： 源文件编译成对象文件。 对象文件和库链接（包括之前编译的源）。 
> 源文件-->预编译文件(.i)-->汇编文件(.s)-->目标文件(.o)+链接(.a和.so)-->可执行程序
> 1. objdump是一款可以查看目标文件的工具。例如`objdump -h simpleSection.o`
> 2. 使用readelf详细查看ELF文件。例如`readelf -h simpleSection.o`

### 预编译

```
# -E选项表示保留预处理器的输出文件(.i)
# -o选项表示将预编译结果输出为预编译文件(.i)
$ gcc -E main.c -o main.i
```

### 编译

```
# -S选项表示编译器将指定文件加工到编译阶段，并生成对应的汇编代码文件(.s)
# -fverbose-asm选项表示编译器为汇编代码添加必要的注释
$ gcc -S -fverbose-asm main.c 
```

### 汇编

```
# -c选项表示将汇编代码文件（或源文件）加工到汇编阶段，生成目标文件(.o)
# -o选项表示将汇编结果输出为目标文件(.o)
$ gcc -c main.s -o test.o
```

### 链接

```
# -L选项指定库文件的搜索路径，搜索路径可以设定环境变量
# -l选项指定库文件名（不需要lib前缀和.a或.so后缀)
## 链接静态库
$ gcc main.c -o main_exe /usr/lib/x86_64-redhat-linux6E/lib64/libm.a
$ gcc main.c -o main_exe -L/usr/lib/x86_64-redhat-linux6E/lib64 -lm
## 链接动态库
$ gcc main.c -o main_exe /usr/lib/x86_64-redhat-linux6E/lib64/libm.so
$ gcc main.c -o main_exe -L/usr/lib/x86_64-redhat-linux6E/lib64 -lm
```

### 生成可执行程序

```
# -o选项表示将目标文件作为输入文件，将可执行文件作为输出文件
$ gcc main.o -o main_exe
```

## GCC生成的库链接

> 库是一种可执行代码的二进制形式，可以被操作系统载入内存执行，主要有静态库和动态库，二者的不同点在于代码被载入的时刻不同
> 静态库的代码在编译过程中已经被载入可执行程序，因此体积较大
> 共享库的代码是在可执行程序运行时才载入内存的，在编译过程中仅简单的引用，因此代码体积较小
> file命令对文件格式检查(`file libtest.o`)
> nm命令对导出符号检查(`nm libtest.o`)
> ldd命令对动态库搜索情况检查（`ldd main_exe`)

### 静态库

```
# 1. 生成目标文件(.o)
$ gcc -c demo.c -o demo.o
# 2. 生成静态库文件
## -c选项表示创建archive（Create）
## -r选项表示将文件插入archive（Replacement）
## -v选项表示显示操作结果
$ ar -crv libdemo.a demo.o
# 3. 使用静态库生成可执行程序（main_exe)
$ gcc main.c -o main_exe ./libtest.a
$ gcc main.c -o main_exe -L. -ldemo
# 4. 运行可执行程序
$ ./main_exe
```

### 动态库

```
# 1. 生成目标文件(.o)
## -fPIC选项表示创建与地址无关的编译程序（PIC，position independent code）
$ gcc -fPIC -c demo.c -o demo.o
# 2. 生成动态库文件
## -shared选项表示生成动态链接库
$ gcc -shared -o demo.so demo.o
# 3. 使用动态库生成可执行程序（main_exe)
$ gcc main.c -o main_exe ./libtest.so
$ gcc main.c -o main_exe -L. -ldemo (需要添加动态库的搜索路径）
# 4. 运行可执行程序
$ ./main_exe
```

## Additional Resources

### Documentation

1. [第 15 章 使用 GCC 构建代码](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/7/html/developer_guide/gcc-compiling-code)

### Useful Websites

1. [C语言编译过程——预处理、编译汇编和链接详解](https://developer.aliyun.com/article/1414653)
2. [【Linux】—— 详解动态库和静态库](https://developer.aliyun.com/article/1460983)

### Related Books

1. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第三版](http://cn.linux.vbird.org/linux_basic/linux_basic.php)
2. [鸟哥的 Linux 私房菜 -- 基础学习篇目录 第四版](https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content)
