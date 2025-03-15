---
layout: post
title:  Armadillo使用手册2-数值的通用操作
categories: [study]
comments: true
tags: [Armadillo]
---

> 本文主要介绍Armadillo基础数值的通用操作。

* TOC
{:toc}

<!--more-->

## 创建数组

### 编程对应表

| Function | Description | Matlab | Armadillo | Note |
| --- | --- | --- | --- | --- |
| 矩阵matrix：A=[] | 返回一个 2×2 的初始化矩阵 | A=[1,2; 3,4] | arma::mat A=\{\{1,2},{3,4}} | $A=[1,2; 3,4]$|
| eye(n, m) | 返回一个主对角线元素为 1 且其他位置元素为 0 的 n×m 矩阵; 如果n=m,返回一个主对角线元素为 1 且其他位置元素为 0 的 n×n 单位矩阵。 | `I = eye(2,3)` | `arma::mat A = arma::eye(2,3);` | $y1 = [1,0,0; 0,1,0]$ |
| ones(n, m) | 返回一个 n×m 的全 1 矩阵 | `X = ones(2,3)` | `arma::mat A = arma::ones(2,3);` | $y1 = [1,1,1; 1,1,1]$ |
| zeros(n, m) | 返回一个 n×m 的全 0 矩阵 | `X = zeros(2,3)` | `arma::mat A = arma::zeros(2,3);` | $y1 = [1,0,0; 0,1,0]$ |
| linspace(x1, x2, n) | 返回包含 x1 和 x2 之间的 n 个等间距点的行向量。 | `y1 = linspace(-5,5,7)` | `arma::vec A = arma::linspace(-5,5,7);` | $y1 = [-5.0000,-3.3333,-1.6667,0,1.6667,3.3333,5.0000]$ |
| logspace(a, b, n) | 在 10 的幂 10^a 和 10^b（10 的 N 次幂）之间生成 n 个点。 | `y1 = logspace(1,5,7)` | `arma::vec A = arma::logspace(1,5,7);` | $y1 = 10^5 * [0.0001,0.0005,0.0022,0.0100,0.0464,0.2154,1.0000]$ |
| x = j:i:k | 创建一个等间距向量 x，以 i 作为元素之间的增量 | `y = 10:-2:0` | `arma::vec A = arma::regspace(10,-2,0);` | $y1 = [10,8,6,4,2,0]$|
| rand(n, m) | 生成一个由介于 0 和 1 之间的均匀分布的随机数组成的 n×m 矩阵。 | `A1 = rand(1,5)` | `arma::mat A1 = arma::randu(1, 5);` | $A1 = [0.8147,0.9058,0.1270,0.9134,0.6324]$ |
| randn(n, m) | 生成一个由标准正态分布的随机数组成的 n×m 矩阵。 | `B1 = randn(1,5)` | `arma::mat B1 = arma::randn(1, 5);` | $B1 = [0.5377,1.8339,-2.2588,0.8622,0.3188]$ |
| sparse(n, m) | 生成 m×n 全零稀疏矩阵。 | `S = sparse(10000,5000)` | `arma::sp_mat S(1000,1000);` | $S = All zero sparse: 10000x5000$ |
| repelem(A, m, n) | 创建一个矩阵，并将A中每个元素复制到一个新矩阵的 m×n 块中。 | `A = [1 2; 3 4]; B = repelem(A,3,2);` | `arma::mat A = \{\{1,2},{3,4}}; arma::mat B = arma::repelem(A,3,2);` | $A = [1 2; 3 4]; B = [1, 1, 2, 2; 1, 1, 2, 2; 1, 1, 2, 2; 3, 3, 4, 4; 3, 3, 4, 4; 3, 3, 4, 4;]$ |
| repmat(A, m, n) | 将矩阵副本重复到 m×n 块排列中。 | `A = diag([100 200 300]); B = repmat(A,2,3);` | `arma::vec v = {100,200,300}; arma::mat A = diagmat(v); arma::mat B = arma::repmat(A,2,3);` | $A = [100, 0, 0; 0, 200, 0; 0, 0, 300]; B = [100, 0, 0, 100, 0, 0, 100, 0, 0; 0, 200, 0, 0, 200, 0, 0, 200, 0; 0, 0, 300, 0, 0, 300, 0, 0, 300; 100, 0, 0, 100, 0, 0, 100, 0, 0; 0, 200, 0, 0, 200, 0, 0, 200, 0; 0, 0, 300, 0, 0, 300, 0, 0, 300;]$ |
| 元胞数组cell：C={A; B} | 创建一个空的 2×1 元胞数组。 | `C = {A1; B1};` | `arma::field<mat> C(2, 1); C(0, 0)=A1; C(1, 0)=B1;` | $C = {[0.8147,0.9058,0.1270,0.9134,0.6324];[0.5377,1.8339,-2.2588,0.8622,0.3188]}$ |

