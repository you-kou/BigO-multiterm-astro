---
title: 'Prim 算法'
published: 1930
draft: false
tags: ['最小生成树', 'Prim 算法']
toc: true
coverImage:
  src: './Vojtech_Jarnik_Robert_Prim.jpg'
  alt: '左：Vojtěch Jarník（亚尔尼克，最小生成树算法最早提出者）；右：Robert C. Prim（罗伯特・普里姆，算法重发现者）黑白双人拼接肖像，Jarník 身着学术礼服配奖章，Prim 穿西装领带'
---

> **最小生成树（Minimum Spanning Tree，简称 MST）**是**带权无向连通图**里的一棵**边权和最小、包含所有顶点、无环**的生成树。
>
> **核心性质**
>
> 1. 一定包含     **n 个顶点，n-1 条边**
> 2. 连通、无环、是树
> 3. 总权值在所有生成树中**最小**
> 4. 边权**互不相同**时，MST **唯一**

Prim 算法（普里姆算法）**是一种用来寻找加权连通图的**最小生成树（Minimum Spanning Tree, 简称 MST）的经典算法。

它的核心思想可以用一句话概括：**从一个点出发，每次都选择一条“连接已选区域和未选区域”的最短边，直到把所有点都连起来。**

## 核心思想

Prim 算法的贪心策略聚焦「局部最优」：

1. 初始化：选任意一个顶点作为**初始连通集**（比如顶点 0）；

2. 每次从「连通集 ↔ 非连通集」的所有边中，选**权值最小**的那条边；

3. 将这条边的「非连通集顶点」加入连通集；

4. 重复步骤 2-3，直到所有顶点都加入连通集（共选 n-1 条边）；

5. 最终选中的边构成最小生成树。

![Prim 算法构建最小生成树的步骤演示](./Prim算法构建最小生成树的步骤演示.png 'Prim 算法构建最小生成树的步骤演示')

Prim 有两种经典实现，适配不同场景：

| **实现方式**       | **数据结构**      | **时间复杂度** | **适合场景**                   |
| ------------------ | ----------------- | -------------- | ------------------------------ |
| 朴素版（邻接矩阵） | 二维数组          | $O(V^2)$       | 顶点少（$V \le 1000$）、边稠密 |
| 堆优化版（邻接表） | 优先队列 + 邻接表 | $O(E \log V)$  | 顶点多、边稀疏                 |

## 朴素实现（邻接矩阵）

每次都暴力遍历所有边，时间复杂度为 $O(V^2)$（其中 $V$ 是顶点数）。适合**稠密图**（边非常多的图）。

```c++
#include <vector>
#include <climits>
#include <iostream>

using namespace std;

// 定义无限大，表示两点之间没有直接相连的边
const int INF = INT_MAX;

/**
 * @brief 朴素版 Prim 算法（基于邻接矩阵）
 * @param graph 邻接矩阵，graph[i][j] 表示点 i 到点 j 的边权，若不相连则为 0 或 INF
 * @param V 顶点数量
 */
void primMST(const vector<vector<int>>& graph, int V) {
    // 存储每个节点在最小生成树（MST）中的父节点
    vector<int> parent(V, -1);
    
    // 存储从未加入MST的集合到各个顶点的最短切边权重
    vector<int> minWeight(V, INF);
    
    // 标记顶点是否已经加入 MST 阵营
    vector<bool> inMST(V, false);

    // 从顶点 0 开始构建
    minWeight[0] = 0; 

    for (int count = 0; count < V; ++count) {
        // 1. 在所有“未加入MST”的点中，寻找最小切边对应的顶点 u
        int u = -1;
        int minVal = INF;
        for (int v = 0; v < V; ++v) {
            if (!inMST[v] && minWeight[v] < minVal) {
                minVal = minWeight[v];
                u = v;
            }
        }

        // 如果找不到有效边，说明剩余的点与当前树不连通
        if (u == -1) return;

        // 2. 将点 u 正式拉入 MST 阵营
        inMST[u] = true;

        // 3. 更新与 u 相邻且未加入 MST 的顶点的权重
        for (int v = 0; v < V; ++v) {
            // graph[u][v] != 0 表示 u 和 v 之间有边相连
            if (graph[u][v] != 0 && !inMST[v] && graph[u][v] < minWeight[v]) {
                parent[v] = u;
                minWeight[v] = graph[u][v];
            }
        }
    }

    // 4. 打印或处理最终生成的 MST 结果（可根据需求改为返回 parent 数组）
    cout << "Edge \tWeight\n";
    for (int i = 1; i < V; ++i) {
        cout << parent[i] << " - " << i << " \t" << graph[i][parent[i]] << "\n";
    }
}
```

## 优先队列优化（邻接表 + 二叉堆）

用最小堆来维护所有的切边，每次直接弹出最小边，时间复杂度可以降到 $O(E \log V)$（其中 $E$ 是边数）。适合**稀疏图**。

