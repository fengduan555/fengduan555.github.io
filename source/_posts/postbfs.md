---
title: bfs
date: 2026-11-25 15:54:30
tags:
  - csp
categories:
  - 算法
---
# BFS（广度优先搜索）知识点与 C++ 模板

## 一、本质与适用场景

BFS（Breadth-First Search，广度优先搜索）本质是**按“距离起点的层数”由近到远逐层扩展**的遍历方式。

### 适用场景

| 场景 | 说明 |
|---|---|
| 无权图/边权相同的最短路 | 第一次到达即最短 |
| 网格最短路（上下左右） | 迷宫、岛屿、连通块 |
| 多源最短路 | 多个起点同时扩散（如“腐烂的橘子”） |
| 层序处理 | 二叉树层序遍历、按层统计 |
| 状态空间搜索 | 八数码、单词接龙等 |
| 0-1 权图 | 用 deque 的 0-1 BFS |

> **不适用**：边权不统一的一般图最短路（用 Dijkstra）。

---

## 二、核心三要素

1. **队列 `queue`**：FIFO，保证先入队的（距离近的）先被扩展。
2. **访问标记 `visited` / `dist`**：防止重复入队、死循环。
3. **距离/层数 `dist` 或 `step`**：记录从起点到该点的最短步数。

---

## 三、关键细节（易错点集中区）

### 1. 入队即标记，而不是出队时标记

这是 BFS 最重要的细节。若出队才标记，同一节点可能被多个邻居重复入队，队列膨胀甚至超时。

```cpp
if (dist[v] != -1) continue;   // 已访问，跳过
dist[v] = dist[u] + 1;         // 入队前就赋值/标记
q.push(v);
```

### 2. 按层扩展

需要“第几层/第几步”时，用 `len = q.size()` 固定当前层节点数：

```cpp
int step = 0;
while (!q.empty()) {
    int len = q.size();
    for (int i = 0; i < len; ++i) { /* 处理本层 */ }
    ++step;
}
```

### 3. 多源 BFS

把所有源点**一次性全部入队**，`dist = 0`，然后正常 BFS。这样得到的是“到最近源点的距离”。

### 4. 路径还原

用 `parent[v] = u` 记录前驱，最后从终点回溯到起点再反转。

### 5. 图可能不连通

需要外层循环枚举所有未访问点作为新起点（连通块计数、遍历全图）。

### 6. 边界与合法性

网格题必须检查 `0 <= nx < n && 0 <= ny < m`；注意障碍、重复边、自环。

### 7. 访问标记的替代

- 距离数组 `dist` 初始化为 `-1`，用 `dist[v] == -1` 判断未访问，可省一个 `visited` 数组。
- 也可直接原地修改网格（如把 `'1'` 改成 `'0'`）来标记。

### 8. 复杂度

- 邻接表：时间 `O(V + E)`，空间 `O(V)`
- 邻接矩阵：时间 `O(V²)`
- 网格：`O(n·m)`，空间 `O(n·m)`

---

## 四、C++ 模板

### 模板 1：通用图 BFS（邻接表，单源最短路）

```cpp
#include <bits/stdc++.h>
using namespace std;

// n 个点，编号 0..n-1；adj 为邻接表；s 为起点
// 返回 dist，dist[i] == -1 表示不可达
vector<int> bfs(int n, const vector<vector<int>>& adj, int s) {
    vector<int> dist(n, -1);
    queue<int> q;
    dist[s] = 0;          // 入队即标记
    q.push(s);

    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) {
            if (dist[v] != -1) continue;   // 已访问
            dist[v] = dist[u] + 1;
            q.push(v);
        }
    }
    return dist;
}
```

### 模板 2：网格 BFS（四方向最短路）

```cpp
// grid: 0 可走，1 障碍；从 (sx, sy) 到 (tx, ty)
int bfsGrid(vector<vector<int>>& grid, int sx, int sy, int tx, int ty) {
    int n = grid.size(), m = grid[0].size();
    const int dx[4] = {-1, 1, 0, 0};
    const int dy[4] = {0, 0, -1, 1};

    vector<vector<int>> dist(n, vector<int>(m, -1));
    queue<pair<int,int>> q;
    dist[sx][sy] = 0;
    q.push({sx, sy});

    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        if (x == tx && y == ty) return dist[x][y];  // 首次到达即最短

        for (int d = 0; d < 4; ++d) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (grid[nx][ny] == 1) continue;        // 障碍
            if (dist[nx][ny] != -1) continue;       // 已访问
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    return -1;  // 不可达
}
```

