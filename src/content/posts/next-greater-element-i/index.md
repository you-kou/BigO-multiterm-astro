---
title: '下一个更大元素 I'
published: 2017-02-12
draft: false
tags: ['算法题', '单调栈', '哈希表']
toc: true
coverImage:
  src: './Next-Greater-Element-I.png'
  alt: 'LeetCode 496 下一个更大元素 I 的算法逻辑演示图，展示了遍历数组 nums2 并填充结果数组的过程'
---

> `nums1` 中数字 `x` 的 **下一个更大元素** 是指 `x` 在 `nums2` 中对应位置 **右侧** 的 **第一个** 比 `x` 大的元素。
>
> 给你两个 **没有重复元素** 的数组 `nums1` 和 `nums2` ，下标从 **0** 开始计数，其中`nums1` 是 `nums2` 的子集。
>
> 对于每个 `0 <= i < nums1.length` ，找出满足 `nums1[i] == nums2[j]` 的下标 `j` ，并且在 `nums2` 确定 `nums2[j]` 的 **下一个更大元素** 。如果不存在下一个更大元素，那么本次查询的答案是 `-1` 。
>
> 返回一个长度为 `nums1.length` 的数组 `ans` 作为答案，满足 `ans[i]` 是如上所述的 **下一个更大元素** 。
>
>  
>
> **示例 1：**
>
> ```
> 输入：nums1 = [4,1,2], nums2 = [1,3,4,2].
> 输出：[-1,3,-1]
> 解释：nums1 中每个值的下一个更大元素如下所述：
> - 4 ，用加粗斜体标识，nums2 = [1,3,4,2]。不存在下一个更大元素，所以答案是 -1 。
> - 1 ，用加粗斜体标识，nums2 = [1,3,4,2]。下一个更大元素是 3 。
> - 2 ，用加粗斜体标识，nums2 = [1,3,4,2]。不存在下一个更大元素，所以答案是 -1 。
> ```
>
> **示例 2：**
>
> ```
> 输入：nums1 = [2,4], nums2 = [1,2,3,4].
> 输出：[3,-1]
> 解释：nums1 中每个值的下一个更大元素如下所述：
> - 2 ，用加粗斜体标识，nums2 = [1,2,3,4]。下一个更大元素是 3 。
> - 4 ，用加粗斜体标识，nums2 = [1,2,3,4]。不存在下一个更大元素，所以答案是 -1 。
> ```
>
>  
>
> **提示：**
>
> - `1 <= nums1.length <= nums2.length <= 1000`
> - `0 <= nums1[i], nums2[i] <= 104`
> - `nums1`和`nums2`中所有整数 **互不相同**
> - `nums1` 中的所有整数同样出现在 `nums2` 中

## 解题思路：单调栈

虽然可以使用暴力法（双重循环）解决，但使用[**单调栈**](monotonic-stack)可以将时间复杂度优化到 $O(N+M)$（$N$ 和 $M$ 分别为两个数组的长度）。

**第一步：预处理 `nums2`（构建哈希表）**

通过单调递减栈，我们可以在一次遍历中确定 `nums2` 中所有元素的“下一个更大元素”。

- **创建单调栈：** 使用单调递减栈辅助遍历 `nums2`。
- **判定逻辑：** 当遍历到的数字大于栈顶元素时，说明该数字就是栈顶的“下一个更大元素”。
- **记录结果：** 将这一映射关系存入哈希表：`Map[被弹出的元素] = 当前数字`。
- **善后处理：** 遍历结束后，对于栈中剩余的元素，将其在哈希表中的值置为 `-1`。

**第二步：映射 `nums1` 的结果**

完成预处理后，我们可以通过哈希表实现 $O(1)$ 的快速查询。

- **查询映射：** 遍历 `nums1` 中的每个元素。
- **生成数组：** 利用预处理好的哈希表，直接获取每个数字对应的“下一个更大元素”，并填入结果数组即可。

## 代码实现 (C++)

```c++
#include <vector>
#include <stack>
#include <unordered_map>

class Solution {
public:
    std::vector<int> nextGreaterElement(std::vector<int>& nums1, std::vector<int>& nums2) {
        // 使用哈希表存储 nums2 中每个元素的“下一个更大元素”
        std::unordered_map<int, int> next_greater;
        std::stack<int> s;

        // 遍历 nums2 构建单调递减栈
        for (int num : nums2) {
            // 当栈不为空，且当前元素大于栈顶元素时，
            // 说明找到了栈顶元素的“下一个更大元素”
            while (!s.empty() && num > s.top()) {
                next_greater[s.top()] = num;
                s.pop();
            }
            s.push(num);
        }

        // 构建结果数组
        std::vector<int> result;
        for (int num : nums1) {
            // 如果在 map 中找到了对应的映射，则取值；否则置为 -1
            if (next_greater.count(num)) {
                result.push_back(next_greater[num]);
            } else {
                result.push_back(-1);
            }
        }

        return result;
    }
};
```

## 复杂度分析

- **时间复杂度：** $O(N + M)$。其中 $N$ 是 `nums1` 的长度，$M$ 是 `nums2` 的长度。虽然嵌套了 `while` 循环，但每个元素在整个过程中只进栈一次、出栈一次。
- **空间复杂度：** $O(M)$。主要用于哈希表存储 `nums2` 的映射关系。