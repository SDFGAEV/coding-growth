# 图形解空间：算法认知框架

> 本文档是“算法认知框架”系列的第二部分，聚焦于**图形解空间**的算法设计与实现。

## 核心理念

### 图形解空间定义
图形解空间由**图节点**和**边**构成，节点之间具有**多对多**的连接关系。解可以是图中的单个节点、一条路径、一个连通分量、一棵生成树、一个匹配、一个环、一个流分配或一个博弈状态。

**基础元素**：图节点、边（有向/无向、有权/无权）。

### 算法本质（在图形解空间中的体现）
**算法 = 在解空间上的聪明穷举 + 辅助变量优化**，对于图形解空间：
- **无遗漏**：必须考虑所有可能的解（如所有路径、所有生成树等）
- **无冗余**：利用图的结构特性（连通性、权值、环）避免重复计算

### 解空间的人为构造
解空间本身是问题的所有可能解的集合，其结构是抽象的。但在设计算法时，我们可以人为地为解空间赋予一种便于遍历的结构。**同一问题的解空间可以有多种不同的构造方式，选择不同的结构会导致不同的算法策略。**

例如：
- **路径解**：原始解空间是所有路径（指数级）。我们可以将其组织成**图**本身，利用 BFS 按层扩展（无权）或 Dijkstra 按距离扩展（带权），避免枚举所有路径。
- **连通分量解**：通过并查集将集合关系组织成树形，实现近乎常数的查询。

因此，在分析图形解空间时，我们关注的是将解空间构造为图结构后的遍历和优化。


## 一、图形解空间的解的形态

图形解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构（人为构造） |
|:---|:---|:---|:---|
| **单点解** | 单个节点 | 节点度、连通分量中的节点 | 节点集合（线性） |
| **路径解** | 节点序列，相邻节点有边相连 | DFS/BFS路径、最短路径（Dijkstra、Floyd）、拓扑排序、网格路径 | 路径集合（图） |
| **连通分量解** | 极大连通子图 | 并查集、DFS/BFS求连通分量、强连通分量（Tarjan）、双连通分量、割点、桥 | 子图集合 |
| **生成树解** | 包含所有节点的树形子图 | 最小生成树（Prim、Kruskal） | 树形子图集合 |
| **匹配解** | 无公共顶点的边集 | 二分图最大匹配（匈牙利）、带权匹配（KM） | 边集集合 |
| **圈解** | 简单环 | 环检测、欧拉回路（Hierholzer）、哈密顿回路 | 环集合 |
| **流解** | 满足容量约束和流量守恒的边流量分配 | 最大流（Dinic、Edmonds-Karp）、最小割、费用流（MCMF） | 函数空间 |
| **博弈状态解** | 游戏状态图上的胜态或 SG 值 | Nim游戏、SG函数、威佐夫博弈 | 状态图 |


## 二、遍历框架与遍历控制变量

图形解空间的遍历框架多样，取决于问题性质。

| 解的形态 | 数据结构 | 遍历框架（模板） | 遍历控制变量（模板） |
|:---|:---|:---|:---|
| **单点解** | 图 | 遍历所有节点 | 节点编号 `i` |
| **路径解（无权）** | 图 | BFS（队列） | 队列中的节点 |
| **路径解（带权非负）** | 图 | Dijkstra（优先队列） | 优先队列中的节点 |
| **路径解（多源）** | 图 | Floyd（三层循环） | `k`, `i`, `j` |
| **拓扑排序** | 有向无环图 | Kahn（队列） | 入度为0的节点 |
| **连通分量解** | 图 | DFS/BFS 或并查集 | 节点编号 |
| **生成树解** | 图 | Kruskal（边排序） | 边（按权值） |
| **匹配解** | 二分图 | 匈牙利（DFS回溯） | 左部节点 |
| **圈解** | 图 | DFS 三色标记 | 节点编号 |
| **流解** | 有向图 | Dinic（BFS+DFS） | 当前节点、当前弧 |
| **博弈状态解** | 有向图 | 递归 + 记忆化 | 当前状态 |


## 三、节点操作

节点操作是在每个被访问的节点或边上执行的具体逻辑。根据算法不同，操作时机各异。

| 解的形态 | 算法 | 节点操作（模板） | 时机 |
|:---|:---|:---|:---|
| **单点解** | 遍历 | 访问节点 | 前序 |
| **路径解（BFS）** | BFS | 邻居入队，标记距离 | 出队时 |
| **路径解（Dijkstra）** | Dijkstra | 松弛邻居 | 取出节点时 |
| **拓扑排序** | Kahn | 输出节点，邻居入度减1 | 出队时 |
| **连通分量解** | 并查集 | 查找根，合并 | 操作时 |
| **生成树解** | Kruskal | 判断两端是否连通，合并 | 遍历边时 |
| **匹配解** | 匈牙利 | 尝试匹配，若冲突则递归 | DFS 回溯时 |
| **圈解** | DFS | 标记状态，检测回边 | 递归前/后 |
| **流解** | Dinic | BFS分层，DFS找增广路 | 分层时、找增广时 |
| **博弈状态解** | SG函数 | 计算mex | 后序 |


