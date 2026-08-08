---
title: 'Dijkstra 算法'
published: 1959-12
draft: false
series: '最短路径问题'
tags: ['图', '贪心']
toc: true
coverImage:
  src: './Edsger_Wybe_Dijkstra.png'
  alt: 'ACM1972 年图灵奖海报，迪杰斯特拉黑白人像，配授奖评语：为将程序设计发展为高深智力领域作出奠基性贡献'
---

**Dijkstra 算法**（迪杰斯特拉算法）是由荷兰计算机科学家 Edsger W. Dijkstra 于 1956 年提出、1959 年发表的经典图论算法。

它主要用于解决单源最短路径（Single-Source Shortest Path，SSSP）问题：在一个给定的带权图 $G=(V, E)$ 中，已知一个特定的起始节点（Source Node），计算从该起点到图中所有其他节点的最短路径长度。

## 算法核心思想：贪心 + 松弛

Dijkstra 的本质是**贪心策略**。它将图中的所有节点划分为两个集合：

- **已确定集 $S$**：已找到从起点出发的最短路径的节点集合。
- **未确定集 $U$**：尚未确定最短路径的节点集合。

### 算法执行步骤

1. **初始化**：

  - 将起点 $s$ 到自己的距离设为 $0$（$d[s] = 0$）。
  - 将起点到其他所有节点的距离设为无穷大（$d[v] = \infty$）。
  - 集合 $S$ 初始只包含起点 $s$。

2. **选取当前最近节点（贪心选择）**：

  - 从未确定的集合 $U$ 中，找出当前距离起点 $s$ 最近（$d[u]$ 最小）的节点 $u$。
  - 将节点 $u$ 加入已确定集合 $S$。此时，起点到 $u$ 的最短路径已经正式确定。

3. **松弛邻居节点（Relaxation）**：

  - 遍历节点 $u$ 的所有邻居 $v$。

  - 尝试通过 $u$ 作为中转点到达 $v$。如果发现通过 $u$ 走到 $v$ 的距离（即 $d[u] + weight(u, v)$）比目前记录的 $d[v]$ 更短，则更新 $d[v]$：

     $$d[v] = \min(d[v], d[u] + weight(u, v))$$

4. **循环重复**：

  - 重复步骤 2 和 3，直到所有节点都加入集合 $S$（或者集合 $U$ 为空）。

![](./dijkstra-step-by-step-demonstration.png 'Dijkstra 算法在带权有向图上的逐步执行过程演示图，展示了距离数组 dis、访问标记数组 vis 以及优先队列 pq 的状态变化。')

## 为何边权必须非负？

Dijkstra 算法生效的关键前提是：**图中所有边的权值必须非负（$\ge 0$）**。

- **理论逻辑**：算法基于贪心假设——当前从集合 $U$ 挑选出 $d[u]$ 最小的节点时，由于不存在负边权，后续不可能再通过更长的路径累加出更小的距离。
- **失效场景**：若图中存在负权边，经由后续节点松弛后可能产生更小的累加距离，从而“颠覆”之前已确定的答案。

> **提示**：若图中包含负权边，应改用 **Bellman-Ford 算法** 或 **SPFA 算法**。

## 算法实现与复杂度比较

根据寻找“距离最小节点”实现方式的不同，时间复杂度有显著差异：

| **实现方式**               | **查找最小值的方法**                          | **时间复杂度**      | **适用场景**                           |
| -------------------------- | --------------------------------------------- | ------------------- | -------------------------------------- |
| **朴素实现**               | 暴力遍历所有未访问节点                        | $O(V^2)$            | 稠密图（边数 $E \approx V^2$）         |
| **优先队列（小顶堆）优化** | 利用二叉堆（`std::priority_queue`）维护最小值 | $O((V + E) \log V)$ | 稀疏图（边数 $E \ll V^2$），**最常用** |
| **斐波那契堆优化**         | 高级数据结构                                  | $O(E + V \log V)$   | 理论最优，实际实现常数偏大             |

## 参考代码（优先队列优化版）

这是竞赛和实际工程中最常用的模板：

```c++
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

const int INF = 1e9; // 表示无穷大

// 结构体保存边信息：到达的目标节点 to，边的权重 weight
struct Edge {
    int to;
    int weight;
};

// 优先队列节点：当前节点 u，起点到 u 的当前已知最短距离 dist
struct Node {
    int u;
    int dist;
    // 重载大于号，让优先队列按照 dist 从小到大排序（构建小顶堆）
    bool operator>(const Node& other) const {
        return dist > other.dist;
    }
};

void dijkstra(int start_node, int n, const vector<vector<Edge>>& adj) {
    vector<int> dist(n + 1, INF); // 存储起点到各节点的最短距离
    priority_queue<Node, vector<Node>, greater<Node>> pq;

    // 初始化起点
    dist[start_node] = 0;
    pq.push({start_node, 0});

    while (!pq.empty()) {
        Node current = pq.top();
        pq.pop();

        int u = current.u;
        int d = current.dist;

        // 懒标记剪枝：如果弹出的距离大于已知最短距离，说明是失效的过时数据，直接跳过
        if (d > dist[u]) continue;

        // 遍历 u 的所有邻居边进行“松弛”
        for (const Edge& edge : adj[u]) {
            int v = edge.to;
            int weight = edge.weight;

            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({v, dist[v]});
            }
        }
    }

    // 输出起点到各个节点的最短距离
    for (int i = 1; i <= n; ++i) {
        if (dist[i] == INF) cout << "节点 " << i << ": 不可达\n";
        else cout << "节点 " << i << ": " << dist[i] << "\n";
    }
}
```

## 常见变体与应用场景

1. **多源最短路径转换（Multi-Source SSSP）**：

   若有多个起点，只需在初始化时将**所有起点**以初始距离 $0$ 存入优先队列，即可实现多源向外同时扩展。

2. **最长路与最大权值传播**：

   将松弛逻辑修改为“寻找最大值（$\max$）”，并将小顶堆替换为大顶堆，即可用于处理具备单调特性的最大权值扩散问题。

3. **地图导航与路径规划**：

   在地图导航（如 A* 搜寻算法）及路由协议（如 OSPF）中，Dijkstra 算法均为核心的底层基础。