```c++
#include <vector>
#include <queue>
#include <climits>
#include <iostream>

using namespace std;

// 定义无限大，表示两点之间没有直接相连的边
const int INF = INT_MAX;

// 定义别名：pair<边权, 目标顶点>，用作堆中的元素
// 注意：pair 默认先按第一个元素（边权）排序，这正是我们想要的
using PII = pair<int, int>;

// 定义邻接表中的边结构
struct Edge {
    int to;     // 目标顶点
    int weight; // 边权
};

/**
 * @brief 优先队列优化版 Prim 算法（基于邻接表 + 二叉堆）
 * @param adjList 邻接表，adjList[u] 存储从顶点 u 出发的所有邻边
 * @param V 顶点数量
 */
void primMST(const vector<vector<Edge>>& adjList, int V) {
    // 存储每个节点在最小生成树（MST）中的父节点
    vector<int> parent(V, -1);
    
    // 存储从未加入MST的集合到各个顶点的最短切边权重
    vector<int> minWeight(V, INF);
    
    // 标记顶点是否已经加入 MST 阵营
    vector<bool> inMST(V, false);

    // C++ 默认是大顶堆，使用 greater<PII> 可以将其转换为小顶堆（二叉堆）
    // 堆中元素格式为：pair<minWeight[v], v>
    priority_queue<PII, vector<PII>, greater<PII>> pq;

    // 从顶点 0 开始构建
    minWeight[0] = 0;
    pq.push({0, 0}); // 将起点推入优先队列

    while (!pq.empty()) {
        // 1. 从堆顶直接弹出当前未加入MST中切边最短的顶点 u
        int u = pq.top().second;
        pq.pop();

        // 惰性删除：如果顶点 u 已经在 MST 中了，直接跳过
        if (inMST[u]) continue;

        // 2. 将点 u 正式拉入 MST 阵营
        inMST[u] = true;

        // 3. 遍历点 u 的所有邻居，更新那些“未加入MST”的邻居的 minWeight
        for (const auto& edge : adjList[u]) {
            int v = edge.to;
            int weight = edge.weight;

            // 如果 v 不在 MST 中，且通过 u 到达 v 的边权比 v 之前记录的切边还短
            if (!inMST[v] && weight < minWeight[v]) {
                parent[v] = u;
                minWeight[v] = weight;
                // 将更新后的切边权重和顶点 v 压入优先队列
                pq.push({minWeight[v], v});
            }
        }
    }

    // 4. 打印或处理最终生成的 MST 结果
    cout << "Edge \tWeight\n";
    for (int i = 1; i < V; ++i) {
        if (parent[i] != -1) {
            cout << parent[i] << " - " << i << " \t" << minWeight[i] << "\n";
        }
    }
}
```

## 为什么贪心策略能保证「总权值最小」？

Prim 贪心策略的正确性，核心依赖 MST 的**切分定理（Cut Property）** —— 这是所有 MST 贪心算法的理论基石（Kruskal 也基于此）。

### 切分定理（回顾：MST 的核心性质）

- **切分**：将图的顶点集划分为两个非空且不相交的子集 S（已选入生成树的顶点）和 V-S（未选顶点）；
- **横切边**：连接 S 和 V-S 的边（一端在 S，一端在 V-S）；
- 切分定理：**对于任意切分，权值最小的横切边必然属于 MST**。

### Prim 与切分定理的完美匹配

Prim 每一步的选择，本质是对「当前切分（S, V-S）」应用切分定理：

- 初始切分：S={s}，V-S = 其余顶点 → 选最小横切边加入 MST（符合切分定理）；
- 每扩张一次 S：新的切分是（新 S, V - 新 S）→ 再次选最小横切边加入 MST；
- 直到 S=V：所有选的边都是 “对应切分的最小横切边”，且这些边恰好组成 MST（无环、连通、总权值最小）。

### 反证法：为什么局部最优能推全局最优？

假设存在一棵总权值更小的生成树 T'，与 Prim 生成的树 T 不同：

- 找到 T 和 T' 中第一条权值不同的边 e：e 是 Prim 选的 “当前切分最小横切边”，而 T' 中替换 e 的边 e' 权值更大；
- 根据切分定理，e 是该切分下的最优横切边，替换 e 为 e' 会让 T' 的总权值变大，与 “T' 更优” 矛盾；
- 因此 Prim 每一步的局部最优选择，最终必然组成全局最优的 MST。

### 为什么不会选到 “无效边”？

Prim 只选「横切边」，而横切边有两个关键特性：

- **无环**：横切边连接 “已选顶点” 和 “未选顶点”，加入后不会形成环（生成树的必要条件）；
- **必连通**：每一步都向 S 中添加新顶点，直到覆盖所有顶点，最终必然是连通图。