## 四、辅助变量

辅助变量用于记录状态、剪枝、缓存结果，保证无冗余。

| 解的形态 | 算法 | 辅助变量（模板） | 功能分类 |
|:---|:---|:---|:---|
| **单点解** | 图遍历 | `vector<bool> visited` | 状态标记 |
| **路径解** | BFS | `vector<int> dist` | 临时计算 |
| **路径解** | Dijkstra | `vector<int> dist`，优先队列 | 优化缓存 |
| **路径解** | Floyd | `vector<vector<int>> dist` | 优化缓存 |
| **拓扑排序** | Kahn | `vector<int> indegree` | 状态标记 |
| **连通分量解** | 并查集 | `vector<int> parent, rank` | 状态标记 |
| **生成树解** | Kruskal | 并查集 | 状态标记 |
| **匹配解** | 匈牙利 | `vector<int> match`, `vector<bool> visited` | 状态标记 |
| **圈解** | DFS | `vector<int> state` | 状态标记 |
| **流解** | Dinic | `vector<int> level`, `vector<int> iter` | 优化缓存 |
| **博弈状态解** | SG函数 | `vector<int> sg` | 优化缓存 |

### 变量生命周期与存放位置
图形算法通常以函数形式实现，辅助变量多为局部数组或类成员。

| 类型 | 描述 | 示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次算法执行中存在 | `visited`, `dist` 数组 | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | DFS中的 `visited` | 递归参数（引用） |
| **类内长期** | 跨多次公共方法调用持久 | 图结构本身、并查集对象 | 类的成员变量 |


## 五、各子类算法详细模板

### 一、单点解

**定义**：解是图中的单个节点，即需要统计节点信息或判断节点是否存在。

#### 暴力枚举（遍历所有节点）

##### 通用模板（所有图结构）

**遍历框架（仅控制变量）**
```
for each node in 图:
    节点操作：处理当前节点
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：节点编号 u
// 辅助变量：无
for u = 0 to n-1:
    // 节点操作：处理节点 u
```

##### 数据结构：邻接表 / 邻接矩阵

**遍历框架（仅控制变量）**
```cpp
for (int u = 0; u < n; ++u) {
    // 节点操作：处理节点 u
}
```

**完整模板（含辅助变量）**
```cpp
// 示例：统计节点度数
vector<int> degree(n, 0);
for (int u = 0; u < n; ++u) {                // 遍历控制变量：u
    degree[u] = graph[u].size();              // 节点操作：统计度数
}
// 辅助变量：degree（结果存储）
```


### 二、路径解

**定义**：解是节点序列，相邻节点有边相连，如最短路径、拓扑序列等。

#### 暴力枚举（枚举所有路径）

##### 通用模板（所有图结构）

**遍历框架（仅控制变量）**
```
递归函数(当前节点, 路径):
    if 到达目标:
        记录路径
        return
    for 每个邻居:
        递归(邻居, 路径+邻居)
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（回溯）
// 遍历控制变量：当前节点 u，当前路径 path
// 辅助变量：visited（状态标记），结果集 res
void dfs(u, path):
    visited[u] = true
    if u == target:
        res.add(path)
    else:
        for v in graph[u]:
            if not visited[v]:
                dfs(v, path + [v])
    visited[u] = false
```

##### 数据结构：邻接表

**遍历框架（仅控制变量）**
```cpp
void dfs(int u, vector<int>& path, vector<bool>& visited, vector<vector<int>>& res) {
    visited[u] = true;
    path.push_back(u);
    if (u == target) {
        res.push_back(path);
    } else {
        for (int v : graph[u]) {
            if (!visited[v]) dfs(v, path, visited, res);
        }
    }
    path.pop_back();
    visited[u] = false;
}
```

**完整模板（含辅助变量）**
```cpp
// 示例：枚举所有从 start 到 end 的路径（不包含环）
vector<vector<int>> allPaths;
vector<bool> visited(n, false);
vector<int> path;
function<void(int)> dfs = [&](int u) {
    visited[u] = true;
    path.push_back(u);
    if (u == end) {
        allPaths.push_back(path);
    } else {
        for (int v : graph[u]) {
            if (!visited[v]) dfs(v);
        }
    }
    path.pop_back();
    visited[u] = false;
};
dfs(start);
```

---

