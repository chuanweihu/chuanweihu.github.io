---
layout: post
title:  Armadillo使用手册1-基础数值类型
categories: [study]
comments: true
tags: [Armadillo]
---

> Armadillo是一个高性能的Cpp矩阵运算库，与Matlab的调用方式非常相似，对于熟悉Matlab的人来说，使用Armadillo库很方便。本文主要介绍Armadillo支持的数据类型。

* TOC
{:toc}

<!--more-->

## 基础数值类型

> - [Row: Classes for row vectors](https://arma.sourceforge.net/docs.html#Row)
> - [Col: Classes for column vectors](https://arma.sourceforge.net/docs.html#Col)
> - [Mat: Classes for dense matrices, with elements stored in column-major ordering](https://arma.sourceforge.net/docs.html#Mat)
> - [SpMat: Classes for sparse matrices; intended for storing large matrices, where most of the elements are zeros](https://arma.sourceforge.net/docs.html#SpMat)
> - [Cube: Classes for cubes (quasi 3rd order tensors), also known as "3D matrices"](https://arma.sourceforge.net/docs.html#Cube)
> - [field: Class for storing arbitrary objects in matrix-like or cube-like layouts](https://arma.sourceforge.net/docs.html#field)

| data | Armadillo | Row(行向量) | Col(列向量) | Mat(二维矩阵) | SpMat(二维稀疏矩阵) | Cube(三维矩阵) | field(域) | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `an unsigned integer` | `uword` | `urowvec = Row<uword>`| `uvec = ucolvec = Col<uword>`| `umat = Mat<uword>`| `sp_umat = SpMat<uword>`| `ucube = Cube<uword>` | `field<urowvec>`, `field<uvec>`, `field<umat>` | 无符号整型 |
| `a signed integer type` | `sword` | `irowvec = Row<sword>`| `ivec = icolvec = Col<sword>`| `imat = Mat<sword>`| `sp_imat = SpMat<sword>` | `icube = Cube<sword>` | `field<irowvec>`, `field<ivec>`, `field<imat>` | 有符号整型 |
| `float` | `float` | `frowvec = Row<float>` | `fvec = fcolvec = Col<float>` | `fmat = Mat<float>` | `sp_fmat = SpMat<float>` | `fcube = Cube<float>` | `field<frowvec>`, `field<fvec>`, `field<fmat>` | 单精度浮点数 |
| `double` | `double` | `drowvec = Row<double>` 或者 `rowvec = Row<double>` | `dvec = dcolvec = Col<double>` 或者 `vec = colvec = Col<double>` | `dmat = Mat<double>` 或者 `mat = Mat<double>`  | `sp_dmat = SpMat<double>` 或者 `sp_mat = SpMat<double>` | `dcube = Cube<double>` 或者 `cube = Cube<double>` | `field<rowvec>`, `field<vec>`, `field<mat>` | 双精度浮点数 |
| `std::complex<float>` | `cx_float` | `cx_frowvec = Row<cx_float>` | `cx_fvec = cx_fcolvec = Col<cx_float>` | `cx_fmat = Mat<cx_float>`  | `sp_cx_fmat = SpMat<cx_float>` | `cx_fcube = Cube<cx_float>` | `field<cx_frowvec>`, `field<cx_fvec>`, `field<cx_fmat>` | 单精度浮点复数 |
| `std::complex<double>` | `cx_double` | `cx_drowvec = Row<cx_double>` 或者 `cx_rowvec = Row<cx_double>` | `cx_dvec = cx_dcolvec = Col<cx_double>` 或者 `cx_vec = cx_colvec = Col<cx_double>` | `cx_dmat = Mat<cx_double>` 或者 `cx_mat = Mat<cx_double>`  | `sp_cx_dmat = SpMat<cx_double>` 或者 `sp_cx_mat = SpMat<cx_double>` | `cx_dcube = Cube<cx_double>` 或者 `cx_cube = Cube<cx_double>` | `field<cx_rowvec>`, `field<cx_vec>`, `field<cx_dmat>` | 双精度浮点复数 |

### 1D-行向量

> [Row<type>](https://arma.sourceforge.net/docs.html#Row)

对于1D-列向量，`Row<type>`类源自`Mat<type>`，并且继承了`Mat<type>`的大部分方法。

#### 构造函数

- 形式
  - `Row<type>`
- 数据类型
  - `float, double, std::complex<float>, std::complex<double>, short, int, long, and unsigned versions of short, int, long`
- 简写形式
  - `rowvec = Row<double>`：双精度浮点数
  - `drowvec = Row<double>`：双精度浮点数
  - `frowvec = Row<float>`：单精度浮点数
  - `cx_rowvec = Row<cx_double>`：复数双精度浮点数
  - `cx_drowvec = Row<cx_double>`：复数双精度浮点数
  - `cx_frowvec = Row<cx_float>`：复数单精度浮点数
  - `urowvec = Row<uword>`：无符号整数
  - `irowvec = Row<sword>`：有符号整数
- 构造函数
  - `rowvec()`
  - `rowvec(n_elem)`
  - `rowvec(n_elem, fill_form)`        (elements are initialised according to fill_form)
  - `rowvec(size(X))`
  - `rowvec(size(X), fill_form) `       (elements are initialised according to fill_form)
  - `rowvec(rowvec)`
  - `rowvec(mat)`        (std::logic_error exception is thrown if the given matrix has more than one row)
  - `rowvec(initializer_list)`
  - `rowvec(string)`        (elements separated by spaces)
  - `rowvec(std::vector)`
  - `cx_rowvec(rowvec,rowvec)`        (for constructing a complex row vector out of two real row vectors)
- 初始化值
  - `fill::zeros` ↦ set all elements to 0
  - `fill::ones` ↦ set all elements to 1
  - `fill::randu` ↦ set all elements to random values from a uniform distribution in the [0,1] interval
  - `fill::randn` ↦ set all elements to random values from a normal/Gaussian distribution with zero mean and unit variance
  - `fill::value(scalar)`     ↦ set all elements to specified scalar
  - `fill::none` ↦ do not initialise the elements

#### 示例介绍

```cpp
// 创建10个元素的行向量，且全部元素初始化为0
arma::rowvec x(10);

// 创建10个元素的行向量，且全部元素初始化为1
arma::rowvec y(10, arma::fill::ones);

// 创建10x10的矩阵，且全部元素初始化为[0, 1]之间，且为均匀分布
arma::mat A(10, 10, arma::fill::randu);

// 获取矩阵第5行元素创建行向量（从0开始）
arma::rowvec z = A.row(5); // extract a row vector
```

### 1D-列向量

> [Col<type>](https://arma.sourceforge.net/docs.html#Col)

对于1D-列向量，`Col<type>`类源自`Mat<type>`，并且继承了`Mat<type>`的大部分方法

#### 构造函数

- 形式
  - `Col<type>`
- 数据类型
  - `float, double, std::complex<float>, std::complex<double>, short, int, long, and unsigned versions of short, int, long`
- 简写形式
  - `vec = colvec = Col<double>`：双精度浮点数
  - `dvec = dcolvec = Col<double>`：双精度浮点数
  - `fvec = fcolvec = Col<float>`：单精度浮点数
  - `cx_vec = cx_colvec = Col<cx_double>`：复数双精度浮点数
  - `cx_dvec = cx_dcolvec = Col<cx_double>`：复数双精度浮点数
  - `cx_fvec = cx_fcolvec = Col<cx_float>`：复数单精度浮点数
  - `uvec = ucolvec = Col<uword>`：无符号整数
  - `ivec = icolvec = Col<sword>`：有符号整数
- 构造函数
  - `vec()`
  - `vec(n_elem)`
  - `vec(n_elem, fill_form)`        (elements are initialised according to fill_form)
  - `vec(size(X))`
  - `vec(size(X), fill_form)`        (elements are initialised according to fill_form)
  - `vec(vec)`
  - `vec(mat)`        (std::logic_error exception is thrown if the given matrix has more than one column)
  - `vec(initializer_list)`
  - `vec(string)`        (elements separated by spaces)
  - `vec(std::vector)`
  - `cx_vec(vec,vec)`        (for constructing a complex vector out of two real vectors)
- 初始化值
  - `fill::zeros` ↦ set all elements to 0
  - `fill::ones` ↦ set all elements to 1
  - `fill::randu` ↦ set all elements to random values from a uniform distribution in the [0,1] interval
  - `fill::randn` ↦ set all elements to random values from a normal/Gaussian distribution with zero mean and unit variance
  - `fill::value(scalar)`     ↦ set all elements to specified scalar
  - `fill::none` ↦ do not initialise the elements

#### 示例介绍

```cpp
// 创建10个元素的列向量，且全部元素初始化为0
arma::vec x(10);

// 创建10个元素的列向量，且全部元素初始化为1
arma::vec y(10, arma::fill::ones);

// 创建10x10的矩阵，且全部元素初始化为[0, 1]之间，且为均匀分布
arma::mat A(10, 10, arma::fill::randu);
// 获取矩阵第5列元素创建列向量（从0开始）
arma::vec z = A.col(5); // extract a column vector
```

### 2D-矩阵

#### 稠密矩阵

> [Mat<type>](https://arma.sourceforge.net/docs.html#Mat)

对于稠密矩阵，矩阵中的元素和Matlab的存储方式一致，都是Column-major形式，即矩阵中的列元素在内存中是连续的

##### 构造函数

- 形式
  - `Mat<type>`
- 数据类型
  - `float, double, std::complex<float>, std::complex<double>, short, int, long, and unsigned versions of short, int, long`
- 简写形式
  - `mat = Mat<double>`：双精度浮点数
  - `dmat = Mat<double>`：双精度浮点数
  - `fmat = Mat<float>`：单精度浮点数
  - `cx_mat = Mat<cx_double>`：复数双精度浮点数
  - `cx_dmat = Mat<cx_double>`：复数双精度浮点数
  - `cx_fmat = Mat<cx_float>`：单精度浮点数
  - `umat = Mat<uword>`：无符号整数
  - `imat = Mat<sword>`：有符号整数
- 构造函数
  - `mat()`
  - `mat(n_rows, n_cols)`
  - `mat(n_rows, n_cols, fill_form)`        (elements are initialised according to fill_form)
  - `mat(size(X))`
  - `mat(size(X), fill_form)`        (elements are initialised according to fill_form)
  - `mat(mat)`
  - `mat(vec)`
  - `mat(rowvec)`
  - `mat(initializer_list)`
  - `mat(string)`
  - `mat(std::vector)`        (treated as a column vector)
  - `mat(sp_mat)`        (for converting a sparse matrix to a dense matrix)
  - cx_mat(mat,mat)`        (for constructing a complex matrix out of two real matrices)
- 初始化值
  - `fill::zeros` ↦ set all elements to 0
  - `fill::ones` ↦ set all elements to 1
  - `fill::eye` ↦ set the elements on the main diagonal to 1 and off-diagonal elements to 0
  - `fill::randu` ↦ set all elements to random values from a uniform distribution in the [0,1] interval
  - `fill::randn` ↦ set all elements to random values from a normal/Gaussian distribution with zero mean and unit variance
  - `fill::value(scalar)`     ↦ set all elements to specified scalar
  - `fill::none` ↦ do not initialise the elements

##### 示例介绍

```cpp
// 创建5x5矩阵，并初始化元素值为[0, 1]之间
arma::mat A(5, 5, fill::randu);

// 取出矩阵中的元素值
double x = A(1,2);

// 矩阵的基本运算
// operators   +  −  *  %  /  ==  !=  <=  >=  <  >  &&  ||
arma::mat B = A + A;
arma::mat C = A * B;
arma::mat D = A % B;

// 构建复数矩阵
arma::cx_mat X(A,B);

// 将B矩阵初始化为0
B.zeros();

// 将B矩阵的维度重置为10x10
B.set_size(10,10);

// 将B矩阵的维度修改为5x6，并将元素初始化为1
B.ones(5,6);

// 矩阵打印全部输出
B.print("B:");

// 矩阵打印简略输出
B.brief_print("B:");

// 创建5x6的维度固定复数矩阵
mat::fixed<5,6> F;

// 创建24个元素的double数组
double aux_mem[24];
// 创建4x6的二维矩阵，并使用aux_MEM的内存地址
arma::mat H(&aux_mem[0], 4, 6, false);  // use auxiliary memory
```

#### 稀疏矩阵

> [SpMat<type>](https://arma.sourceforge.net/docs.html#SpMat)

对于稀疏矩阵（矩阵中主要元素为零），Armadillo有专门的矩阵类型存储

##### 构造函数

- 形式
  - `SpMat<type>`
- 数据类型
  - `float, double, std::complex<float>, std::complex<double>, short, int, long, and unsigned versions of short, int, long`
- 简写形式
  - `sp_mat = SpMat<double>`：双精度浮点数
  - `sp_dmat = SpMat<double>`：双精度浮点数
  - `sp_fmat = SpMat<float>`：单精度浮点数
  - `sp_cx_mat = SpMat<cx_double>`：复数双精度浮点数
  - `sp_cx_dmat = SpMat<cx_double>`：复数双精度浮点数
  - `sp_cx_fmat = SpMat<cx_float>`：复数单精度浮点数
  - `sp_umat = SpMat<uword>`：无符号整数
  - `sp_imat = SpMat<sword>`：有符号整数
- 构造函数
  - `sp_mat()`
  - `sp_mat(n_rows, n_cols)`
  - `sp_mat(size(X))`
  - `sp_mat(sp_mat)`
  - `sp_mat(vec)`            (for converting a dense matrix to a sparse matrix)
  - `cx_mat(sp_mat,sp_mat)`        (for constructing a complex matrix out of two real matrices)
- 初始化值
  - `fill::zeros` ↦ set all elements to 0
  - `fill::ones` ↦ set all elements to 1
  - `fill::eye` ↦ set the elements on the main diagonal to 1 and off-diagonal elements to 0
  - `fill::randu` ↦ set all elements to random values from a uniform distribution in the [0,1] interval
  - `fill::randn` ↦ set all elements to random values from a normal/Gaussian distribution with zero mean and unit variance
  - `fill::value(scalar)`     ↦ set all elements to specified scalar
  - `fill::none` ↦ do not initialise the elements

##### 示例介绍

```cpp
// 创建1000x2000矩阵，矩阵中有1%的元素值为[0, 1]之间，且为均匀分布
arma::sp_mat A = arma::sprandu(1000, 2000, 0.01);

// 创建2000x1000矩阵，矩阵中有1%的元素值为[0, 1]之间，且为0-1正态分布
arma::sp_mat B = arma::sprandn(2000, 1000, 0.01);

// 创建矩阵
arma::sp_mat C = 2*B;

// 创建矩阵
arma::sp_mat D = A*C;

// 创建2000x1000矩阵，且在（1，2）的位置赋值123
arma::sp_mat E(1000,1000);
E(1,2) = 123;

// 批量插入3个值在 (1, 2), (7, 8), (9, 9)三个位置
arma::umat locations = { { 1, 7, 9 },
                   { 2, 8, 9 } };
arma::vec values = { 1.0, 2.0, 3.0 };
arma::sp_mat X(locations, values);
```

### 3D-矩阵

> [Cube<type>](https://arma.sourceforge.net/docs.html#Cube)

对于3D矩阵，矩阵中的元素通过一组切片（矩阵）实现连续存储，在每一个切片数据中，矩阵中的元素和Matlab的存储方式一致，都是Column-major形式，即矩阵中的列元素在内存中是连续的，第三维通过一系列的切片来实现连续存储；3D矩阵有自动回收机制，也可以通过reset()手动释放内存

#### 构造函数

- 形式
  - `Cube<type>`
- 数据类型
  - `float, double, std::complex<float>, std::complex<double>, short, int, long, and unsigned versions of short, int, long`
- 简写形式
  - `cube = Cube<double>`：双精度浮点数
  - `dcube = Cube<double>`：双精度浮点数
  - `fcube = Cube<float>`：单精度浮点数
  - `cx_cube = Cube<cx_double>`：复数双精度浮点数
  - `cx_dcube = Cube<cx_double>`：复数双精度浮点数
  - `cx_fcube = Cube<cx_float>`：复数单精度浮点数
  - `ucube = Cube<uword>`：无符号整数
  - `icube = Cube<sword>`：有符号整数
- 构造函数
  - `cube()`
  - `cube(n_rows, n_cols, n_slices)`
  - `cube(n_rows, n_cols, n_slices, fill_form)`        (elements are initialised according to fill_form)
  - `cube(size(X))`
  - `cube(size(X), fill_form)`        (elements are initialised according to fill_form)
  - `cube(cube)`
  - `cx_cube(cube, cube)`        (for constructing a complex cube out of two real cubes)
- 初始化值
  - `fill::zeros` ↦ set all elements to 0
  - `fill::ones` ↦ set all elements to 1
  - `fill::randu` ↦ set all elements to random values from a uniform distribution in the [0,1] interval
  - `fill::randn` ↦ set all elements to random values from a normal/Gaussian distribution with zero mean and unit variance
  - `fill::value(scalar)`     ↦ set all elements to specified scalar
  - `fill::none` ↦ do not initialise the elements

#### 示例介绍

```cpp
// 创建1x2x3矩阵，并初始化元素值为0
arma::cube x(1, 2, 3);

// 创建4x5x6矩阵，且全部元素初始化为[0, 1]之间，且为均匀分布
arma::cube y(4, 5, 6, arma::fill::randu);

// 取出3d矩阵中的第1个矩阵（从0开始）
arma::mat A = y.slice(1);  // extract a slice from the cube
                     // (each slice is a matrix)

// 创建4x5矩阵，且全部元素初始化为[0, 1]之间，且为均匀分布
arma::mat B(4, 5, fill::randu);
// 将B矩阵赋值给3d矩阵中的第2个矩阵（从0开始）
y.slice(2) = B;     // set a slice in the cube

// 3d矩阵的基本运算
// operators   +  −  *  %  /  ==  !=  <=  >=  <  >  &&  ||
arma::cube q = y + y;     // cube addition
arma::cube r = y % y;     // element-wise cube multiplication

// 创建4x5x6的维度固定复数矩阵，并初始化元素值为0
arma::cube::fixed<4,5,6> f;

// 将f矩阵的元素初始化为1
f.ones();
```

### field

> [field<object_type>](https://arma.sourceforge.net/docs.html#field)

field用于存储任意矩阵形式的数据，里面的每一个元素不是标量，而是向量或者矩阵；每一个元素可以是任意大小；对于同样大小的数据，Cube类更有效率

#### 构造函数

- 形式
  - `field<object_type>`
- 数据类型
  - `float, double, std::complex<float>, std::complex<double>, short, int, long, and unsigned versions of short, int, long`
- 构造函数
  - `field<object_type>()`
  - `field<object_type>(n_elem)`
  - `field<object_type>(n_rows, n_cols)`
  - `field<object_type>(n_rows, n_cols, n_slices)`
  - `field<object_type>(size(X))`
  - `field<object_type>(field<object_type>)`

#### 示例介绍

```cpp
// 创建2x3矩阵，且全部元素初始化为[0, 1]之间，且为0-1正态分布
arma::mat A = arma::randn(2,3);
// 创建4x5矩阵，且全部元素初始化为[0, 1]之间，且为0-1正态分布
arma::mat B = arma::randn(4,5);

// 创建2x1的“容器”
arma::field<arma::mat> F(2,1);
// 将A矩阵赋值给容器的（0，0）位置
F(0,0) = A;
// 将B矩阵赋值给容器的（1，0）位置
F(1,0) = B; 

// 打印容器
F.print("F:");

// 保存容器名为mat_field的文件
F.save("mat_field");
```

## 更新日志

- date: 2024-02-22
  - desc: 添加目录
  - desc: 添加更新日志，推荐阅读和文档

## 推荐阅读

- [Armadillo使用手册1](https://www.huchuanwei.com/articles/2023-05/armadillo_userguide_1)

## 补充资源

### 文档

1. [pyarma](https://pyarma.sourceforge.io/docs.html)
2. [arma](https://arma.sourceforge.net)

### 网站

1. [Armadillo使用说明¶](https://docs.hpc.sjtu.edu.cn/app/compilers_and_languages/armadillo.html)
2. [C++线代运算库Armadillo安装与使用](http://zhaoxuhui.top/blog/2020/10/11/armadillo-introduction-and-installation.html)