### 示例介绍（matlab vs. cpp）

```cpp
// 1. 创建一个向量(vector)或矩阵(matrix)
/*
I11 = 4×1

 1
 2
 3
 4

I12 = 4×1

 4
 3
 2
 1

I21 = 2×2

 1 2
 3 4

I22 = 3×2

 22 22
 22 22
 22 22

I3 = 4×1

 1.0000 + 4.0000i
 2.0000 + 3.0000i
 3.0000 + 2.0000i
 4.0000 + 1.0000i
*/
// matlab
I11 = [1;2;3;4]; // 生成4×1矩阵
I12 = [4;3;2;1]; // 生成4×1矩阵
I21 = [1,2; 3,4]; // 生成2×2矩阵
I22 = repmat(22,3,2); // 生成全部元素为22的3×2矩阵
I3 = complex(I11, I12); // 生成4×1复数矩阵
// cpp
arma::vec I11={1,2,3,4}; // 生成4×1矩阵
arma::vec I12={4,3,2,1}; // 生成4×1矩阵
arma::mat I21=\{\{1,2},{3,4}}; // 生成2x2矩阵
arma::mat I22=arma::mat(3, 2, fill::value(22)); // 生成全部元素为22的3×2矩阵
arma::cx_mat I3=arma::cx_mat(I11, I12);// 生成4×1复数矩阵

// 2. 创建一个主对角线元素为1的向量或矩阵(eye)
/*
I1 = 3×1

 1
 0
 0

I2 = 2×3

 1 0 0
 0 1 0
*/
// matlab
I1 = eye(3, 1); // 生成3×1单位向量
I2 = eye(2, 3); // 生成2×3单位矩阵
// cpp
arma::vec I1 = eye(3, 1); // 生成3×1单位向量
arma::vec I1(3, fill::eye);
arma::mat I2 = eye(2, 3); // 生成2×3单位矩阵
arma::mat I2(2, 3， fill::eye);

// 3. 创建一个全部值为1的向量或矩阵(ones)
/*
I1 = 3×1

 1
 1
 1

I2 = 2×3

 1 1 1
 1 1 1
*/
// matlab
I1 = ones(3, 1); // 生成3×1单位向量
I2 = ones(2, 3); // 生成2×3单位矩阵
// cpp
arma::vec I1 = ones(3, 1); // 生成3×1单位向量
arma::vec I1(3, fill::ones);
arma::mat I2 = ones(2, 3); // 生成2×3单位矩阵
arma::mat I2(2, 3， fill::ones);

// 4. 创建一个全部值为0的向量或矩阵
/*
I1 = 3×1

 0
 0
 0

I2 = 2×3

 0 0 0
 0 0 0
*/
// matlab
I1 = zeros(3, 1); // 生成3×1单位向量
I2 = zeros(2, 3); // 生成2×3单位矩阵
// cpp
arma::vec I1 = zeros(3, 1); // 生成3×1单位向量
arma::vec I1(3, fill::zeros);
arma::mat I2 = zeros(2, 3); // 生成2×3单位矩阵
arma::mat I2(2, 3， fill::zeros);

// 5. 创建一个包含 x1 和 x2 之间的 n 个等间距点的行向量(linspace)
/*
I1 = 1×7

 -5.0000 -3.3333 -1.6667 0 1.6667 3.3333 5.0000

I2 = 1×7

 0 0.833333 1.6667 2.5000 3.3333 4.1667 5.0000
*/
// matlab
I1 = linspace(-5,5,7) // 创建一个由区间 [-5,5] 中的 7 个等间距点组成的向量
I2 = linspace(0,5,7) // 创建一个由区间 [0,5] 中的 7 个等间距点组成的向量
// cpp
arma::vec I1 = arma::linspace(-5,5,7); // 创建一个由区间 [-5,5] 中的 7 个等间距点组成的向量
arma::uvec I2 = arma::linspace<uvec>(0,5,7); // 创建一个由区间 [0,5] 中的 7 个等间距点组成的无符号向量

// 6. 创建一个在 10 的幂 10^a 和 10^b（10 的 N 次幂）之间生成 n 个的行向量(logspace)
/*
y1 = 1×7
105 ×

 0.0001 0.0005 0.0022 0.0100 0.0464 0.2154 1.0000
*/
// matlab
I1 = logspace(1,5,7) // 创建一个由区间 [10^1,10^5] 内的 7 个对数间距点组成的向量
// cpp
arma::vec I1 = arma::logspace(1,5,7); // 创建一个由区间 [10^1,10^5] 内的 7 个对数间距点组成的向量
arma::uvec I1 = arma::logspace<uvec>(1,5,7); // 创建一个由区间 [10^1,10^5] 内的 7 个对数间距点组成的无符号向量

// 7. 创建一个等间距向量 x，以 i 作为元素之间的增量(regspace)
/*
I1 = 1×6

 10 8 6 4 2 0
*/
// matlab
I1 = 10:-2:0 // 创建一个由区间 [10,0] 内以-2 作为元素之间的增量的向量
// cpp
arma::vec A = arma::regspace(10,-2,0); // 创建一个由区间 [10,0] 内以-2 作为元素之间的增量的向量
arma::uvec A = arma::regspace<uvec>(10,-2,0); // 创建一个由区间 [10,0] 内以-2 作为元素之间的增量的向量

// 8. 创建一个由介于 0 和 1 之间的均匀分布的随机数组成的 n×m 矩阵(rand)
/*
I1 = 1×5

 0.8147 0.9058 0.1270 0.9134 0.6324
*/
// matlab
I1 = rand(1,5) // 生成一个由介于 0 和 1 之间的均匀分布的随机数组成的1×5向量
// cpp
arma::mat I1 = arma::randu(1, 5); // 生成一个由介于 0 和 1 之间的均匀分布的随机数组成的1×5向量

// 9. 创建一个由标准正态分布的随机数组成的 n×m 矩阵(randn)
/*
I1 = 1×5

 0.5377 1.8339 -2.2588 0.8622 0.3188
*/
// matlab
I1 = randn(1,5) // 生成一个由正态分布的随机数组成的1×5向量
// cpp
arma::mat I1 = arma::randn(1, 5); // 生成一个由正态分布的随机数组成的1×5向量

// 10. 生成 m×n 全零稀疏矩阵(sparse)
/*
S = 
   All zero sparse: 10000x5000

*/
// matlab
S = sparse(10000,5000) // 生成一个全零的10000×5000稀疏矩阵
// cpp
arma::sp_mat S(1000,1000); // 生成一个全零的10000×5000稀疏矩阵

| sparse(n, m) | 生成 m×n 全零稀疏矩阵。 | `S = sparse(10000,5000)` | `arma::sp_mat S(1000,1000);` | $S = All zero sparse: 10000x5000$ |

// 11. 创建一个矩阵，并将其每个元素复制到一个新矩阵的 m×n 块中。(repelem)
/*
A = 2×2

     1     2
     3     4

B = 6×4

     1     1     2     2
     1     1     2     2
     1     1     2     2
     3     3     4     4
     3     3     4     4
     3     3     4     4

*/
// matlab
A = [1 2; 3 4] // 生成一个的2×2矩阵
B = repelem(A,3,2) // 将其A中每个元素复制到一个新矩阵的 3×2 块中
// cpp
arma::mat A = \{\{1,2},{3,4}}； // 生成一个的2×2矩阵
arma::mat B = arma::repelem(A,3,2); // 将其A中每个元素复制到一个新矩阵的 3×2 块中

// 12. 将矩阵副本重复到 m×n 块排列中(repmat)
/*
A = 3×3

   100     0     0
     0   200     0
     0     0   300

B = 6×9

   100     0     0   100     0     0   100     0     0
     0   200     0     0   200     0     0   200     0
     0     0   300     0     0   300     0     0   300
   100     0     0   100     0     0   100     0     0
     0   200     0     0   200     0     0   200     0
     0     0   300     0     0   300     0     0   300
*/
// matlab
A = diag([100 200 300]); // 生成一个的3×3矩阵
B = repmat(A,2,3); // 将矩阵副本A重复到 2×3 块排列中
// cpp
arma::vec v = {100,200,300}; // 生成一个的1×3向量 
arma::mat A = diagmat(v); // 生成一个的3×3矩阵 
arma::mat B = arma::repmat(A,2,3); // 将矩阵副本A重复到 2×3 块排列中

// 13. 创建一个元胞数组
/*
I1=2×1 cell array
 {1x5 double}
 {1x5 double}
*/
// matlab
C = {A1; B1}; // 创建由大A1和B1组成的元胞数组
// cpp
arma::field<mat> C(2, 1); // 创建由大A1和B1组成的元胞数组（仅支持同类型数据）
C(0, 0)=A1; 
C(1, 0)=B1;
```