#### 优化算法1：无权图最短路径（BFS）

**优化说明**：利用**BFS按层扩展**的特性，第一次访问到某节点时即为最短路径，将指数级枚举降为 O(V+E)。

##### 通用模板

**遍历框架（仅控制变量）**
```
队列 q
dist[start] = 0
q.push(start)
while q not empty:
    u = q.front(); q.pop()
    for v in graph[u]:
        if dist[v] == -1:
            dist[v] = dist[u] + 1
            q.push(v)
```

**完整模板（含辅助变量）**
```
// 遍历框架：BFS（队列）
// 遍历控制变量：队列中的节点 u
// 辅助变量：dist（距离数组），queue
vector<int> dist(n, -1);
queue<int> q;
dist[start] = 0;
q.push(start);
while (!q.empty()) {
    int u = q.front(); q.pop();                // 遍历控制变量：u
    for (int v : graph[u]) {                  // 节点操作：遍历邻居
        if (dist[v] == -1) {
            dist[v] = dist[u] + 1;
            q.push(v);
        }
    }
}
```

##### 数据结构：图（无权）

**遍历框架（仅控制变量）**
```cpp
queue<int> q;
vector<int> dist(n, -1);
dist[start] = 0;
q.push(start);
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : graph[u]) {
        if (dist[v] == -1) {
            dist[v] = dist[u] + 1;
            q.push(v);
        }
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> bfsShortestPath(int start, const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> dist(n, -1);                    // 辅助变量：距离数组
    queue<int> q;                               // 遍历控制变量：队列中的节点
    dist[start] = 0;
    q.push(start);
    while (!q.empty()) {
        int u = q.front(); q.pop();             // 遍历控制变量：u
        for (int v : graph[u]) {                // 节点操作：遍历邻居
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
    return dist;
}
```

---

#### 优化算法2：带权图最短路径（Dijkstra）

**优化说明**：利用**贪心**和**优先队列**，每次选择当前距离最小的节点进行松弛，保证已确定的节点不会被再次更新，将 O(V²) 降为 O((V+E) log V)。

##### 通用模板

**遍历框架（仅控制变量）**
```
dist[start] = 0
pq.push({0, start})
while pq not empty:
    (d, u) = pq.top(); pq.pop()
    if d > dist[u]: continue
    for (v, w) in graph[u]:
        if dist[u] + w < dist[v]:
            dist[v] = dist[u] + w
            pq.push({dist[v], v})
```

**完整模板（含辅助变量）**
```
// 遍历框架：优先队列迭代
// 遍历控制变量：优先队列中的节点 u
// 辅助变量：dist（距离数组），pq（优先队列）
vector<int> dist(n, INF);
priority_queue<pair<int,int>, vector<...>, greater<...>> pq;
dist[start] = 0;
pq.push({0, start});
while (!pq.empty()) {
    auto [d, u] = pq.top(); pq.pop();          // 遍历控制变量：u
    if (d > dist[u]) continue;                  // 节点操作：过期判断
    for (auto [v, w] : graph[u]) {              // 节点操作：遍历邻居
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            pq.push({dist[v], v});               // 节点操作：松弛
        }
    }
}
```

##### 数据结构：带权图

**遍历框架（仅控制变量）**
```cpp
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
vector<int> dist(n, INF);
dist[start] = 0;
pq.push({0, start});
while (!pq.empty()) {
    int u = pq.top().second; pq.pop();
    // 如果当前距离大于记录的距离，跳过
    for (auto [v, w] : graph[u]) {
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            pq.push({dist[v], v});
        }
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> dijkstra(int start, const vector<vector<pair<int,int>>>& graph) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);               // 辅助变量：距离数组
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // 遍历控制变量
    dist[start] = 0;
    pq.push({0, start});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();       // 遍历控制变量：u
        if (d > dist[u]) continue;               // 节点操作：过期判断
        for (auto [v, w] : graph[u]) {          // 节点操作：遍历邻居
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});           // 节点操作：松弛
            }
        }
    }
    return dist;
}
```

---

#### 优化算法3：多源最短路径（Floyd-Warshall）

**优化说明**：利用**动态规划**，通过中间点 k 逐步松弛所有点对，时间复杂度 O(n³)，适用于小规模图。

##### 通用模板

**遍历框架（仅控制变量）**
```
for k = 0 to n-1:
    for i = 0 to n-1:
        for j = 0 to n-1:
            if dist[i][k] + dist[k][j] < dist[i][j]:
                dist[i][j] = dist[i][k] + dist[k][j]
```

