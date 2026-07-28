---
title: '欧几里得算法'
published: '0001-01-01'
date: "公元前 300 年"
draft: false
tags: ['最大公约数', '递归']
series: '基础算法'
toc: true
coverImage:
  src: './euclidean-algorithm-cover.jpg'
  alt: '欧几里得算法（辗转相除法）原理示意图'
---

**欧几里得算法**（Euclidean Algorithm），又称**辗转相除法**，是用于计算两个非负整数 $a$ 和 $b$ 的**最大公约数**（Greatest Common Divisor, GCD）的最古老、最高效的算法之一。

该算法最早记录于公元前 300 年左右古希腊数学家欧几里得所著的《几何原本》第七卷中，是人类已知且至今仍被广泛使用的最古老的数值算法之一。

## 核心原理

算法的核心原理建立在以下数学定理之上：

> **定理：** 两个整数 $a$ 和 $b$（$a > b$）的最大公约数，等于 $b$ 和 $a$ 除以 $b$ 的余数 $r$（即 $a \bmod b$）的最大公约数。

用公式表达即为：

$$\gcd(a, b) = \gcd(b, a \bmod b)$$

**终止条件：** 当余数为 $0$ 时，此时的除数就是两者的最大公约数，即 $\gcd(a, 0) = a$。

### 简易推导

假设 $d$ 是 $a$ 和 $b$ 的公约数，即 $a = k_1 d$ 且 $b = k_2 d$。

由于余数 $r = a - q \cdot b$（其中 $q$ 是商），代入可得：

$$r = k_1 d - q \cdot k_2 d = (k_1 - q \cdot k_2) d$$

因此，$d$ 必定也是余数 $r$ 的约数。由此证明了 $\gcd(a, b)$ 与 $\gcd(b, a \bmod b)$ 的等价性。

## 计算示例：求解 $\gcd(252, 105)$

以下是逐步求解过程：

1. $252 \div 105 = 2 \dots \dots 42 \implies \gcd(252, 105) = \gcd(105, 42)$
2. $105 \div 42 = 2 \dots \dots 21 \implies \gcd(105, 42) = \gcd(42, 21)$
3. $42 \div 21 = 2 \dots \dots 0 \implies \gcd(42, 21) = \gcd(21, 0)$

当余数为 $0$ 时，计算结束，最大公约数为 **$21$**。

## 代码实现 (C++)

```c++
#include <numeric> // 使用 std::gcd 需要引入此头文件

// 1. 递归实现
long long gcd_recursive(long long a, long long b) {
    return b == 0 ? a : gcd_recursive(b, a % b);
}

// 2. 迭代实现（效率更高，无栈溢出风险）
long long gcd_iterative(long long a, long long b) {
    while (b != 0) {
        long long temp = a % b;
        a = b;
        b = temp;
    }
    return a;
}

// 3. 极简递归写法（三目运算符）
long long gcd(long long a, long long b) {
    return b ? gcd(b, a % b) : a;
}

// 4. C++17 标准库直接调用
// auto ans = std::gcd(a, b);
```

## 复杂度与应用

- **时间复杂度：** $O(\log(\min(a, b)))$。算法每执行两次，问题规模至少缩小一半（Lamé 定理保证了极高的运行效率，即使输入上千位的庞大整数也能快速得出结果）。
- **空间复杂度：** 迭代版本为 $O(1)$，递归版本为 $O(\log(\min(a, b)))$ 栈空间。
- **主要扩展与应用：**
  - **扩展欧几里得算法（Extended Euclidean Algorithm）：** 不仅求出 $\gcd(a, b)$，还能求解贝祖等式（Bézout's identity） $ax + by = \gcd(a, b)$ 中的整数解 $(x, y)$。
  - **密码学：** 是 RSA 公钥加密算法中求解模逆元（Modular Multiplicative Inverse）的核心工具。