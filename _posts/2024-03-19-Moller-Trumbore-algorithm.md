---
layout: post
title:  "Möller–Trumbore算法"
date:   2024-03-19 19:38:00 +0800
categories: Computer-Graphics
math: true
---

Möller–Trumbore 算法（ Möller–Trumbore ray-triangle intersection algorithm ）用于三维三维中射线和三角形的快速求交。该算法由 Tomas Möller 和 Ben Trumbore 于 1997 年提出，是计算机图形学中最常用的求交算法之一。Möller–Trumbore 算法计算量小，适用于实时渲染。

## 算法引入

会写这个主要是因为在腾讯天美的二面被问到了，让我给出判断空间中射线是否穿过一个三角形的方法。我当时的回答很常规，大概是这样：

空间平面上任意一点 $p$ 的表达式：

$$ (P - P') \cdot \vec{n} = 0 $$

空间射线上任意一点的表达式：

$$ r(t) = O + t \vec{d} $$

联立可解得：

$$ t = \frac{(P'-O)\cdot\vec{n}}{\vec{d}\cdot\vec{n}} $$

带入射线表达式后解出交点坐标，然后检查点与三角形每个顶点的向量叉乘的符号，如果符号都相同即在三角形内部。

这么做是最符合正常人已有的思维模式的，但效率不够高。因此有Möller–Trumbore 算法。

## 算法推导

首先三角形平面上任意一点可以写成重心坐标的形式：

$$ P = b_0P_0 + b_1P_1 + b_2P_2 = (1-b1-b2)P_0 + b_1P_1 + b_2P_2 $$​

把空间射线上任意一点的表达式带入得：

$$ O + t \vec{d} = (1-b1-b2)P_0 + b_1P_1 + b_2P_2 $$

单纯看这个式子，从直观上可以说我们一定有解决的办法。因为一共有 $t$、$ b_1$ 和 $b_2 $ 三个未知量，而三维空间让我们可以有三个方程。

我们将上面的式子整理一下：

$$ O - P_0 = b_1(P_1-P_0) + b_2(P_2-P_0) - t\vec{d} $$

将其中的已知量进行替换：

$$ S = O - P $$

$$ E_1 = P_1 - P_0 $$

$$ E_2 = P_2 - P_0 $$

得到：

$$ S = b_1E_1 + b_2E_2 - t\vec{d} =\begin{bmatrix} -\vec{d} & E_1 & E_2 \end{bmatrix} \begin{bmatrix} t \\ b_1 \\ b_2 \end{bmatrix} $$

根据 Cramer 法则和向量混合积的性质：

$$ t = \frac{\det\begin{bmatrix} S & E_1 & E_2 \end{bmatrix}}{\det\begin{bmatrix} -\vec{d} & E_1 & E_2 \end{bmatrix}} = \frac{(S\times E_1)\cdot E_2}{ ( \vec{d}\times E_2)\cdot E_1 }$$

同理分别解出 $b_1$ 和 $b_2$ ：

$$ b_1 = \frac{(\vec{d}\times E_2)\cdot S}{ ( \vec{d}\times E_2)\cdot E_1 } $$

$$b_2= \frac{(S\times E_1)\cdot \vec{d}}{ ( \vec{d}\times E_2)\cdot E_1 } $$

同样进行已知量的替换：

$$ S_1 = d\times E_2$$

$$S_2 = S\times E_1$$

最后得到：

$$\begin{bmatrix} t \\ b_1 \\ b_2 \end{bmatrix} = \frac{1}{S_1\cdot E_1} \begin{bmatrix} S_2\cdot E_2 \\ S_1 \cdot S \\ S_2 \cdot \vec{d} \end{bmatrix}$$

其中：

$$ S = O - P $$

$$ E_1 = P_1 - P_0 $$

$$ E_2 = P_2 - P_0 $$

$$ S_1 = d\times E_2$$

$$S_2 = S\times E_1$$

## 代码实现

```cpp
bool ray_triangle_intersection(const Point3f& P0, const Point3& P1, const Point3& P2, 
                               const Vector3f& orig, const Vector3f& dir, 
                               float& t, float& b1, float& b2) {
    auto S = orig - P0;
    auto E1 = P1 - P0;
    auto E2 = P2 - P0;
    auto S1 = cross(dir, E2);
    auto S2 = cross(S, E1);

    auto divisor = dot(S1, E1);
    if (divisor == 0) 
        return false;
    t = dot(S2, E2) / divisor;
    b1 = dot(S1, S) / divisor;
    b2 = dot(S2, dir) / divisor;

    if (t < 0 || b1 < 0 || b2 < 0 || b1 + b2 > 1) 
        return false;
        
    return true;
}
```