**完整模板（含辅助变量）**
```
// 遍历框架：三层循环
// 遍历控制变量：k, i, j
// 辅助变量：dist（距离矩阵）
for k = 0 to n-1:
    for i = 0 to n-1:
        for j = 0 to n-1:
            if dist[i][k] + dist[k][j] < dist[i][j]:   // 节点操作：松弛
                dist[i][j] = dist[i][k] + dist[k][j]
```

##### 数据结构：带权图（邻接矩阵）

**遍历框架（仅控制变量）**
```cpp
for (int k = 0; k < n; ++k) {
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j) {
            if (dist[i][k] < INF && dist[k][j] < INF) {
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
            }
        }
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<vector<int>> floydWarshall(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<vector<int>> dist = graph;                 // 辅助变量：距离矩阵
    for (int k = 0; k < n; ++k) {                     // 遍历控制变量：k
        for (int i = 0; i < n; ++i) {                 // 遍历控制变量：i
            for (int j = 0; j < n; ++j) {             // 遍历控制变量：j
                if (dist[i][k] != INT_MAX && dist[k][j] != INT_MAX) {
                    if (dist[i][k] + dist[k][j] < dist[i][j]) { // 节点操作：松弛
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }
    }
    return dist;
}
```

---

#### 优化算法4：拓扑排序（Kahn算法）

**优化说明**：利用**入度**性质，将 DAG 的节点线性排序，时间复杂度 O(V+E)。

##### 通用模板

**遍历框架（仅控制变量）**
```
计算入度 indegree
队列 q 存放所有入度为0的节点
while q not empty:
    u = q.front(); q.pop()
    输出 u
    for v in graph[u]:
        indegree[v]--
        if indegree[v] == 0: q.push(v)
```

**完整模板（含辅助变量）**
```
// 遍历框架：队列迭代
// 遍历控制变量：队列中的节点 u
// 辅助变量：indegree（入度数组）
vector<int> indegree(n, 0);
for (int u = 0; u < n; ++u) {
    for (int v : graph[u]) indegree[v]++;
}
queue<int> q;
for (int i = 0; i < n; ++i) if (indegree[i] == 0) q.push(i);
vector<int> res;
while (!q.empty()) {
    int u = q.front(); q.pop();                     // 遍历控制变量：u
    res.push_back(u);                               // 节点操作：输出
    for (int v : graph[u]) {
        indegree[v]--;                              // 节点操作：减少入度
        if (indegree[v] == 0) q.push(v);
    }
}
```

##### 数据结构：有向无环图

**遍历框架（仅控制变量）**
```cpp
vector<int> indegree(n, 0);
for (auto& edges : graph) {
    for (int v : edges) indegree[v]++;
}
queue<int> q;
for (int i = 0; i < n; ++i) if (indegree[i] == 0) q.push(i);
vector<int> res;
while (!q.empty()) {
    int u = q.front(); q.pop();
    res.push_back(u);
    for (int v : graph[u]) {
        indegree[v]--;
        if (indegree[v] == 0) q.push(v);
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> topologicalSort(int n, const vector<vector<int>>& graph) {
    vector<int> indegree(n, 0);                     // 辅助变量：入度数组
    for (int u = 0; u < n; ++u) {
        for (int v : graph[u]) indegree[v]++;
    }
    queue<int> q;                                   // 遍历控制变量：队列中的节点
    for (int i = 0; i < n; ++i) if (indegree[i] == 0) q.push(i);
    vector<int> res;                                // 辅助变量：结果存储
    while (!q.empty()) {
        int u = q.front(); q.pop();                 // 遍历控制变量：u
        res.push_back(u);                           // 节点操作：输出
        for (int v : graph[u]) {
            indegree[v]--;                          // 节点操作：更新入度
            if (indegree[v] == 0) q.push(v);
        }
    }
    if (res.size() != n) return {};                 // 存在环
    return res;
}
```


### 三、连通分量解

**定义**：解是图的极大连通子图（无向图）或强连通分量（有向图）。

#### 暴力枚举（DFS/BFS）

##### 通用模板（所有图结构）

**遍历框架（仅控制变量）**
```
visited = 全 false
for each node u:
    if not visited[u]:
        DFS/BFS 从 u 开始标记所有可达节点，计数+1
```

**完整模板（含辅助变量）**
```
// 遍历框架：外层循环 + DFS
// 遍历控制变量：节点 u
// 辅助变量：visited（状态标记），count（结果存储）
visited = [false]*n
count = 0
for u = 0 to n-1:
    if not visited[u]:
        count++
        dfs(u)   // 标记所有连通节点
```

##### 数据结构：图（无向）

**遍历框架（仅控制变量）**
```cpp
vector<bool> visited(n, false);
int count = 0;
for (int u = 0; u < n; ++u) {
    if (!visited[u]) {
        count++;
        dfs(u);
    }
}
```

