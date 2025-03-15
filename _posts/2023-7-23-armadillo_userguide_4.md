---
layout: post
title:  Armadillo使用手册4-扩展函数
categories: [libs]
comments: true
tags: [Armadillo]
---

> 本文主要介绍Armadillo的扩展函数。

* TOC
{:toc}

<!--more-->

## 扩展函数

### setdiff

> 设置两个数组的差集

#### 语法

`arma::mat C = setdiff(A,B)`

#### 说明

`C = setdiff(A,B)`返回 A 中存在但 B 中不存在的数据，不包含重复项。C 是有序的。

```cpp
arma::mat setdiff(arma::mat A, arma::mat B){
    arma::mat sortedA = arma::sort(arma::unique(A));
    if (B.is_empty()){
        return sortedA;
    }
    else{
        double num, indexC, flag;

        arma::mat intersectAB = arma::intersect(A, B);
        arma::mat C(1, sortedA.n_elem, arma::fill:value(-1.0));
        arma::mat CRes = {};
        arma::uvec findABRes = {};
        
        indexC = 0;
        flag = 0;
        
        for(int i = 0; i < sortedA.n_elem; i++){
            num = sortedA(i);
            findABRes = arma::find(intersectAB == num);
            if (findABRes.is_empty()){
                C(0, indexC) = num;
                flag = 1;
                indexC++;
            }
        }

        if (flag){
            CRes = C.cols(0, indexC - 1);
        }
        return CRes;
    }
}
```

#### 示例

- 两个向量的差集

```cpp
/*
C = 1×3

     1     3     5
*/

// 查找 A 中存在，但 B 中不存在的值。
arma::mat A = {3, 6, 2, 1, 5, 1, 1}; 
arma::mat B = {2, 4, 6};
arma::mat C = setdiff(A, B);
```

### ismember

> 判断数组元素是否为集数组成员

#### 语法

`arma::field<arma::mat> C = ismember(A, B)`

#### 说明

`[Lia,Locb] = ismember(A, B)` 对于 A 中属于 B 的成员的每一个值，Lia返回一个在该位置包含逻辑值 1 (true) 的数组，Locb 会包含该值在B中的最小索引。值为 0 表示 A 不是 B 的成员。

```cpp
arma::field<arma::mat> ismember(arma::mat A, arma::mat B){
    arma::filed<arma::mat> result(2, 1);
    arma::uvec findRes = {};
    arma::mat Lia(1, A.n_elem, arma::fill::zeros), Locb(1, A.n_elem, arma
    ::fill::zeros);
    arma::mat notFind = {};

    for (int i = 0; i < A.n_elem; i++){
        findRes = arma::find(B == A[i]);
        if (!findRes.is_empty()){
            Lia[i] = 1;
            Locb[i] = arma::min(findRes) + 1;
        }
        else{
            Lia[i] = 0;
            Locb[i] = 0;
        }
    }

    if (arma::accu(Lia) == 0 && arma::accu(Locb) == 0){
        result(0, 0) = notFind;
        result(1, 0) = notFind;
    }
    else{
        result(0, 0) = Lia;
        result(1, 0) = Locb;
    }

    return result;
}
```

#### 示例

- 判断查询值是否为集成员并确定共有值的索引

```cpp
/*
Lia = 1x4 logical array

   0   0   1   1

Locb = 1×4

     0     0     2     1
*/

// A(3) 的最小索引为 B(2)，A(4) 可以在 B(1) 中找到。
arma::mat A = {5, 3, 4, 2,}; 
arma::mat B = {2, 4, 4, 4, 6, 8};
arma::field<arma::mat> C = ismember(A, B);
arma::mat Lia = C(0, 0);
arma::mat Locb = C(1, 0);
```

## 更新日志

- date: 2024-02-22
  - desc: 添加目录
  - desc: 添加更新日志，推荐阅读和文档

## 推荐阅读

- [Armadillo使用手册1](https://www.huchuanwei.com/articles/2023-05/armadillo_userguide_1)
- [Armadillo使用手册2](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_2)
- [Armadillo使用手册3](https://www.huchuanwei.com/articles/2023-06/armadillo_userguide_3)
- [Armadillo使用手册4](https://www.huchuanwei.com/articles/2023-07/armadillo_userguide_4)

## 补充资源

### 文档

1. [pyarma](https://pyarma.sourceforge.io/docs.html)
2. [arma](https://arma.sourceforge.net)
3. [matrices-and-arrays](https://ww2.mathworks.cn/help/matlab/matrices-and-arrays.html)
4. [ismember](https://ww2.mathworks.cn/help/matlab/ref/double.ismember.html)
5. [setdiff](https://ww2.mathworks.cn/help/matlab/ref/double.setdiff.html)

### 网站

1. [Armadillo使用说明¶](https://docs.hpc.sjtu.edu.cn/app/compilers_and_languages/armadillo.html)
2. [C++线代运算库Armadillo安装与使用](http://zhaoxuhui.top/blog/2020/10/11/armadillo-introduction-and-installation.html)