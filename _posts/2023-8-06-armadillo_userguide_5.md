---
layout: post
title:  Armadillo使用手册5-常见的功能表格
categories: [libs]
comments: true
tags: [Armadillo]
---

> 本文主要介绍Armadillo常见的功能表格整理。

* TOC
{:toc}

<!--more-->

## 功能表格

### 重载操作

| Matlab | Armadillo | 用途 | 描述 | 参考页 |
| --- | --- | --- | --- | --- |
| 基本算术 | | | | |
| `-` | `A-k` | 减法 | 表示将 `A`中所有元素减去`k` | [`minus`](https://ww2.mathworks.cn/help/matlab/ref/minus.html) |
| `-` | `k-A` | 减法 | 表示将 `A`中所有元素减去`k` | [`minus`](https://ww2.mathworks.cn/help/matlab/ref/minus.html) |
| `+` | `A+k， k+A` | 加法 | 表示将 `A`中所有元素加上`k` | [`plus`](https://ww2.mathworks.cn/help/matlab/ref/plus.html) |
| `*` | `A*k， k*A` | 乘法 | 表示将 `A`中所有元素乘以`k` | [`times`](https://ww2.mathworks.cn/help/matlab/ref/times.html) |
| `+` | `A+B` | 加法 | `A+B` 表示将 `A` 和 `B` 加在一起 | [`plus`](https://ww2.mathworks.cn/help/matlab/ref/plus.html) |
| `-` | `A-B` | 减法 | 表示从 `A` 中减去 `B` | [`minus`](https://ww2.mathworks.cn/help/matlab/ref/minus.html) |
| `*` | `A*B` | 乘法 | 表示将 `A`乘以`B` | [`times`](https://ww2.mathworks.cn/help/matlab/ref/times.html) |
| `.*` | `A%B` | 按元素乘法 | 表示 `A` 和 `B` 的逐元素乘积。 | [`times`](https://ww2.mathworks.cn/help/matlab/ref/times.html) |
| `./` | `A/B` | 数组右除 | 表示包含元素 `A(i,j)/B(i,j)` 的矩阵。 | [`rdivide`](https://ww2.mathworks.cn/help/matlab/ref/rdivide.html) |
| 关系运算 | | | | |
| `==` | `A==B` | 等于 | 表示比较A和B矩阵相等关系的位置的逻辑矩阵。 | [`relational-operators`](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-comparison-with-relational-operators.html) |
| `~=` | `A!=B` | 不等于 | 表示比较A和B矩阵不相等关系的位置的逻辑矩阵。 | [`relational-operators`](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-comparison-with-relational-operators.html) |
| `>=` | `A>=B` | 大于等于 | 表示比较A和B矩阵大于等于关系的位置的逻辑矩阵。 | [`relational-operators`](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-comparison-with-relational-operators.html) |
| `<=` | `A<=B` | 小于等于 | 表示比较A和B矩阵小于等于关系的位置的逻辑矩阵。 | [`relational-operators`](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-comparison-with-relational-operators.html) |
| `>` | `A>B` | 大于 | 表示比较A和B矩阵大于关系的位置的逻辑矩阵。 | [`relational-operators`](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-comparison-with-relational-operators.html) |
| `<` | `A<B` | 小于 | 表示比较A和B矩阵小于关系的位置的逻辑矩阵。 | [`relational-operators`](https://ww2.mathworks.cn/help/matlab/matlab_prog/array-comparison-with-relational-operators.html) |

### 成员函数

| Function/Variable | Description |
| --- | --- |
| | |
| .n_rows | 返回行数 |
| .n_cols | 返回列数 |
| .n_elem | 返回所有元素个数 |
| | |
| (i) | 访问第i个元素，按照列优先获取值 |
| (r,c) | 访问第r行，c列的元素 |
| [i] | 访问第i个元素，按照列优先获取值，无边界检查，仅用于调式后 |
| .at(r,c) | 访问第r行，c列的元素，无边界检查，仅用于调式后 |
| .memptr() | 获取对象的指针 |
| | |
| .in_range(i) | 检查给定的坐标(i)是合法的 |
| .in_range(r,c) | 检查给定的坐标(r,c)是合法的 |
| | |
| .reset() | 把维数设置成0，意味着无元素 |
| .copy_size(A) | 把维数设置成和A一样 |
| .set_size(rows, cols) | 改变矩阵大小为rows行，cols列（不保留矩阵中内容） (fast) |
| .reshape(rows, cols) | 改变矩阵大小为rows行，cols列（按列优先保留矩阵中内容） (slow) |
| .resize(rows, cols) | 改变矩阵大小为rows行，cols列（不保留矩阵中内容） (slow) |
| | |
| .ones(rows, cols) | 设置rows行和cols列的数据为1 |
| .zeros(rows, cols) | 设置rows行和cols列的数据为0 |
| .randu(rows, cols) | 把矩阵的值设置成从均匀分布中抽取的随机值 |
| .randn(rows, cols) | 与.randu()相同,只不过从正态分布中抽取随机数 |
| .fill(k) | 将所有元素设置为k |
| | |
| .is_empty() | 检查是否为空 |
| .is_finite() | 检查对象是否有限 |
| .is_square() |检查是否是方阵 |
| .is_vec() | 检查一个矩阵是否是向量 |
| .is_sorted() | 检查对象是否是被排列过的 |
| | |
| .has_inf() | 检查是否含有inf值 |
| .has_nan() | 检查是否含有NaN |
| | |
| .begin() | 返回第一个元素 |
| .end() | 返回最后一个元素 |
| .begin_row(i) | 返回第i行的第一个元素 |
| .end_row(j) | 返回第i行的最后一个元素 |
| .begin_col(i) | 返回第i列的第一个元素 |
| .end_col(j) | 返回第i列的最后一个元素 |
| | |
| .print(header) | 打印此对象 |
| .brief_print(header) | 简略打印此对象 |
| .raw_print(header) | 打印此对象 |
| | |
| .save(name, format) | 向文件或流写入对象 |
| .load(name, format) | 从文件或流读取对象 |
| | |
| .diag(k) | 访问矩阵X的第k个对角线(k是可选的，主对角线为k=0，上对角线为k>0，下对角线为k<0) |
| .row(i) | 访问矩阵X的第i行 |
| .col(i) | 访问矩阵X的第i列 |
| .rows(a,b) | 访问矩阵X从第a行到第b行的子矩阵 |
| .cols(c,d) | 访问矩阵X从第c列到第d列的子矩阵 |
| .submat( span(a,b), span(c,d) ) | 访问矩阵从第a行到第b行和第c列到第d列的子矩阵 |
| .submat( P, q, size(A) ) | 访问矩阵从第a行到第b行和第c列到第d列的子矩阵 |
| .rows( vector of row indices) | 访问矩阵X相应行索引的子矩阵 |
| .cols( vector of col indices ) | 访问矩阵X相应列索引的子矩阵 |
| .elem( vector of ,indices ) | 访问矩阵X相应索引的子矩阵 |
| | |
| .each_row() | 对每行进行向量操作 |
| .each_col() | 对每列进行向量操作 |
| .swap_rows(p,q) | 交换行 |
| .swap_cols(p,q) | 交换列 |
| .insert_rows(row,X) | 插入Xl到第row行 |
| .insert_cols(col,X) | 插入X到第col列 |
| .shed_rows(first_row,last_row) | 移除特first_row行到last_row行 |
| .shed_cols(first_col,last_col) | 移除first_col列到last_col |
| | |
| .min() | 返回矩阵或立方体的最小值 |
| .max() | 返回矩阵或立方体的最大值 |
| .index_min() |返回矩阵或立方体最小值的坐标；返回值为一个无符号整数 |
| .index_max() | 返回矩阵或立方体最大值的坐标；返回值为一个无符号整数 |

## 更新日志

- date: 2024-02-22
  - desc: 添加目录
  - desc: 添加更新日志，推荐阅读和文档

## 推荐阅读

- [Armadillo使用手册1](https://www.huchuanwei.com/articles/2023-05/armadillo_userguide_1)
- [Armadillo使用手册2](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_2)
- [Armadillo使用手册3](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_3)
- [Armadillo使用手册4](https://www.huchuanwei.com/articles/2023-07/armadillo_userguide_4)
- [Armadillo使用手册5](https://www.huchuanwei.com/articles/2023-08/armadillo_userguide_5)

## 补充资源

### 文档

1. [pyarma](https://pyarma.sourceforge.io/docs.html)
2. [arma](https://arma.sourceforge.net)
3. [Armadillo: a template-based C++ library for linear algebra.](https://arma.sourceforge.net/armadillo_joss_2016.pdf)
4. [arithmetic-operators](https://ww2.mathworks.cn/help/matlab/arithmetic-operators.html)

### 网站

1. [Armadillo使用说明¶](https://docs.hpc.sjtu.edu.cn/app/compilers_and_languages/armadillo.html)
2. [C++线代运算库Armadillo安装与使用](http://zhaoxuhui.top/blog/2020/10/11/armadillo-introduction-and-installation.html)