**完整模板（含辅助变量）**
```cpp
int connectedComponents(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<bool> visited(n, false);                 // 辅助变量：状态标记
    int count = 0;                                   // 辅助变量：结果存储
    function<void(int)> dfs = [&](int u) {
        visited[u] = true;
        for (int v : graph[u]) {
            if (!visited[v]) dfs(v);
        }
    };
    for (int u = 0; u < n; ++u) {                    // 遍历控制变量：u
        if (!visited[u]) {
            count++;
            dfs(u);
        }
    }
    return count;
}
```

---

#### 优化算法1：并查集（动态连通性）

**优化说明**：利用**树形结构**维护集合，通过**路径压缩**和**按秩合并**，将动态连通性查询降为近似 O(α(n))。

##### 通用模板

**遍历框架（仅控制变量）**
```
初始化 parent[i]=i
合并：对于每条边 (u,v)，unite(u,v)
查询：find(u) == find(v)
```

**完整模板（含辅助变量）**
```
// 遍历控制变量：元素 x
// 辅助变量：parent, rank（类成员）
int find(x):
    if parent[x] != x: parent[x] = find(parent[x])
    return parent[x]
void unite(x, y):
    x = find(x); y = find(y)
    if x == y: return
    if rank[x] < rank[y]: parent[x] = y
    else if rank[x] > rank[y]: parent[y] = x
    else: parent[y] = x; rank[x]++
```

##### 数据结构：并查集

**遍历框架（仅控制变量）**
```cpp
class UnionFind {
    vector<int> parent, rank;
public:
    UnionFind(int n) {
        parent.resize(n);
        rank.resize(n, 0);
        for (int i = 0; i < n; ++i) parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    void unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return;
        if (rank[x] < rank[y]) parent[x] = y;
        else if (rank[x] > rank[y]) parent[y] = x;
        else { parent[y] = x; rank[x]++; }
    }
};
```

---

#### 优化算法2：强连通分量（Tarjan算法）

**优化说明**：利用**DFS**和**栈**，在 O(V+E) 内找出有向图的所有强连通分量。

##### 通用模板

**遍历框架（仅控制变量）**
```
dfn[u] = low[u] = ++time
stack.push(u); instack[u]=true
for v in graph[u]:
    if not dfn[v]:
        tarjan(v)
        low[u] = min(low[u], low[v])
    else if instack[v]:
        low[u] = min(low[u], dfn[v])
if dfn[u] == low[u]:
    pop stack until u, 记录一个SCC
```

**完整模板（含辅助变量）**
```
// 遍历框架：DFS
// 遍历控制变量：节点 u
// 辅助变量：dfn, low, stack, instack, sccId
```

##### 数据结构：有向图

**遍历框架（仅控制变量）**
```cpp
vector<int> dfn(n), low(n), sccId(n);
vector<bool> instack(n);
stack<int> st;
int time = 0, sccCount = 0;
function<void(int)> tarjan = [&](int u) {
    dfn[u] = low[u] = ++time;
    st.push(u); instack[u] = true;
    for (int v : graph[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (instack[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (dfn[u] == low[u]) {
        int x;
        do {
            x = st.top(); st.pop();
            instack[x] = false;
            sccId[x] = sccCount;
        } while (x != u);
        sccCount++;
    }
};
```

**完整模板（含辅助变量）**
```cpp
void tarjan(int u, const vector<vector<int>>& graph,
            vector<int>& dfn, vector<int>& low, vector<bool>& instack,
            stack<int>& st, int& time, int& sccCount, vector<int>& sccId) {
    dfn[u] = low[u] = ++time;
    st.push(u); instack[u] = true;
    for (int v : graph[u]) {
        if (!dfn[v]) {
            tarjan(v, graph, dfn, low, instack, st, time, sccCount, sccId);
            low[u] = min(low[u], low[v]);
        } else if (instack[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (dfn[u] == low[u]) {
        int x;
        do {
            x = st.top(); st.pop();
            instack[x] = false;
            sccId[x] = sccCount;
        } while (x != u);
        sccCount++;
    }
}
```


### 四、生成树解

**定义**：解是包含所有节点的树形子图，如最小生成树。

#### 暴力枚举（枚举所有生成树）

##### 通用模板

**遍历框架（仅控制变量）**
```
递归枚举所有边集，检查是否构成树（连通且无环）
```

**完整模板（含辅助变量）**
```
// 指数级，不实际使用
```

---

#### 优化算法1：最小生成树（Kruskal）

**优化说明**：利用**贪心**性质，按边权从小到大加入，用并查集判环，将指数级降为 O(m log m)。

##### 通用模板

**遍历框架（仅控制变量）**
```
排序边
for each edge (u,v,w) in sorted order:
    if find(u) != find(v):
        unite(u, v)
        total += w
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（按权值排序的边）
// 遍历控制变量：边
// 辅助变量：并查集（状态标记）
```

