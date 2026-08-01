---
title: '二分查找'
published: 1962-01-01
draft: false
tags: ['分治']
toc: true
coverImage:
  src: './binary-search.jpg'
  alt: '二分算法查找步骤示意图，有序数组展示 low、mid、high 指针三轮迭代变化'
---

**二分查找（Binary Search）**，也称折半查找，是一种在**有序数组**中查找特定元素的极高效算法。

它的核心思想是**分治法（Divide and Conquer）**：每次将查找区间一分为二，通过比较中间元素与目标值的大小，直接排除掉不符合条件的一半数据，从而快速缩小搜索范围。

## 核心原理解析

类似于在字典中查单词，或者玩“猜数字”游戏（1 到 100 猜一个数，每次告诉你猜大了还是猜小了）：

1. **设定指针**：定义当前搜索区间的左边界 `left` 和右边界 `right`。
2. **取中间值**：计算中间位置 `mid`，比较 `array[mid]` 与目标值 `target`：
  - 若 `array[mid] == target`：**找到目标**，返回索引。
  - 若 `array[mid] < target`：说明目标在右半区，将左边界更新为 `left = mid + 1`。
  - 若 `array[mid] > target`：说明目标在左半区，将右边界更新为 `right = mid - 1`。
3. **循环迭代**：重复上述过程，直到找到目标值；若 `left > right` 仍未找到，说明目标不在数组中。

## 代码实现 (C++)

```c++
#include <iostream>
#include <vector>

/**
 * 基础二分查找（迭代版）
 * 
 * @param nums   有序递增数组
 * @param target 目标值
 * @return int   目标值的下标；若不存在则返回 -1
 */
int binarySearch(const std::vector<int>& nums, int target) {
    int left = 0;
    int right = nums.size() - 1; // 1. 采用左闭右闭区间 [left, right]

    while (left <= right) {      // 2. 当 left == right 时，区间 [left, left] 依然有效
        // 3. 防溢出计算 mid，等价于 (left + right) / 2
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) {
            return mid;          // 找到目标，直接返回索引
        } else if (nums[mid] < target) {
            left = mid + 1;      // target 在右侧，缩小左边界
        } else {
            right = mid - 1;     // target 在左侧，缩小右边界
        }
    }

    return -1; // 搜索结束仍未找到
}
```

### 关键细节与避坑指南

1. **防溢出计算**：直接写 `(left + right) // 2` 在极大数值时可能导致整型溢出，推荐写成 `left + (right - left) // 2`。
2. **边界条件（`<=` 与 `<`）**：
  - 若初始化 `right = len - 1`，搜索区间是左闭右闭 `[left, right]`，循环条件应为 `while left <= right`。
  - 若初始化 `right = len`，搜索区间是左闭右开 `[left, right)`，循环条件应为 `while left < right`，边界更新也需相应微调。

## 性能与复杂度

| **指标**       | **复杂度**           | **说明**                                             |
| -------------- | -------------------- | ---------------------------------------------------- |
| **时间复杂度** | $O(\log n)$          | 每次缩小一半范围，$100$ 万数据最多只需约 $20$ 次比较 |
| **空间复杂度** | $O(1)$               | 迭代实现仅需常数级别的额外空间                       |
| **必备前提**   | **数组必须是有序的** | 且需支持随机访问（如数组，链表则不适用）             |
