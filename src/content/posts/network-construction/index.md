---
title: '线网建设'
published: 2026-06-29
draft: false
tags: ['算法题', '最小生成树', 'Prim 算法']
toc: true
---

> A 市有 $n$ 座基站需要通过线网互相连接。第 $i$ 座基站位于二维平面上坐标 $(x_i, y_i)$ 处。
>
> 第 $i$ 座基站与第 $j$ 座基站之间的距离定义为 $\sqrt{(x_i - x_j)^2 + (y_i - y_j)^2}$。
>
> 如果两座基站之间的距离不超过给定的整数 $l$，那么可以修建连接这两座基站的线路，线路长度为基站间的距离。
>
> 如果从一座基站出发，经过一系列线网中的线路可以到达另一座基站，则称这两座基站是互相连接的。
>
> 请问使得 $n$ 座基站两两之间都互相连接，需要修建的线路总长度最小是多少？如果不能修建满足条件的线网，则输出 `Impossible`。
>
> **输入格式**
>
> 第一行，两个正整数 $n, l$，分别表示基站数量与线路长度上限。
>
> 接下来 $n$ 行，每行两个整数 $x_i, y_i$，表示基站的坐标。
>
> **输出格式**
>
> 输出一行。如果能修建满足条件的线网，则输出需要修建的最小线路总长度，保留两位小数。否则输出 `Impossible`。
>
> **样例**
>
> **输入样例 1**
>
> ```
> 4 2
> 1 0
> -1 -1
> 0 0
> 1 1
> ```
>
> **输出样例 1**
>
> ```
> 3.41
> ```
>
> **输入样例 2**
>
> ```
> 4 1
> 1 0
> -1 -1
> 0 0
> 1 1
> ```
>
> **输出样例 2**
>
> ```
> Impossible
> ```
>
> **数据范围**
>
> 对于 $40\%$ 的测试点，保证 $1 \le n \le 100$。
>
> 对于所有测试点，保证 $1 \le n \le 500$，$1 \le l \le 100$，$-100 \le x_i, y_i \le 100$。

结合题目背景，这道题的本质是一个经典的最小生成树（Minimum Spanning Tree, MST）问题。由于要求让所有基站两两之间都互相连接，且使得需要修建的线路总长度最小，这正好对应了图论中最小生成树的概念。

## 题意分析

### 图的构建

- 每一个基站是一个**顶点（Vertex）**，总共有 $n$ 个顶点。
- 任意两个基站 $i$ 和 $j$ 之间如果满足欧几里得距离 $\le l$，则它们之间存在一条**边（Edge）**，边的权重（Cost）就是它们之间的距离：$\sqrt{(x_i-x_j)^2 + (y_i-y_j)^2}$。如果距离 $> l$，则这两点之间无法直接建连。

### 目标

- 找出一个边的集合，使得所有顶点连通，且这些边的权重之和最小。
- 如果最后无法让所有顶点连通（即连通分量个数大于 1），则输出 `Impossible`。

由于数据范围 $n \le 500$，属于**稠密图**（边数可能达到 $n(n-1)/2 \approx 125,000$）。对于稠密图的最小生成树，使用 **Prim 算法**（时间复杂度 $O(n^2)$）是非常高效且易于实现的。

## 算法设计 (Prim 算法)

1. **初始化**：
   - 用一个数组 `dist` 记录每个点到当前已构建的生成树的最短距离。初始时，所有点的 `dist` 设为无穷大（$\infty$），`dist[1] = 0` 表示从 1 号基站开始构建。
   - 用一个布尔数组 `visited` 标记节点是否已经加入了生成树。
2. **循环 $n$ 次**：
   - 在所有还未加入生成树的点中，找出 `dist` 最小的点 $u$。
   - 如果在某一步找到的最小 `dist` 仍然是无穷大，说明图是不连通的，直接输出 `Impossible`。
   - 将 $u$ 标记为已访问（加入生成树），并将它的 `dist` 累加到总长度 `ans` 中。
   - 遍历所有还未加入生成树的点 $v$，如果 $u$ 和 $v$ 之间的距离 $\le l$，则尝试用它们之间的距离去更新 `dist[v]`。
3. **输出**：
   - 成功循环 $n$ 次后，输出 `ans`，保留两位小数。

## 代码实现 (C++)

```c++
#include <iostream>
#include <vector>
#include <cmath>
#include <iomanip>

using namespace std;

// 结构体存储基站坐标
struct Point {
    double x, y;
};

// 计算两点之间的欧几里得距离
double getDistance(const Point& a, const Point& b) {
    return sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y));
}

int main() {
    // 优化输入输出流
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int n;
    double l;
    if (!(cin >> n >> l)) return 0;

    vector<Point> points(n);
    for (int i = 0; i < n; ++i) {
        cin >> points[i].x >> points[i].y;
    }

    // Prim 算法核心变量
    const double INF = 1e18; // 用一个足够大的数表示无穷大
    vector<double> minDist(n, INF);
    vector<bool> visited(n, false);

    minDist[0] = 0; // 从第 0 个基站开始
    double totalLength = 0;
    bool possible = true;

    for (int i = 0; i < n; ++i) {
        int u = -1;
        double minVal = INF;

        // 1. 寻找当前未加入生成树且距离最近的点
        for (int j = 0; j < n; ++j) {
            if (!visited[j] && minDist[j] < minVal) {
                minVal = minDist[j];
                u = j;
            }
        }

        // 如果找不到有效的点，说明图不连通
        if (u == -1 || minVal == INF) {
            possible = false;
            break;
        }

        // 2. 将点 u 加入生成树，并累加距离
        visited[u] = true;
        totalLength += minVal;

        // 3. 用点 u 更新其他未加入生成树的点的距离
        for (int v = 0; v < n; ++v) {
            if (!visited[v]) {
                double d = getDistance(points[u], points[v]);
                // 只有当距离不超过给定的上限 l 时，才能修建线路
                if (d <= l && d < minDist[v]) {
                    minDist[v] = d;
                }
            }
        }
    }

    // 输出结果
    if (possible) {
        cout << fixed << setprecision(2) << totalLength << "\n";
    } else {
        cout << "Impossible\n";
    }

    return 0;
}
```

## 复杂度分析

- **时间复杂度**：$O(n^2)$。外层循环执行 $n$ 次，每次内层有两个并列的 $n$ 次循环（找最小值和更新距离），对于 $n \le 500$ 的数据量，运算量大概在 $2.5 \times 10^5$ 次左右，在 $1.00s$ 的时限内可以轻松秒过（通常在几毫秒内即可完成）。
- **空间复杂度**：$O(n)$。只使用了几个大小为 $n$ 的结构体数组和状态数组，内存消耗极小（远低于 $512\text{ MB}$ 的限制）。