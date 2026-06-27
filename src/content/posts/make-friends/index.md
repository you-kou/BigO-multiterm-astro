---
title: '交朋友'
published: 2026-03-15
draft: false
tags: ['最优解维护']
toc: true
---

> Alice 班上共有 4 个小朋友，身高分别为 $H_1$, $H_2$, $H_3$, $H_4$，其中 Alice 的身高为 $H_1$。Alice 想要和身高最接近她的人交朋友，如果有多个人符合条件，则 Alice 想和其中较矮的那一人做朋友，你能告诉她这个人的身高是多少吗？
>
> **输入格式**
>
> 输入共 4 行，第 $i$ 行包含一个整数 $H_i$，表示班上小朋友的身高。
>
> **输出格式**
>
> 输出 1 行，包含一个整数 $h$，表示 Alice 想交的朋友的身高。
>
> **样例**
>
> **输入样例**
>
> ```
> 150
> 165
> 135
> 133
> ```
>
> **输出样例**
>
> ```
> 135
> ```
>
> **样例解释**
>
> 样例 1 中，Alice 身高为 $150$，第 2、3 个小朋友与 Alice 身高差距为 $15$，同样最接近，Alice 选较矮的一个即第 $3$ 个身高为 $135$ 的小朋友交朋友。
>
> **数据范围**
>
> 保证 $100 \leq H_i \leq 199$ 且 $H_i$ 互不相同。

由于数据量极小（总共只有 4 个数），我们可以直接用一个循环或简单的条件判断来维护最优解：

1. 先读入 Alice 的身高 `alice_h`。
2. 初始化两个变量：`min_diff`（记录最小身高差，初始设为无穷大）和 `ans_h`（记录最终答案的身高）。
3. 循环读入其余 3 个人的身高 `h`：
  - 计算当前身高与 Alice 的差距：`diff = abs(h - alice_h)`。
  - **更新条件**：
    - 如果 `diff < min_diff`：说明找到了更接近的身高，直接更新 `min_diff` 和 `ans_h`。
    - 如果 `diff == min_diff`：说明距离一样，根据题意需要选择较矮的，所以如果 `h < ans_h`，则更新 `ans_h`。

## 代码实现 (C++)

```c++
#include <iostream>
#include <cmath>   // 用于 std::abs
#include <climits> // 用于 INT_MAX

int main() {
    // 提高输入输出效率
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int alice_h;
    if (!(std::cin >> alice_h)) return 0;

    int min_diff = INT_MAX; // 初始化最小差值为一个很大的数
    int ans_h = 0;          // 记录最终的目标身高

    // 循环读入剩下的 3 个小朋友的身高
    for (int i = 0; i < 3; ++i) {
        int h;
        std::cin >> h;
        
        int current_diff = std::abs(h - alice_h);

        // 如果当前的差值更小，或者差值相等但当前小朋友更矮
        if (current_diff < min_diff) {
            min_diff = current_diff;
            ans_h = h;
        } else if (current_diff == min_diff) {
            if (h < ans_h) {
                ans_h = h;
            }
        }
    }

    // 输出最终找到的朋友身高
    std::cout << ans_h << "\n";

    return 0;
}
```