### 模板 3：多源 BFS

```cpp
// 所有源点初始 dist=0 一起入队，求每个点到最近源点的距离
vector<vector<int>> multiSourceBFS(vector<vector<int>>& grid,
                                   const vector<pair<int,int>>& sources) {
    int n = grid.size(), m = grid[0].size();
    const int dx[4] = {-1, 1, 0, 0};
    const int dy[4] = {0, 0, -1, 1};

    vector<vector<int>> dist(n, vector<int>(m, -1));
    queue<pair<int,int>> q;

    for (auto [x, y] : sources) {
        dist[x][y] = 0;
        q.push({x, y});
    }

    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        for (int d = 0; d < 4; ++d) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (grid[nx][ny] == 1) continue;   // 障碍
            if (dist[nx][ny] != -1) continue;
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    return dist;
}
```

### 模板 4：按层 BFS

```cpp
// 返回每一层的节点列表
vector<vector<int>> levelOrder(int n, const vector<vector<int>>& adj, int s) {
    vector<vector<int>> levels;
    vector<bool> vis(n, false);
    queue<int> q;
    vis[s] = true;
    q.push(s);

    while (!q.empty()) {
        int len = q.size();
        vector<int> cur;
        for (int i = 0; i < len; ++i) {
            int u = q.front(); q.pop();
            cur.push_back(u);
            for (int v : adj[u]) {
                if (vis[v]) continue;
                vis[v] = true;
                q.push(v);
            }
        }
        levels.push_back(move(cur));
    }
    return levels;
}
```

### 模板 5：BFS 路径还原

```cpp
// 返回从 s 到 t 的一条最短路径；不可达返回空
vector<int> bfsPath(int n, const vector<vector<int>>& adj, int s, int t) {
    vector<int> dist(n, -1), parent(n, -1);
    queue<int> q;
    dist[s] = 0;
    q.push(s);

    while (!q.empty()) {
        int u = q.front(); q.pop();
        if (u == t) break;
        for (int v : adj[u]) {
            if (dist[v] != -1) continue;
            dist[v] = dist[u] + 1;
            parent[v] = u;
            q.push(v);
        }
    }

    if (dist[t] == -1) return {};
    vector<int> path;
    for (int cur = t; cur != -1; cur = parent[cur])
        path.push_back(cur);
    reverse(path.begin(), path.end());
    return path;
}
```

### 模板 6：0-1 BFS（边权 0/1）

```cpp
// 边权只有 0 和 1 时，用 deque：0 权入队首，1 权入队尾，O(V+E)
vector<int> bfs01(int n, const vector<vector<pair<int,int>>>& adj, int s) {
    const int INF = 1e9;
    vector<int> dist(n, INF);
    deque<int> dq;
    dist[s] = 0;
    dq.push_back(s);

    while (!dq.empty()) {
        int u = dq.front(); dq.pop_front();
        for (auto [v, w] : adj[u]) {   // w ∈ {0, 1}
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.push_front(v);
                else        dq.push_back(v);
            }
        }
    }
    return dist;
}
```

---

## 五、常见坑总结

1. **出队才标记** → 重复入队、超时。应入队即标记。
2. **忘记 `len = q.size()`** → 层数算错。
3. **多源 BFS 忘了全部入队** → 退化成单源。
4. **网格边界检查顺序写反** → 越界访问。
5. **在 `while` 里修改 `q.size()` 判断** → 层循环失效。
6. **需要路径时只记距离不记 parent** → 无法还原。
7. **边权不为 1 仍用普通 BFS** → 结果错误，需 Dijkstra 或 0-1 BFS。

---

**一句话总结**：BFS = 队列 + 入队即标记 + 按层扩展；在无权图上首次到达即最短，模板化后只需替换“邻居生成规则”（图邻居 / 网格四方向 / 状态转移）即可复用。

