---
title: '穷举'
published: 1936-11-30 00:00:01
draft: false
tags: ['算法设计', '查找问题', '最值问题', '满足下限的最小配对和', '部分和问题']
toc: true
coverImage:
  src: './brute_force_attack.webp'
  alt: '攻击者通过程序逐个尝试全部数字密码组合，直至破解密码、突破安全防护。'
---

**穷举搜索**（又称**暴力搜索**，Brute Force Search）是一种最直接、最原始的计算机求解策略。其核心在于**毫无遗漏地列举并检验**解空间中的每一个候选解，直到找到正确解或最优解，本质上是以计算机的强大算力来弥补算法策略上的不足。

## 线性穷举

在**线性穷举**中，解空间表现为一维数组或容器（时间复杂度通常为 $O(n)$）。其求解核心非常直观：**使用单层循环遍历每一个元素，逐个进行条件检验。**

### 满足条件的元素所在位置

- **问题描述：** 在一个无序数组中，找出值为 `target` 的元素所在的位置（下标）。如果不存在，返回 `-1`。
- **穷举策略：** 从索引 `0` 开始顺次检查每一个元素，匹配成功立即返回当前下标；若查完整个数组都没找到，返回 `-1`。

```c++
#include <iostream>
#include <vector>

// 线性穷举查找元素位置
int linearSearch(const std::vector<int>& arr, int target) {
    for (std::size_t i = 0; i < arr.size(); ++i) {
        if (arr[i] == target) {
            return static_cast<int>(i); // 穷举找到，返回索引
        }
    }
    return -1; // 遍历完毕，未找到
}
```

### 寻找最小值

- **问题描述：** 给定一个包含 $n$ 个数字的数组，找出其中的最小值。
- **穷举策略：** 假设第一个数字是当前最小值；随后**穷举遍历数组中的所有后续元素**，一旦发现更小的数，就更新当前最小值。

```c++
#include <iostream>
#include <vector>

// 线性穷举查找最小值
int findMin(const std::vector<int>& arr) {
    if (arr.empty()) return -1;
    
    int minVal = arr[0];
    // 使用 C++11 基于范围的 for 循环 (Range-based for loop)
    for (const auto& val : arr) {
        if (val < minVal) {
            minVal = val; // 更新最小值
        }
    }
    return minVal;
}
```

## 成对/多元穷举

在**成对/多元穷举**中，问题往往涉及两个或多个变量的组合关系，解空间规模随之扩大为平方级 $O(n^2)$ 或多项式级 $O(n^k)$。它的求解核心在于**使用多层嵌套循环，穷举并尝试所有可能的变量配对或多元组合。**

### 在大于或等于 $k$ 的配对和中找到最小值

- **问题描述：** 给定一个非负整数数组 `arr` 和一个阈值 $k$。从数组中挑选两个不同位置的元素配对（$i \neq j$），要求它们的和 $arr[i] + arr[j] \ge k$，并求出所有满足条件的配对和中的**最小值**。
- **穷举策略：**
  1. 使用双重循环遍历所有可能的索引对 $(i, j)$，其中 $i < j$（避免重复比较与自配对）。
  2. 计算每对元素的和 $pairSum = arr[i] + arr[j]$。
  3. 检验条件 $pairSum \ge k$，在所有满足条件的配对和中保留并更新最小值。

```c++
#include <iostream>
#include <vector>
#include <limits>
#include <algorithm>

int minPairSumAtLeastK(const std::vector<int>& arr, int k) {
    std::size_t n = arr.size();
    // 使用 C++11 <limits> 标头获取整型最大值
    int minSum = std::numeric_limits<int>::max();
    bool found = false;

    // 穷举所有成对的可能性 (i, j)
    for (std::size_t i = 0; i < n; ++i) {
        for (std::size_t j = i + 1; j < n; ++j) {
            int pairSum = arr[i] + arr[j];
            // 检验约束条件，并更新最优解
            if (pairSum >= k) {
                minSum = std::min(minSum, pairSum);
                found = true;
            }
        }
    }
    return found ? minSum : -1;
}
```

## 组合穷举

在**组合穷举**中，问题的搜索空间由集合的子集、排列或组合构成，解空间规模呈现出指数级 $O(2^n)$ 或阶乘级 $O(n!)$ 的急剧增长。它的求解核心在于**利用位运算或递归/回溯机制，无遗漏地尝试所有可能的集合状态。**

### 部分和问题

