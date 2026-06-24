---
title: '多重背包问题'
published: 1975-01-01
draft: false
tags: ['动态规划', '多重背包问题']
toc: true
---

> 给定一个容量为 $W$ 的背包，以及 $n$ 种不同的物品。对于每一种物品 $i$，我们已知：
>
> - **重量** $w_i$
> - **价值** $v_i$
> - **数量限制** $c_i$（即每种物品最多只能取 $c_i$ 个）
>
> **目标：** 在不超过背包总容量 $W$ 的前提下，选择若干物品装入背包，使得装入物品的总价值最大。

由于多重背包限制了每种物品的数量，直接使用动态规划处理时，根据复杂度和场景的不同，有三种主流策略：

## 基础思路：转化为 0/1 背包

最直观的方法是将第 $i$ 种物品的 $c_i$ 个物品看作 $c_i$ 个独立的物品，这样就完全转化为了 0/1 背包问题。但如果 $c_i$ 很大，时间复杂度会非常高。

```c++
#include <vector>
#include <algorithm>

/**
 * 多重背包基础实现（二维数组，输入下标从 1 开始）
 * W: 背包容量
 * weights: 物品重量 (假设大小为 n+1，索引 1 到 n 有效)
 * values: 物品价值 (假设大小为 n+1，索引 1 到 n 有效)
 * counts: 物品数量 (假设大小为 n+1，索引 1 到 n 有效)
 */
int multipleKnapsack2D_Index1(int W, const std::vector<int>& weights, 
                              const std::vector<int>& values, const std::vector<int>& counts) {
    int n = weights.size() - 1;
    
    // 1. 统计拆分后的总物品数
    int total_items = 0;
    for (int i = 1; i <= n; ++i) total_items += counts[i];

    // 2. 初始化 dp 表: dp[i][j] 表示处理到第 i 个拆分后的物品时，容量为 j 的最大价值
    std::vector<std::vector<int>> dp(total_items + 1, std::vector<int>(W + 1, 0));

    int current_idx = 1;
    // 3. 遍历每种原始物品
    for (int i = 1; i <= n; ++i) {
        int w = weights[i];
        int v = values[i];
        int c = counts[i];

        // 4. 将每种物品的 c 个副本视为独立的 0/1 背包物品
        for (int k = 1; k <= c; ++k) {
            for (int j = 1; j <= W; ++j) {
                if (j < w) {
                    dp[current_idx][j] = dp[current_idx - 1][j];
                } else {
                    dp[current_idx][j] = std::max(dp[current_idx - 1][j], 
                                                  dp[current_idx - 1][j - w] + v);
                }
            }
            current_idx++;
        }
    }

    return dp[total_items][W];
}
```

## 优化思路：二进制拆分

