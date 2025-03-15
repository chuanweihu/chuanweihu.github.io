---
layout: post
title:  Armadillo使用手册3-数据的保存和读取
categories: [libs]
comments: true
tags: [Armadillo]
---

> 本文主要介绍Armadillo数据的保存和读取。

* TOC
{:toc}

<!--more-->

## 显示输出

> [print](https://arma.sourceforge.net/docs.html#print)

### 编程对应表

| Matlab | Armadillo | Notes |
| :--- | :--- | --- |
| `fprintf(formatSpec,A1,...,An)` | `printf(formatSpec,A1,...,An)` | 格式化输出显示|
| `disp(X)` | `X.print("X")/X.raw_print("X")/X.brief_print("X")` | 输出显示X矩阵|

### 示例介绍（matlab vs. cpp）

```cpp
// 1. 初始化A矩阵
// matlab
A = [1 2 3; 4 5 6; 7 8 9];
// cpp
A = \{\{1, 2, 3}, {4，5, 6}, {7, 8， 9}};

// 2. 格式化显示矩阵A中的（1，1）元素
// matlab
fprintf('A(1, 1) is %d\n', A(1, 1));
// cpp
printf('A(1, 1) is %d\n', A(0, 0));

// 3. 显示矩阵A
// matlab
disp(A);
// cpp
X.print("X");
X.raw_print("X");
X.brief_print("X");
```

## 数据保存和读取

> [save_load_mat](https://arma.sourceforge.net/docs.html#save_load_mat)

### 编程对应表

| Matlab | Armadillo | Notes |
| :--- | :--- | --- |
| `save(filename,variables)` | `.save( filename )` | 格式化输出显示|
| `load( filename )` | `.load( filename )` | 输出显示X矩阵|

### 示例介绍（matlab vs. cpp）

```cpp
// 1. 初始化A矩阵
// matlab
A = [1 2 3; 4 5 6; 7 8 9];
// cpp
A = \{\{1, 2, 3}, {4，5, 6}, {7, 8， 9}};

// 2. 保存A矩阵
// matlab
save('matrixA.mat', 'A');
// cpp
A.save("matrixA.bin");

// 3. 读取matrixA
// matlab
load('matrixA.mat', 'A');
// cpp
mat B;
bool = B.load("matrixA.bin");
if(ok == false){
  cout << "problem with loading" << endl;
}
```

## 更新日志

- date: 2024-02-22
  - desc: 添加目录
  - desc: 添加更新日志，推荐阅读和文档

## 推荐阅读

- [Armadillo使用手册1](https://www.huchuanwei.com/articles/2023-05/armadillo_userguide_1)
- [Armadillo使用手册2](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_2)
- [Armadillo使用手册3](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_3)

## 补充资源

### 文档

1. [pyarma](https://pyarma.sourceforge.io/docs.html)
2. [arma](https://arma.sourceforge.net)
3. [matrices-and-arrays](https://ww2.mathworks.cn/help/matlab/matrices-and-arrays.html)

### 网站

1. [Armadillo使用说明¶](https://docs.hpc.sjtu.edu.cn/app/compilers_and_languages/armadillo.html)
2. [C++线代运算库Armadillo安装与使用](http://zhaoxuhui.top/blog/2020/10/11/armadillo-introduction-and-installation.html)