## 数组索引

根据元素在数组中的位置（索引）访问数组元素的方法主要有三种：按位置索引、线性索引和逻辑索引。

### 位置索引

通过访问索引位置来确定矩阵中的元素，例如通过元素的行号和列号来访问摸个元素。

#### 编程对应表

| Matlab | Armadillo | Notes |
| :--- | :--- | --- |
| `A(1, 1)` | `A(0, 0)` | 获取矩阵第一行第一列的元素 |
| `A(k, k)` | `A(k-1, k-1)` | 获取矩阵第k行第k列的元素 |
| `A(:, k)` | `A.col(k)` | 获取矩阵第k列的元素 |
| `A(k, :)` | `A.row(k)` | 获取矩阵第k行的元素 |
| `A(:, p:q)` | `A.cols(p, q)` | 获取矩阵第p到q列的元素 |
| `A(p:q, :)` | `A.rows(p, q)` | 获取矩阵第p到q行的元素 |
| `A(p:q, r:s)` | `A(span(p,q),span(r,s))` | 获取矩阵第p到q行且第r到s列的元素 |
| `A[p:delta1:q, r:delta2:s]` | `uvec row_ind = regspace(p, delta1, q); uvec col_ind = regspace(r, delta2, s); A(row_ind, col_ind);` | 获取矩阵第p到q行且第r到s列的元素，步长为delta1和delta2 |
| `Q(:, :, k)` | `Q.slice(k)` | 获取矩阵第k页的二维矩阵元素 |
| `Q(:, :, t:u)` | `Q.slices(t, u)` | 获取矩阵第k到u页的二维矩阵元素 |
| `Q(p:q, r:s, t:u)` | `Q(span(p,q),span(r,s),span(t,u))` | 获取矩阵第t到u页第p到q行第r到s列的二维矩阵元素 |

