---
title: '完全背包问题'
published: 1975-01-01
draft: false
tags: ['动态规划', '完全背包问题']
toc: true
---

> 假设你有一个容量为 $W$ 的背包，以及 $n$ 种物品。每种物品 $i$ 都有一个重量 $w_i$ 和一个价值 $v_i$。
>
> 你的目标是在不超过背包总容量的前提下，选择物品使得背包内物品的总价值最大。

与“0/1背包问题”不同，在完全背包中，**每种物品的数量都是无限的**。你可以根据需要，对同一种物品选择零次、一次或多次放入背包。

## 动态规划

### 状态定义

定义 $dp[i][j]$ 为：**仅考虑前 $i$ 种物品，且背包最大容量为 $j$ 时，所能获取的最大价值。**

- **$i$**：代表你当前拥有的“物品种类池”。当 $i=3$ 时，意味着你只能从第 1, 2, 3 种物品中选择。
- **$j$**：代表你的“背包容量约束”。无论你选了多少物品，总重量不能超过这个值。
- **$dp[i][j]$**：代表在上述两个约束下，你能达到的最大价值。

### 状态转移

当你站在 $dp[i][j]$ 这个位置时，你只有两种决策：

1. **不选第 $i$ 种物品**：

   你当前能获得的最大价值，等于“只用前 $i-1$ 种物品，且容量为 $j$”时的最大价值。

   即：**$dp[i-1][j]$**

2. **选第 $i$ 种物品**：

   既然选了，你就要占用 $w_i$ 的重量并获得 $v_i$ 的价值。剩余容量为 $j - w_i$。

   此时，因为物品是“完全”的（无限供应），你依然可以在“前 $i$ 种物品”中继续挑选。

   即：**$dp[i][j - w_i] + v_i$**

结合上述决策，为了获得最大价值，我们取两者的最大值：

$$dp[i][j] = \max(dp[i-1][j], dp[i][j - w_i] + v_i)$$

### 边界条件

- 当没有物品可选时（$i=0$），价值全为 0：`dp[0][j] = 0`。
- 当背包容量为 0 时（$j=0$），价值全为 0：`dp[i][0] = 0`。

## 代码实现（C++）

```c++
#include <vector>
#include <algorithm>

/**
 * 完全背包问题二维 DP 实现（假设输入数组从下标 1 开始记录数据）
 * @param W 背包最大容量
 * @param weights 物品重量数组（下标 1 到 n 有效）
 * @param values 物品价值数组（下标 1 到 n 有效）
 * @return 最大价值
 */
int unboundedKnapsack(int W, const std::vector<int>& weights, const std::vector<int>& values) {
    int n = weights.size() - 1; // 有效物品数量
    
    // 初始化 dp[n+1][W+1]
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(W + 1, 0));

    // 逐行填充
    for (int i = 1; i <= n; ++i) {
        int w = weights[i]; // 直接使用下标 i
        int v = values[i];  // 直接使用下标 i
        
        for (int j = 0; j <= W; ++j) {
            // 不选第 i 种物品
            dp[i][j] = dp[i - 1][j];
            
            // 选第 i 种物品（完全背包：引用当前行 dp[i]）
            if (j >= w) {
                dp[i][j] = std::max(dp[i][j], dp[i][j - w] + v);
            }
        }
    }

    return dp[n][W];
}
```

## 空间优化

在二维公式 $dp[i][j] = \max(dp[i-1][j], dp[i][j - w_i] + v_i)$ 中：

- $dp[i-1][j]$ 是“上一行”的数据。
- $dp[i][j - w_i]$ 是“当前行（即本次更新后）”的数据。

如果我们使用一维数组 `dp[j]`，当我们更新 `dp[j]` 时，`dp[j - w_i]` 已经是在当前这一轮循环中被更新过的值，这恰好满足了完全背包的需求。因此，我们只需要**正序遍历容量**即可。

```c++
#include <vector>
#include <algorithm>

/**
 * 完全背包问题一维空间优化实现
 * @param W 背包最大容量
 * @param weights 物品重量数组（下标 1 到 n 有效）
 * @param values 物品价值数组（下标 1 到 n 有效）
 * @return 最大价值
 */
int unboundedKnapsack(int W, const std::vector<int>& weights, const std::vector<int>& values) {
    int n = weights.size() - 1; // 有效物品数量
    
    // 初始化一维数组 dp[W+1]，dp[j] 表示容量为 j 时的最大价值
    std::vector<int> dp(W + 1, 0);

    // 外层循环：遍历物品
    for (int i = 1; i <= n; ++i) {
        int w = weights[i];
        int v = values[i];
        
        // 内层循环：正序遍历容量
        // 因为 dp[j - w] 已经是当前轮次更新过的值，直接利用它即实现了重复选取的逻辑
        for (int j = w; j <= W; ++j) {
            dp[j] = std::max(dp[j], dp[j - w] + v);
        }
    }

    return dp[W];
}
```

**为什么正序能“无限叠加”？**

当我们执行 `dp[j] = max(dp[j], dp[j - w_i] + v_i)` 时：

1. **当计算 `dp[j]` 时**：`j - w_i` 必然小于 `j`。
2. **正序的推导路径**：
  - 因为我们是从小到大遍历，计算 `dp[j]` 时，`dp[j - w_i]` 这一项**已经被更新过了**。
  - 这意味着 `dp[j - w_i]` 代表的是“考虑过第 $i$ 种物品后的状态”。
  - 因此，`dp[j - w_i] + v_i` 实际上是在“已经考虑过一个 $i$”的基础上，**再尝试放入一个 $i$**。
  - 这种逻辑允许物品被一次又一次地叠加，直到容量 $W$ 被填满。