##### 数据结构：图（边集）

**遍历框架（仅控制变量）**
```cpp
sort(edges.begin(), edges.end());
UnionFind uf(n);
int total = 0, cnt = 0;
for (auto [w, u, v] : edges) {
    if (uf.find(u) != uf.find(v)) {
        uf.unite(u, v);
        total += w;
        if (++cnt == n-1) break;
    }
}
```

**完整模板（含辅助变量）**
```cpp
int kruskal(int n, vector<tuple<int,int,int>>& edges) {
    sort(edges.begin(), edges.end());                 // 预处理：排序
    UnionFind uf(n);                                   // 辅助变量：并查集
    int total = 0, cnt = 0;                           // 辅助变量：结果存储
    for (auto [w, u, v] : edges) {                    // 遍历控制变量：边
        if (uf.find(u) != uf.find(v)) {               // 节点操作：判断
            uf.unite(u, v);                           // 节点操作：合并
            total += w;
            if (++cnt == n-1) break;
        }
    }
    return total;
}
```

---

#### 优化算法2：最小生成树（Prim）

**优化说明**：利用**优先队列**，从任意节点开始，每次选择与当前集合相连的最小边加入，时间复杂度 O((V+E) log V)。

##### 通用模板

**遍历框架（仅控制变量）**
```
visited[start] = true
优先队列 pq 存放与当前集合相连的边
while pq not empty:
    (w, u) = pq.top(); pq.pop()
    if visited[u]: continue
    visited[u] = true; total += w
    for (v, w2) in graph[u]:
        if !visited[v]: pq.push({w2, v})
```

**完整模板（含辅助变量）**
```
// 遍历框架：优先队列迭代
// 遍历控制变量：优先队列中的边
// 辅助变量：visited（状态标记），pq（优先队列）
```

##### 数据结构：图（邻接表）

**遍历框架（仅控制变量）**
```cpp
vector<bool> visited(n, false);
priority_queue<pair<int,int>, vector<...>, greater<...>> pq;
visited[0] = true;
for (auto [v, w] : graph[0]) pq.push({w, v});
int total = 0, cnt = 1;
while (!pq.empty() && cnt < n) {
    auto [w, u] = pq.top(); pq.pop();
    if (visited[u]) continue;
    visited[u] = true;
    total += w;
    cnt++;
    for (auto [v, w2] : graph[u]) {
        if (!visited[v]) pq.push({w2, v});
    }
}
```

**完整模板（含辅助变量）**
```cpp
int prim(const vector<vector<pair<int,int>>>& graph) {
    int n = graph.size();
    vector<bool> visited(n, false);                 // 辅助变量：状态标记
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // 遍历控制变量
    visited[0] = true;
    for (auto [v, w] : graph[0]) pq.push({w, v});
    int total = 0, cnt = 1;
    while (!pq.empty() && cnt < n) {
        auto [w, u] = pq.top(); pq.pop();          // 遍历控制变量：u
        if (visited[u]) continue;
        visited[u] = true;                         // 节点操作：标记
        total += w;
        cnt++;
        for (auto [v, w2] : graph[u]) {            // 节点操作：扩展
            if (!visited[v]) pq.push({w2, v});
        }
    }
    return total;
}
```


### 五、匹配解

**定义**：解是边集，且没有公共顶点，如二分图最大匹配。

#### 暴力枚举（枚举所有匹配）

##### 通用模板

**遍历框架（仅控制变量）**
```
枚举所有边集，检查是否匹配，取最大
```

**完整模板（含辅助变量）**
```
// 指数级，不实际使用
```

---

#### 优化算法1：二分图最大匹配（匈牙利算法）

**优化说明**：利用**DFS回溯**，尝试为每个左部节点寻找匹配，若冲突则尝试让原匹配节点换人，时间复杂度 O(V·E)。

##### 通用模板

**遍历框架（仅控制变量）**
```
matchR = [-1]*nR
for u in 左部:
    visited = [false]*nR
    if dfs(u): ans++
bool dfs(u):
    for v in graph[u]:
        if not visited[v]:
            visited[v] = true
            if matchR[v]==-1 or dfs(matchR[v]):
                matchR[v] = u
                return true
    return false
```

**完整模板（含辅助变量）**
```
// 遍历框架：外层循环 + DFS
// 遍历控制变量：左部节点 u
// 辅助变量：matchR（匹配数组），visited（临时标记）
```

##### 数据结构：二分图（邻接表）

**遍历框架（仅控制变量）**
```cpp
vector<int> matchR(nR, -1);
int ans = 0;
for (int u = 0; u < nL; ++u) {
    vector<bool> visited(nR, false);
    if (dfs(u, visited, matchR)) ans++;
}
```