#### 示例介绍（matlab vs. cpp）

```cpp
// 1. 初始化A矩阵
// matlab
A = [1 2 3 4; 5 6 7 8; 9 10 11 12; 13 14 15 16];
// cpp
A = \{\{1, 2, 3, 4}, {5, 6, 7, 8}, {9, 10, 11, 12}, {13, 14, 15, 16}};

// 2. 显式指定元素的索引
// matlab
e = A(3, 2);
// cpp
int e = A(2, 1);

// 3. 一个向量中指定多个元素的索引，从而一次引用多个元素。
// matlab
r = A(2, [1, 3]);
// cpp 
arma::uvec row_ind = {1};  
arma::uvec col_ind = {0, 2};  
arma::mat r = A(row_ind, col_ind); 
// cpp 
arma::mat r = A.submat(row_ind, col_ind);
// cpp
arma::uvec indices = {1, 10};
arma::mat r = A.elem(indices); 

// 4. 要访问某个行范围或列范围内的元素
// matlab
r = A(1:3, 2:4);
r = 1;
r = 2;
// cpp 
arma::mat r = A(span(0,2), (1,3));
arma::uvec row_ind = arma::regspace<arma::uvec>(0,  1,  2);
arma::uvec col_ind = arma::regspace<arma::uvec>(1,  1,  3);
arma::mat r = A(row_ind, col_ind);
// cpp
arma::mat r = A.submat(row_ind, col_ind);
r.ones(); //某个行范围或列范围内的元素变为1
r.fill(1.0); //某个行范围或列范围内的元素变为1
r.fill(2.0); //某个行范围或列范围内的元素变为2

// 5. 使用关键字 end 指定第二直至最后一列或行
// matlab
r = A(1:3,2:end)
// cpp 
arma::uvec row_ind = arma::regspace<arma::uvec>(0,  1,  2);
arma::uvec col_ind = arma::regspace<arma::uvec>(1,  1,  A.n_cols-1);
arma::mat r = A(row_ind, col_ind); 
// cpp 
arma::mat r = A.submat(row_ind, col_ind);

// 6. 如果要访问所有行或所有列，只使用冒号运算符即可。
// matlab
r = A(:,3)
r = A(3,:)
// cpp 
arma::mat r = A.col(2);
arma::mat r = A.row(2);

// 7. 如果要访问所有行或所有列，然后访问某个列或行的部分元素，可以使用冒号运算符。
// matlab
r = A(:,1:3)
r = A(1:3,:)
// cpp 
arma::mat r = A.cols(0, 2);
arma::mat r = A.rows(0, 2);
```