二进制拆分（Binary Splitting）的[核心思想](https://math.youkou.cc.cd/posts/proof-binary-integer-interval-coverage/)是：**将数量为 $c_i$ 的物品拆分为若干个“小包”，通过这些小包的组合，可以凑出 $0$ 到 $c_i$ 之间的任意数量。**

这样，我们不需要将 $c_i$ 个物品平铺为 $c_i$ 个独立的物品，而只需处理约 $\log_2(c_i)$ 个物品，从而将时间复杂度大幅优化。

### 为什么是二进制？

根据数学原理，任何一个正整数 $n$ 都可以由 $1, 2, 4, \dots, 2^k$ 以及一个剩余值 $R$ 组合而成。

例如，假设某种物品有 **13** 个：

- 我们将其拆分为：$1, 2, 4, 6$
- **组合验证：**
  - $1 = 1$
  - $2 = 2$
  - $3 = 1+2$
  - $4 = 4$
  - $5 = 1+4$
  - $6 = 2+4$
  - $7 = 1+2+4$
  - $8 = 2+6$ (或者其他组合)
  - ...以此类推，通过这些数字的不同相加，可以无重叠、无遗漏地表示 $[0, 13]$ 范围内的所有数量。

### 拆分规则

对于数量为 $c_i$ 的物品：

1. 从 $k=1$ 开始，依次取 $1, 2, 4, \dots, 2^p$ 个物品作为一个“新物品”。
2. 每次取出后，剩余数量 $c_i$ 减去取出的数量。
3. 如果剩余的数量小于下一个 $2^{p+1}$，则将剩下的所有数量作为最后一个“新物品”。

**拆分示例表 (以 $c_i=13$ 为例)：**

| **拆分组合** | **剩余数量**  | **备注**         |
| ------------ | ------------- | ---------------- |
| 取出 1       | $13 - 1 = 12$ | 组成第一个新物品 |
| 取出 2       | $12 - 2 = 10$ | 组成第二个新物品 |
| 取出 4       | $10 - 4 = 6$  | 组成第三个新物品 |
| 取出 6       | $6 - 6 = 0$   | 剩余部分直接打包 |

### 实现逻辑思路

1. **预处理**：遍历每一种物品 $i$，按照上述规则进行拆分。
2. **存入新列表**：创建一个新的物品列表 `new_items`，存储拆分后的 `{重量, 价值}` 对。
   - 拆分出来的第 $k$ 个物品：
     - `weight = original_weight * k`
     - `value = original_value * k`
3. **转化为 0/1 背包**：对 `new_items` 执行标准的 0/1 背包算法。由于拆分后的物品总数只有 $O(n \cdot \log c_i)$ 个，计算量显著下降。

### 代码实现（C++）

```c++
#include <vector>
#include <algorithm>

struct Item {
    int weight;
    int value;
};

/**
 * 二进制拆分优化多重背包 (非空间优化/二维 DP)
 * W: 背包容量
 * weights, values, counts: 数据均从下标 1 开始
 */
int multipleKnapsackBinary2D(int W, const std::vector<int>& weights, 
                             const std::vector<int>& values, const std::vector<int>& counts) {
    int n = weights.size() - 1;
    std::vector<Item> binaryItems;

    // 1. 二进制拆分：将物品拆分为 {weight, value} 的组合
    for (int i = 1; i <= n; ++i) {
        int w = weights[i];
        int v = values[i];
        int c = counts[i];
        for (int k = 1; c > 0; k <<= 1) {
            int num = std::min(k, c);
            binaryItems.push_back({w * num, v * num});
            c -= num;
        }
    }

    // 2. 二维 DP 状态转移
    // row 表示拆分后的物品索引，col 表示背包容量
    int total_binary_items = binaryItems.size();
    std::vector<std::vector<int>> dp(total_binary_items + 1, std::vector<int>(W + 1, 0));

    for (int i = 1; i <= total_binary_items; ++i) {
        int w = binaryItems[i - 1].weight;
        int v = binaryItems[i - 1].value;
        
        for (int j = 0; j <= W; ++j) {
            // 不选择该组合物品
            dp[i][j] = dp[i - 1][j];
            
            // 选择该组合物品 (前提是容量足够)
            if (j >= w) {
                dp[i][j] = std::max(dp[i][j], dp[i - 1][j - w] + v);
            }
        }
    }

    return dp[total_binary_items][W];
}
```

### 为什么要这样拆？

- **基础思路**：复杂度为 $O(W \cdot \sum c_i)$。如果 $c_i$ 很大（例如 $10^5$），该算法会超时。
- **二进制拆分**：复杂度为 $O(W \cdot \sum \log c_i)$。$\log$ 的增长速度极慢，即便是 $c_i=10^5$，$2^{16} < 10^5 < 2^{17}$，原本需要处理 10 万次的操作，现在只需处理 17 次左右，性能提升了几个数量级。

## 最优解：单调队列优化

在多重背包问题中，除了“转化为 0/1 背包（拆分法）”和“二维 DP 基础版”，最著名的第三种思路是：**使用单调队列进行优化**。

这是处理多重背包问题在时间复杂度上最优的方法，其复杂度可以达到 **$O(n \cdot W)$**，即与物品的个数和背包容量成线性关系。

### 多重背包的原始 DP 模型

设 $dp[i][j]$ 为在考虑前 $i$ 种物品且背包总容量限制为 $j$ 时，所能获取的最大总价值。其中，$i \in [0, n]$，$j \in [0, W]$。

对于第 $i$ 种物品，设其重量为 $w_i$、价值为 $v_i$、数量上限为 $c_i$。该物品的状态转移方程定义为：

$$dp[i][j] = \max_{0 \le x \le c_i} \{ dp[i-1][j - x \cdot w_i] + x \cdot v_i \}$$

该方程体现了多重背包问题的决策过程：为了求得 $dp[i][j]$，我们需在 $[0, c_i]$ 的范围内枚举第 $i$ 种物品的装入数量 $x$。对于每一种合法的选择 $x$，我们将其带来的收益 $x \cdot v_i$ 与剩余容量 $j - x \cdot w_i$ 下的前 $i-1$ 种物品的最优价值（即 $dp[i-1][j - x \cdot w_i]$）求和。由于各决策分支互斥，我们取所有可能中的最大值作为当前状态的最优解。

### 数学转换

#### 原始模型的同余分组

对于第 $i$ 种物品（重量 $w_i$，价值 $v_i$，上限 $c_i$），状态转移方程为：

$$dp[i][j] = \max_{0 \le x \le c_i} \{ dp[i-1][j - x \cdot w_i] + x \cdot v_i \}$$

我们观察到，无论 $j$ 是多少，只要 $j - x \cdot w_i$ 减去 $w_i$ 的倍数，其模 $w_i$ 的余数都是固定的。因此，我们可以按照余数 $r$ ($0 \le r < w_i$) 将所有容量 $j$ 分组：

$$j = k \cdot w_i + r$$

其中 $k = \lfloor j / w_i \rfloor$ 代表“容纳了 $w_i$ 的倍数”。

#### 为什么要定义 $p = k - x$？

当我们固定余数 $r$ 后，对于同一个余数类，所有容量的状态都排成了一条直线：

- $k=0$ 时：容量为 $r$
- $k=1$ 时：容量为 $w_i + r$
- $k=2$ 时：容量为 $2w_i + r$
- ...以此类推。

当我们想计算容量为 $j = k \cdot w_i + r$ 的状态时，我们看的是“上一轮”第 $i-1$ 种物品在不同容量下的值。方程中的 $j - x \cdot w_i$ 可以写成：

$$(k \cdot w_i + r) - x \cdot w_i = (k - x) \cdot w_i + r$$

在这里，**$p = k - x$ 实际上就是上一轮状态在该余数序列中的“倍数坐标”**。

- 如果 $x=0$（选 0 个），我们要看的状态就是 $dp[i-1][k \cdot w_i + r]$，即 $p=k$。
- 如果 $x=1$（选 1 个），我们要看的状态就是 $dp[i-1][(k-1) \cdot w_i + r]$，即 $p=k-1$。
- 以此类推，选 $x$ 个，我们要看的状态就是 $p=k-x$。

所以 $p$ 代表的是在同一条余数直线上，**前一轮状态的“位置索引”**。

#### 方程的深度推导

现在我们将 $p = k - x$ 代入方程。由于 $0 \le x \le c_i$，且 $p = k - x$，通过移项得到 $x = k - p$。此时 $x$ 的约束条件变为：

$$0 \le k - p \le c_i \implies k - c_i \le p \le k$$

代入原始方程：

$$dp[i][k \cdot w_i + r] = \max_{k-c_i \le p \le k} \{ dp[i-1][p \cdot w_i + r] + (k - p) \cdot v_i \}$$

为了使用单调队列，我们要将只包含 $p$ 的项留在 $\max$ 内部。我们将 $(k-p) \cdot v_i$ 拆开：

$$dp[i][k \cdot w_i + r] = \max_{k-c_i \le p \le k} \{ dp[i-1][p \cdot w_i + r] + k \cdot v_i - p \cdot v_i \}$$

由于 $k \cdot v_i$ 与 $p$ 无关，我们可以将其提取到 $\max$ 之外：

$$dp[i][k \cdot w_i + r] = \left( \max_{k-c_i \le p \le k} \{ dp[i-1][p \cdot w_i + r] - p \cdot v_i \} \right) + k \cdot v_i$$

### 为什么它是“滑动窗口”？

#### 二维表的结构化观察

我们将 $dp[i-1]$ 这一行按照模 $w_i$ 的余数 $r$ 进行“拆解”。你会发现，原本连续的一行数据，被切割成了 $w_i$ 条独立的“余数链”：

- **余数链 0**：$dp[i-1][0], dp[i-1][w_i], dp[i-1][2w_i], \dots$
- **余数链 1**：$dp[i-1][1], dp[i-1][w_i+1], dp[i-1][2w_i+1], \dots$
- **余数链 $r$**：$dp[i-1][r], dp[i-1][w_i+r], dp[i-1][2w_i+r], \dots$

在二维表中，这就像是把原本平铺的数据，按照一定的步长（$w_i$）抽取出来，排成了一列列“队列”。

#### 在二维视角下的“窗口滑动”

当我们计算第 $i$ 行某个位置 $j = k \cdot w_i + r$ 的值时，方程告诉我们：

$$dp[i][k \cdot w_i + r] = \max_{k-c_i \le p \le k} \{ dp[i-1][p \cdot w_i + r] - p \cdot v_i \} + k \cdot v_i$$

在二维表上，这意味着：

1. **定位**：我们锁定在第 $i-1$ 行的“余数链 $r$”上。
2. **划窗**：以当前倍数 $k$ 为终点，向左回溯 $c_i$ 个位置，形成一个长度为 $c_i+1$ 的**滑动窗口**。
3. **提取**：在这一行对应的“余数链”上，提取 $dp[i-1][p \cdot w_i + r] - p \cdot v_i$ 的最大值。

随着我们在二维表上从左向右移动（$k$ 增大），这个窗口在“余数链 $r$”上同步向右移动。

### 空间极致优化：从二维到一维

在二维 DP 模型中，我们需要保存 $dp[n+1][W+1]$。然而，观察状态转移方程：

$$dp[i][j] = \max_{0 \le x \le c_i} \{ dp[i-1][j - x \cdot w_i] + x \cdot v_i \}$$

对于当前物品 $i$ 的每一个容量 $j$，它**仅仅依赖于上一层 $i-1$ 的状态**。这意味着：在计算完第 $i$ 行后，第 $i-1$ 行的数据对于计算第 $i+1$ 行不再有任何贡献。

#### 滚动数组的数学降维

既然我们只需要“上一行”的信息，我们可以引入**滚动数组**技术，将空间复杂度从 $O(nW)$ 直接降至 $O(W)$。

- **逻辑映射**：我们将二维数组的第 $i$ 行映射为当前数组 $dp$，将第 $i-1$ 行映射为前序状态。
- **物理实现**：在单调队列优化中，由于我们需要保证在同一轮更新时，队列读取的始终是上一轮的“旧值”，我们通常通过以下方式实现：
  - **方式 A（备份法）**：在每一轮物品决策前，将当前的 $dp$ 数组完整拷贝一份作为 `prev_dp`。
  - **方式 B（状态分离）**：单调队列本身就是一种特殊的缓冲区，它存储了上一轮经过数学变换后的最优值，从而解耦了读写操作。

#### 从二维方程到一维的演变

通过上述优化，我们方程中的 $i$ 维度被彻底“摊平”：

1. **原始二维方程**：

   $$dp[i][k \cdot w_i + r] = \left( \max_{k-c_i \le p \le k} \{ dp[i-1][p \cdot w_i + r] - p \cdot v_i \} \right) + k \cdot v_i$$

2. **一维滚动方程**：

   $$dp[k \cdot w_i + r] = \left( \max_{k-c_i \le p \le k} \{ prev\_dp[p \cdot w_i + r] - p \cdot v_i \} \right) + k \cdot v_i$$

在这个方程中，下标映射完全保持不变，但内存布局从二维矩阵变成了两个一维向量：`dp`（写目标）和 `prev_dp`（读源）。

### 代码实现（C++）

```c++
#include <vector>
#include <deque>
#include <algorithm>

using namespace std;

/**
 * 多重背包单调队列优化实现
 * 时间复杂度: O(nW)
 * 空间复杂度: O(W)
 */
int solveMultipleKnapsack(int W, int n, const vector<int>& weights, 
                          const vector<int>& values, const vector<int>& counts) {
    // dp[j] 表示容量为 j 时的最大价值
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; ++i) {
        int w = weights[i];
        int v = values[i];
        int c = counts[i];
        
        // 为了实现滚动数组，在这一轮物品决策前拷贝上一轮的状态
        vector<int> prev_dp = dp;

        // 按照余数 r 进行分组 (0 <= r < w)
        for (int r = 0; r < w; ++r) {
            deque<int> dq; // 存储的是“倍数” k
            
            // 在当前余数 r 的直线上滑动窗口
            for (int k = 0; k * w + r <= W; ++k) {
                // 计算当前位置的值：val = prev_dp[k*w+r] - k*v
                // 这是我们要维护的滑动窗口序列中的“原始数据”
                int val = prev_dp[k * w + r] - k * v;
                
                // 1. 维护单调队列：保持队首元素对应的 val 最大
                while (!dq.empty() && prev_dp[dq.back() * w + r] - dq.back() * v <= val) {
                    dq.pop_back();
                }
                dq.push_back(k);
                
                // 2. 移除超出窗口范围的过期下标 (k - p > c)
                if (dq.front() < k - c) {
                    dq.pop_front();
                }
                
                // 3. 更新 dp[j]，j = k * w + r
                // 当前 dp[j] = (max_val_in_window) + k * v
                dp[k * w + r] = prev_dp[dq.front() * w + r] + (k - dq.front()) * v;
            }
        }
    }
    return dp[W];
}
```

