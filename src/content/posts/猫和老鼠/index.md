---
title: '猫和老鼠'
published: 2025-12-29
draft: false
tags: ['Dijkstra 算法']
toc: true
---

> 猫和老鼠所在的庄园可以视为一张由 $n$ 个点和 $m$ 条带权无向边构成的连通图。结点依次以 $1,2,\ldots,n$ 编号，结点 $i$（$1\le i\le n$）有价值为 $c_i$ 的奶酪。在 $m$ 条带权无向边中，第 $i$（$1\le i\le m$）条无向边连接结点 $u_i$ 与结点 $v_i$，边权 $w_i$ 表示猫和老鼠通过这条边所需的时间。
>
> 猫窝位于结点 $a$，老鼠洞位于结点 $b$。对于老鼠而言，结点 $u$ 是**安全的**当且仅当：
>
> - 老鼠能规划一条从结点 $u$ 出发逃往老鼠洞的路径，使得对于路径上任意结点 $x$（包括结点 $u$ 与老鼠洞）都有：猫从猫窝出发到结点 $x$ 的最短时间**严格大于**老鼠从结点 $u$ **沿这条路径**前往结点 $x$ 所需的时间。
>
> 老鼠在拿取安全结点的奶酪时不存在被猫抓住的可能，但在拿取不是安全结点的奶酪时则不一定。为了确保万无一失，老鼠决定只拿取安全结点放置的奶酪。请你计算老鼠所能拿到的奶酪价值之和。
>
> **输入格式**
>
> 第一行，两个正整数 $n,m$，分别表示图的结点数与边数。
>
> 第二行，两个正整数 $a,b$，分别表示猫窝的结点编号，以及老鼠洞的结点编号。
>
> 第三行，$n$ 个正整数 $c_1,c_2,\ldots,c_n$，表示各个结点的奶酪价值。
>
> 接下来 $m$ 行中的第 $i$ 行（$1\le i\le m$）包含三个正整数 $u_i,v_i,w_i$，表示图中连接结点 $u_i$ 与结点 $v_i$ 的边，边权为 $w_i$。
>
> **输出格式**
>
> 输出一行，一个整数，表示老鼠所能拿到的奶酪价值之和。
>
> **输入样例 1**
>
> ```
> 5 5
> 1 2
> 1 2 4 8 16
> 1 2 4
> 2 3 3
> 3 4 1
> 2 5 2
> 3 1 8
> ```
>
> **输出样例 1**
>
> ```
> 22
> ```
>
> **输入样例 2**
>
> ```
> 6 10
> 3 4
> 1 1 1 1 1 1
> 1 2 6
> 2 3 3
> 3 1 4
> 3 4 5
> 4 5 8
> 5 6 2
> 6 4 1
> 3 2 4
> 5 4 4
> 3 3 6
> ```
>
> **输出样例 2**
>
> ```
> 3
> ```
>
> **数据范围**
>
> 对于 $40\%$ 的测试点，保证 $1\le n\le 500$，$1\le m\le 500$。
>
> 对于所有测试点，保证 $1\le n\le 10^5$，$1\le m\le 10^5$，$1\le a,b\le n$ 且 $a\neq b$，$1\le u_i,v_i\le n$，$1\le w_i\le 10^9$。
>

这道题本质上是一道极其精妙的**图论与数学变形题**。看似复杂的路径动态判定，推导后会发现一个非常惊艳的等价条件。

## 核心结论

一个结点 $u$ 是安全的，**当且仅当：老鼠从 $u$ 到老鼠洞 $b$ 的最短距离，严格小于猫从猫窝 $a$ 到老鼠洞 $b$ 的最短距离。**

即：

$$\text{结点 } u \text{ 安全} \iff d(u, b) < d(a, b)$$

只需以老鼠洞 $b$ 为起点跑**一次 Dijkstra 单源最短路**，即可在 $O((n+m)\log n)$ 的时间内解决全题！

## 严谨的数学推导证明

根据题目定义，结点 $u$ 安全，意味着存在一条从 $u$ 到 $b$ 的路径 $P$，使得对于 $P$ 上的**任意**结点 $x$，都有：

$$dist_P(u, x) < d_{cat}(x)$$

其中 $dist_P(u, x)$ 表示老鼠沿路径 $P$ 从 $u$ 走到 $x$ 所需的时间；$d_{cat}(x)$ 表示猫从猫窝 $a$ 到 $x$ 的图上最短时间。

### 符号定义