### 线性索引

访问数组元素的另一种方法是只使用单个索引，而不管数组的大小或维度如何。此方法称为线性索引。虽然 MATLAB 根据定义的大小和形状显示数组，但实际上数组在内存中都存储为单列元素。

#### 编程对应表

| Matlab | Armadillo | Notes |
| :--- | :--- | --- |
| `ind2sub(sz,ind)` | `ind2sub(size(A), index)` | 将矩阵的线性索引转换为下标 |
| `sub2ind(sz,row,col)` | `sub2ind(size(A), row, col)` | 将下标转换为矩阵的线性索引 |

#### 示例介绍（matlab vs. cpp）

```cpp
// 1. 初始化A矩阵
// matlab
A = [1 2 3 4; 5 6 7 8; 9 10 11 12; 13 14 15 16];
// cpp
A = \{\{1, 2, 3, 4}, {5, 6, 7, 8}, {9, 10, 11, 12}, {13, 14, 15, 16}};

// 2. 线性化矩阵
// matlab
Alinear = A(:);
// cpp
arma::mat Alinear = arma::vectorise(A);

// 3. 线性索引访问元素
// matlab
e = A(3,2);
elinear = A(10);
// cpp
int e = A(2, 1);
int elinear = A(9);

// 4. 原始索引和线性索引之间进行转换
// matlab
linearidx = sub2ind(size(A),3,2);
// cpp
int linearidx = arma::sub2ind(arma::size(A),3,2);

// matlab
[row,col] = ind2sub(size(A),10);
// cpp
arma::uvec u = arma::ind2sub(arma::size(A),9);
int row = u(0);
int col = u(1);
```

### 逻辑索引

使用 true 和 false 逻辑指示符也可以对数组进行索引，在处理条件语句时尤其便利。

#### 编程对应表

| Matlab | Armadillo | Notes |
| :--- | :--- | --- |
|ind=A>0; A(ind)|arma::uvec ind=find(A>0); A(ind)||

#### 示例介绍（matlab vs. cpp）

```cpp
// 1. 初始化A矩阵
// matlab
A = [1 2 6; 4 3 6];
B = [0 3 7; 3 7 5];
// cpp
A = \{\{1, 2, 6}, {4, 3, 6}};
B = \{\{0, 3, 7}, {3, 7, 5}};

// 2. 取出矩阵A中的元素小于矩阵B中的对应元素
// matlab
ind = A<B;
C = A(ind);
D = B(ind);
C = 1; //某个行范围或列范围内的元素变为1
C = 2; //某个行范围或列范围内的元素变为2

// cpp
arma::umat ind = arma::find(AA >= BB);
arma::mat C = A(ind);
arma::mat D = B(ind);
C.ones(); //某个行范围或列范围内的元素变为1
C.fill(1.0); //某个行范围或列范围内的元素变为1
C.fill(2.0); //某个行范围或列范围内的元素变为2
```

## 矩阵操作

#### 编程对应表