**完整模板（含辅助变量）**
```cpp
bool dfs(int u, vector<bool>& visited, vector<int>& matchR, const vector<vector<int>>& graph) {
    for (int v : graph[u]) {                         // 节点操作：遍历邻居
        if (!visited[v]) {
            visited[v] = true;
            if (matchR[v] == -1 || dfs(matchR[v], visited, matchR, graph)) {
                matchR[v] = u;                       // 节点操作：匹配
                return true;
            }
        }
    }
    return false;
}
int hungarian(int nL, int nR, const vector<vector<int>>& graph) {
    vector<int> matchR(nR, -1);                      // 辅助变量：匹配数组
    int ans = 0;
    for (int u = 0; u < nL; ++u) {                   // 遍历控制变量：u
        vector<bool> visited(nR, false);              // 辅助变量：临时标记
        if (dfs(u, visited, matchR, graph)) ans++;
    }
    return ans;
}
```


### 六、圈解

**定义**：解是简单环，如环检测、欧拉回路等。

#### 暴力枚举（枚举所有环）

##### 通用模板

**遍历框架（仅控制变量）**
```
DFS 枚举所有路径，记录环
```

**完整模板（含辅助变量）**
```
// 指数级，不实际使用
```

---

#### 优化算法1：有向图环检测（DFS三色标记）

**优化说明**：利用**DFS**和**状态标记**，在 O(V+E) 内检测环。

##### 通用模板

**遍历框架（仅控制变量）**
```
state[u] = 1 (访问中)
for v in graph[u]:
    if state[v] == 1: 有环
    elif state[v] == 0: dfs(v)
state[u] = 2 (已完成)
```

**完整模板（含辅助变量）**
```
// 遍历框架：DFS
// 遍历控制变量：节点 u
// 辅助变量：state（状态数组）
```

##### 数据结构：有向图

**遍历框架（仅控制变量）**
```cpp
vector<int> state(n, 0);
bool hasCycle = false;
function<void(int)> dfs = [&](int u) {
    state[u] = 1;
    for (int v : graph[u]) {
        if (state[v] == 1) hasCycle = true;
        else if (state[v] == 0) dfs(v);
    }
    state[u] = 2;
};
```

**完整模板（含辅助变量）**
```cpp
bool hasCycle(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> state(n, 0);                        // 辅助变量：状态数组（0未访问，1访问中，2完成）
    bool cycle = false;
    function<void(int)> dfs = [&](int u) {
        state[u] = 1;
        for (int v : graph[u]) {                    // 节点操作：遍历邻居
            if (state[v] == 1) cycle = true;
            else if (state[v] == 0) dfs(v);
        }
        state[u] = 2;
    };
    for (int i = 0; i < n; ++i) {                   // 遍历控制变量：i
        if (state[i] == 0) dfs(i);
    }
    return cycle;
}
```

---

#### 优化算法2：欧拉回路（Hierholzer算法）

**优化说明**：利用**DFS**和**栈**，在 O(E) 内找到经过每条边恰好一次的回路。

##### 通用模板

**遍历框架（仅控制变量）**
```
stack st
st.push(start)
while st not empty:
    u = st.top()
    if graph[u] not empty:
        v = graph[u].back(); graph[u].pop_back()
        st.push(v)
    else:
        path.push_back(u); st.pop()
reverse(path)
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（栈）
// 遍历控制变量：栈顶节点 u
// 辅助变量：stack, path
```

##### 数据结构：图（邻接表，需支持删除边）

**遍历框架（仅控制变量）**
```cpp
stack<int> st;
st.push(start);
vector<int> path;
while (!st.empty()) {
    int u = st.top();
    if (!graph[u].empty()) {
        int v = graph[u].back();
        graph[u].pop_back();
        st.push(v);
    } else {
        path.push_back(u);
        st.pop();
    }
}
reverse(path.begin(), path.end());
```

**完整模板（含辅助变量）**
```cpp
vector<int> eulerCircuit(vector<vector<int>>& graph, int start) {
    stack<int> st;                                   // 遍历控制变量：栈中的节点
    vector<int> path;                                // 辅助变量：结果路径
    st.push(start);
    while (!st.empty()) {
        int u = st.top();                            // 遍历控制变量：u
        if (!graph[u].empty()) {
            int v = graph[u].back();                 // 节点操作：取最后一条边
            graph[u].pop_back();                    // 节点操作：删除边
            st.push(v);
        } else {
            path.push_back(u);
            st.pop();
        }
    }
    reverse(path.begin(), path.end());
    return path;
}
```


### 七、流解