- **问题描述：** 给定一个包含 $n$ 个整数的集合 `arr` 和一个目标值 `target`。判断是否存在一个**子集**，使其所有元素的和恰好等于 `target`。
- **穷举策略：** 含有 $n$ 个元素的集合共有 $2^n$ 个子集，穷举这 $2^n$ 种解空间状态并验证其和。

#### 方法 1：二进制位掩码穷举

利用整数的二进制位表示子集（第 $i$ 位为 `1` 代表选择 `arr[i]`，为 `0` 代表不选）。$0 \sim 2^n - 1$ 的每个整数唯一对应一种子集状态。

```c++
#include <iostream>
#include <vector>

bool subsetSumBitmask(const std::vector<int>& arr, int target) {
    std::size_t n = arr.size();
    // C++11 允许使用 1ULL << n 防止 n 较大时发生 32 位整型溢出
    unsigned long long totalSubsets = 1ULL << n; 

    // 从 0 到 2^n - 1 穷举每一种子集状态
    for (unsigned long long mask = 0; mask < totalSubsets; ++mask) {
        int currentSum = 0;
        for (std::size_t i = 0; i < n; ++i) {
            // 判断第 i 个元素是否在当前子集中（第 i 位是否为 1）
            if ((mask >> i) & 1) {
                currentSum += arr[i];
            }
        }
        // 检验条件
        if (currentSum == target) {
            return true;
        }
    }
    return false;
}
```

#### 方法 2：递归 DFS 回溯展开解空间

利用递归树展开全解空间，在递归边界做条件检验。代码中使用 C++11 的 **Lambda 闭包** 进行递归实现。

```c++
#include <iostream>
#include <vector>
#include <functional>

bool subsetSumRecursive(const std::vector<int>& arr, int target) {
    // 使用 std::function 定义递归 Lambda 表达式
    std::function<bool(std::size_t, int)> dfs = [&](std::size_t index, int currentSum) -> bool {
        // 递归终止条件：已考虑完所有元素
        if (index == arr.size()) {
            return currentSum == target;
        }

        // 穷举分支 1：选当前元素
        bool choose = dfs(index + 1, currentSum + arr[index]);
        if (choose) return true;

        // 穷举分支 2：不选当前元素
        bool ignore = dfs(index + 1, currentSum);
        return ignore;
    };

    return dfs(0, 0); // 从第 0 个元素、当前和为 0 开始递归
}
```

## 穷举搜索的优缺点

### 优点

1. **绝对的完备性：** 只要问题有解且解空间有限，就**一定会找到解**（不会漏解）。
2. **算法实现极简：** 无需复杂的数学推导，逻辑简单直白，不易写出逻辑漏洞。
3. **适用于无规律问题：** 对于没有任何特殊数学结构可利用的问题（如密码破解），穷举是唯一的办法。

### 缺点

1. **计算爆炸：** 随着输入规模 $N$ 的增长，候选解呈指数级（$2^n$）或阶乘级（$n!$）激增，导致计算耗时不可接受。
2. **资源消耗大：** 盲目搜索会执行大量无效计算。

## 学习穷举搜索的实际意义

1. **算法设计的起点与“基准线（Baseline）”**

  - 穷举是解决新问题时最直观的保底解法。
  - 在算法竞赛和工程中，常写一个绝对正确的穷举算法用于“对拍”，验证高级高效算法（如动态规划）的准确性。

2. **建立对“复杂度”与“问题空间”的直觉**

  - 帮助开发者建立对数据规模（$N \le 20$ 还是 $N \le 1000$）的敏感度，明确性能瓶颈所在的维度。

3. **掌握高级优化算法（剪枝、DP）的基石**

  - 所有“聪明”的算法本质都是“策略性穷举”。例如：

    $$\text{纯粹穷举} \xrightarrow{+ \text{递归回退}} \text{回溯法} \xrightarrow{+ \text{提前止损}} \text{剪枝} \xrightarrow{+ \text{记忆化重叠子问题}} \text{动态规划}$$

4. **工程实践中的“性价比”选择**

  - 当数据规模极小（例如只有几十个可能）时，写一个简单的穷举既省时又易于后人维护，避免**过度设计（Over-engineering）**。

5. **解决 NP 困难问题的终极保底手段**

  - 对于无法在多项式时间内求解的极端 NP-Hard 问题，在追求绝对精确解时，优化后的穷举（如分支限界）是唯一的选择。