- $d(x, y)$：结点 $x$ 到 $y$ 的**最短路径长度**（满足三角不等式：$d(x, z) \le d(x, y) + d(y, z)$）。
- $d_{cat}(x) = d(a, x)$：猫从猫窝 $a$ 到 $x$ 的最短时间。
- $\text{dist}_P(u, x)$：老鼠沿路径 $P$ 从 $u$ 走到 $x$ 的实际耗时。
- **安全定义**：存在路径 $P$（起点 $u$，终点 $b$），使得 $\forall x \in P$，均有 $\text{dist}_P(u, x) < d_{cat}(x)$。

### 双向证明

#### 必要性（$u \text{ 安全} \implies d(u, b) < d(a, b)$）

若 $u$ 安全，则存在一条合法逃跑路径 $P$。直接取路径 $P$ 的终点 $x = b$，代入定义得：

$$\text{dist}_P(u, b) < d_{cat}(b) = d(a, b)$$

结合最短路性质 $d(u, b) \le \text{dist}_P(u, b)$，由传递性可得：

$$d(u, b) < d(a, b) \quad \blacksquare$$

#### 充分性（$d(u, b) < d(a, b) \implies u \text{ 安全}$）

已知 $d(u, b) < d(a, b)$，**构造路径 $P^*$ 为 $u$ 到 $b$ 的图上最短路径**。

对于 $P^*$ 上的任意结点 $x$：

1. **最短路分解**：$\text{dist}_{P^*}(u, x) = d(u, x) = d(u, b) - d(x, b)$。
2. **三角不等式**：$d(a, b) \le d(a, x) + d(x, b) \implies d_{cat}(x) = d(a, x) \ge d(a, b) - d(x, b)$。

利用已知条件 $d(u, b) < d(a, b)$ 进行放缩：

$$\text{dist}_{P^*}(u, x) = d(u, b) - d(x, b) < d(a, b) - d(x, b) \le d_{cat}(x)$$

即对 $P^*$ 上的任意结点 $x$ 均满足 $\text{dist}_{P^*}(u, x) < d_{cat}(x)$。说明直接走最短路径 $P^*$ 即可安全逃脱，故 $u$ 必为安全结点。 $\blacksquare$

## 算法步骤

1. 将无向图建立邻接表（注意边权可达 $10^9$，距离需用 `long long` 存储）。
2. 以老鼠洞 $b$ 为起点，运行一次 Dijkstra 算法，求出 $b$ 到所有结点 $i$ 的最短路 $d(i, b)$。
3. 由于无向图的对称性，$d(a, b)$ 即为 $b$ 到 $a$ 的最短距离（即猫到老鼠洞的时间）。
4. 遍历每个结点 $u$（$1 \le u \le n$）：
  - 若 $d(u, b) < d(a, b)$，则 $u$ 是安全结点，累加奶酪价值 $c_u$。
5. 输出总价值。

## 参考代码

```c++
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

const long long INF = 1e18; // 边权最大 10^9，距离可能达到 10^14，需开 long long

struct Edge {
    int to;
    long long w;
};

int main() {
    // 加速 I/O
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int n, m;
    if (!(cin >> n >> m)) return 0;

    int a, b;
    cin >> a >> b;

    vector<long long> c(n + 1);
    for (int i = 1; i <= n; ++i) {
        cin >> c[i];
    }

    vector<vector<Edge>> adj(n + 1);
    for (int i = 0; i < m; ++i) {
        int u, v;
        long long w;
        cin >> u >> v >> w;
        adj[u].push_back({v, w});
        adj[v].push_back({u, w});
    }

    // 以老鼠洞 b 为起点跑单源最短路
    vector<long long> dist(n + 1, INF);
    priority_queue<pair<long long, int>, vector<pair<long long, int>>, greater<pair<long long, int>>> pq;

    dist[b] = 0;
    pq.push({0, b});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if (d > dist[u]) continue;

        for (const auto& edge : adj[u]) {
            if (dist[u] + edge.w < dist[edge.to]) {
                dist[edge.to] = dist[u] + edge.w;
                pq.push({dist[edge.to], edge.to});
            }
        }
    }

    // 猫到老鼠洞的时间 d_cat_b 等于 dist[a]
    long long d_cat_b = dist[a];

    // 统计安全结点的奶酪价值和
    long long total_cheese = 0;
    for (int u = 1; u <= n; ++u) {
        if (dist[u] < d_cat_b) {
            total_cheese += c[u];
        }
    }

    cout << total_cheese << "\n";

    return 0;
}
```

## 复杂度分析

- **时间复杂度**：$\mathcal{O}((n + m) \log n)$。只需跑一次优先队列优化的 Dijkstra 算法，完全轻松应对 $n, m \le 10^5$ 的数据规模。
- **空间复杂度**：$\mathcal{O}(n + m)$，用于保存图的邻接表与最短路数组。