**定义**：解是满足容量约束和流量守恒的边流量分配，如最大流。

#### 优化算法1：最大流（Dinic算法）

**优化说明**：利用**BFS分层**和**DFS阻塞流**，在 O(E·V²) 内找到最大流，实际效率更高。

##### 通用模板

**遍历框架（仅控制变量）**
```
while BFS 建立层次图:
    重置当前弧指针
    while (f = DFS(s, t, INF)) > 0:
        flow += f
```

**完整模板（含辅助变量）**
```
// 遍历框架：BFS + 多次DFS
// 遍历控制变量：BFS中的队列节点，DFS中的当前节点
// 辅助变量：level（层次数组），iter（当前弧）
```

##### 数据结构：有向图（带容量）

**遍历框架（仅控制变量）**
```cpp
class Dinic {
    struct Edge { int to, rev; long long cap; };
    vector<vector<Edge>> g;
    vector<int> level, iter;
    int n;
public:
    Dinic(int n) : n(n), g(n), level(n), iter(n) {}
    void addEdge(int from, int to, long long cap) {
        g[from].push_back({to, (int)g[to].size(), cap});
        g[to].push_back({from, (int)g[from].size()-1, 0});
    }
    bool bfs(int s, int t) {
        fill(level.begin(), level.end(), -1);
        queue<int> q;
        level[s] = 0; q.push(s);
        while (!q.empty()) {
            int v = q.front(); q.pop();
            for (auto& e : g[v]) {
                if (e.cap > 0 && level[e.to] < 0) {
                    level[e.to] = level[v] + 1;
                    q.push(e.to);
                }
            }
        }
        return level[t] >= 0;
    }
    long long dfs(int v, int t, long long f) {
        if (v == t) return f;
        for (int& i = iter[v]; i < g[v].size(); ++i) {
            Edge& e = g[v][i];
            if (e.cap > 0 && level[v] < level[e.to]) {
                long long d = dfs(e.to, t, min(f, e.cap));
                if (d > 0) {
                    e.cap -= d;
                    g[e.to][e.rev].cap += d;
                    return d;
                }
            }
        }
        return 0;
    }
    long long maxFlow(int s, int t) {
        long long flow = 0;
        while (bfs(s, t)) {
            fill(iter.begin(), iter.end(), 0);
            long long f;
            while ((f = dfs(s, t, 1e18)) > 0) flow += f;
        }
        return flow;
    }
};
```


### 八、博弈状态解

**定义**：解是游戏状态图上的胜态或 SG 值。

#### 优化算法1：SG函数（有向图游戏）

**优化说明**：利用**DFS + 记忆化**，计算每个状态的 SG 值，用于组合游戏。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(u):
    if sg[u] != -1: return sg[u]
    set<int> s
    for v in graph[u]:
        s.insert(sg(v))
    sg[u] = mex(s)
    return sg[u]
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归 + 记忆化
// 遍历控制变量：状态 u
// 辅助变量：sg（记忆化数组）
```

##### 数据结构：有向图（游戏状态）

**遍历框架（仅控制变量）**
```cpp
vector<int> sg(n, -1);
function<int(int)> getSG = [&](int u) {
    if (sg[u] != -1) return sg[u];
    vector<bool> vis(100, false); // 根据出度调整大小
    for (int v : graph[u]) {
        vis[getSG(v)] = true;
    }
    for (int i = 0; ; ++i) if (!vis[i]) { sg[u] = i; break; }
    return sg[u];
};
```

**完整模板（含辅助变量）**
```cpp
int mex(const vector<bool>& vis) {
    for (int i = 0; ; ++i) if (!vis[i]) return i;
}
vector<int> computeSG(const vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> sg(n, -1);                           // 辅助变量：记忆化数组
    function<int(int)> dfs = [&](int u) -> int {
        if (sg[u] != -1) return sg[u];
        vector<bool> vis(100, false);                // 临时变量
        for (int v : graph[u]) {                      // 遍历控制变量：v
            vis[dfs(v)] = true;                       // 节点操作：标记后继SG值
        }
        sg[u] = mex(vis);                             // 节点操作：计算mex
        return sg[u];
    };
    for (int i = 0; i < n; ++i) dfs(i);               // 遍历控制变量：i
    return sg;
}
```


## 六、总结

图形解空间是算法设计中极具挑战性的领域，涵盖从基础遍历到复杂优化问题的丰富内容。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种图形算法。本框架覆盖了图的遍历、最短路径、连通分量、生成树、匹配、圈、网络流、博弈等典型算法，并展示了每种算法如何从暴力枚举出发，利用图的结构特性（连通性、权值、环）进行优化，实现从指数级到多项式级的跨越。掌握这套框架，能够帮助你快速设计出清晰、高效的图形算法。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。