| Matlab | Armadillo | Notes |
| :--- | :--- | --- |
| `reshape( X, n_rows, n_cols )` | `arma::reshape( X, n_rows, n_cols )` | 重构 |
| `A.'或transpose(A)` | `A.st()` | 非共轭转置 |
| `A'` | `A.t()` | 共轭转置 |
| `flipud(X)` | `arma::flipud(X)` | 上下翻转矩阵的行 |
| `fliplr(X)` | `arma::fliplr(X)` | 左右翻转矩阵的列 |
| `circshift(A,K,dim)` | `arma::shift( X, N, dim ))` | 循环将 A 中的值沿维度 dim 平移 K 个位置 |
| `X = A(:)` | `arma::vec X =arma::vectorise(A)` | 将 A 中的所有元素重构成一个列向量。|
| `X = [AB]` | `arma::mat X =arma::join_horiz(A,B)` | 按行串联矩阵 |
| `X =[A;B]` | `arma::mat X =arma::join_vert(A,B)` | 按列串联矩阵 |

#### 示例介绍（matlab vs. cpp）

```cpp
// 1. 将 3×4 矩阵重构成 2×6 矩阵(reshape)
/*
A = 3×4

     1     4     7    10
     2     5     8    11
     3     6     9    12

B = 2×6

     1     3     5     7     9    11
     2     4     6     8    10    12
*/

// matlab
B = reshape(A,2,6);
// cpp
arma::mat B = arma::reshape(A,2,6);
arma::mat B = A.reshape(2,6);

// 2. 计算一个 2×2 复矩阵的非共轭转置和共轭转置(transpose)
/*
A = 2×2 complex

   1.0000 + 1.0000i   1.0000 - 1.0000i
   0.0000 - 1.0000i   0.0000 + 1.0000i

B = 2×2 complex

   1.0000 + 1.0000i   0.0000 - 1.0000i
   1.0000 - 1.0000i   0.0000 + 1.0000i

C = 2×2 complex

   1.0000 - 1.0000i   0.0000 + 1.0000i
   1.0000 + 1.0000i   0.0000 - 1.0000i
*/

// matlab
B = A.';
B = transpose(A);
C = A';
// cpp
arma::mat B = A.st();
arma::mat C = A.t();

// 3. 将 2×2 矩阵上下和左右翻转(flipud和fliplr)
/*
A = 2×2

     1     2
     3     4

B = 2×2

     3     4
     1     2

C = 2×2

     2     1
     4     3
*/

// matlab
B = flipud(A);
C = fliplr(A);
// cpp
arma::mat B = arma::flipud(A);
arma::mat C = arma::fliplr(A);

// 4. 将矩阵的各行向上平移 1 个位置而各列保持不动(shift)
/*
A = 3×4

     1     2     3     4
     5     6     7     8
     9    10    11    12

B = 3×4

     5     6     7     8
     9    10    11    12
     1     2     3     4
*/

// matlab
B = circshift(A,[-1 0]);
// cpp
arma::mat B = arma::shift(A,-1,0);

// 5. 将矩阵中的所有元素重构成一个列向量(vectorise)
/*
A = 3×3

     8     1     6
     3     5     7
     4     9     2

B = 9×1

     8
     3
     4
     1
     5
     9
     6
     7
     2
*/

// matlab
B = A(:);
// cpp
arma::vec B = arma::vectorise(A);

// 6. 按行或列串联矩阵(join_horiz/join_vert)
/*
A = 2×2

     1     1
     1     1


B = 2×2

     0     0
     0     0

C = 4×2

     1     1
     1     1
     0     0
     0     0

D = 2×4

     1     1     0     0
     1     1     0     0
*/

// matlab
C = [A B];
D = [A; B];
// cpp
arma::mat C =arma::join_horiz(A,B);
arma::mat D =arma::join_vert(A,B);
```

## 更新日志

- date: 2024-02-22
  - desc: 添加目录
  - desc: 添加更新日志，推荐阅读和文档

## 推荐阅读

- [Armadillo使用手册1](https://www.huchuanwei.com/articles/2023-05/armadillo_userguide_1)
- [Armadillo使用手册2](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_2)

## 补充资源

### 文档

1. [pyarma](https://pyarma.sourceforge.io/docs.html)
2. [arma](https://arma.sourceforge.net)
3. [matrices-and-arrays](https://ww2.mathworks.cn/help/matlab/matrices-and-arrays.html)

### 网站

1. [Armadillo使用说明¶](https://docs.hpc.sjtu.edu.cn/app/compilers_and_languages/armadillo.html)
2. [C++线代运算库Armadillo安装与使用](http://zhaoxuhui.top/blog/2020/10/11/armadillo-introduction-and-installation.html)