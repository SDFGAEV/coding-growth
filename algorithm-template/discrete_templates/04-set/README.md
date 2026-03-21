# 集合解空间：算法认知框架

> 本文档是“算法认知框架”系列的第四部分，聚焦于**集合解空间**的算法设计与实现。

## 核心理念

### 集合解空间定义
集合解空间由**无序元素**构成，元素之间没有固定的顺序或连接关系。解可以是单个元素、元素的子集、两个集合之间的关系，或者集合的划分。

**基础元素**：元素（可以是整数、字符、对象等），元素之间仅具有“属于”关系。

### 算法本质（在集合解空间中的体现）
**算法 = 在解空间上的聪明穷举 + 辅助变量优化**，对于集合解空间：
- **无遗漏**：必须考虑所有可能的解（如所有子集、所有划分等）
- **无冗余**：利用元素的标识、哈希值、位运算等特性避免重复计算

### 解空间的人为构造
解空间本身是问题的所有可能解的集合，其结构是抽象的。但在设计算法时，我们可以人为地为解空间赋予一种便于遍历的结构。**同一问题的解空间可以有多种不同的构造方式，选择不同的结构会导致不同的算法策略。**

例如：
- **子集解**：原始解空间是所有子集（幂集），是指数级的。我们可以将其组织成**决策树**（每个元素选或不选），通过回溯深度优先遍历；也可以组织成**DAG**（如背包问题），通过动态规划避免重复计算。
- **划分解**：可以组织成**树形结构**（并查集），通过路径压缩和按秩合并实现高效操作。

因此，在分析集合解空间时，我们关注的是将解空间构造为便于遍历的结构后的搜索和优化。


## 一、集合解空间的解的形态

集合解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构（人为构造） |
|:---|:---|:---|:---|
| **单元素解** | 元素的存在性 | 哈希表查找、成员判断 | 元素集合（线性） |
| **子集解** | 原集合的一个子集 | 子集枚举、子集和、背包问题（0-1、完全、多重、分组） | 幂集（决策树/DAG） |
| **关系解** | 两个集合的交集、并集、差集 | 集合运算 | 集合对 |
| **划分解** | 将集合划分为若干组（等价类） | 并查集 | 树形（森林） |


## 二、遍历框架与遍历控制变量

集合解空间的遍历框架多样，取决于解的形态。

| 解的形态 | 数据结构 | 遍历框架（模板） | 遍历控制变量（模板） |
|:---|:---|:---|:---|
| **单元素解** | 数组 | `for x in 集合` | 元素 `x` |
| **单元素解** | 哈希表 | `遍历键值` | 键 `key` |
| **子集解** | 数组 | 回溯（递归） | 当前索引 `pos`，当前路径 `path` |
| **子集解** | 位运算 | 迭代（掩码） | 掩码 `mask` |
| **子集解** | DP | 双层循环 | 物品 `i`，容量 `j` |
| **关系解** | 哈希表 | 遍历 | 元素 `x` |
| **划分解** | 并查集 | 迭代（路径压缩） | 元素 `x` |


## 三、节点操作

节点操作是在每个被访问的“节点”上执行的具体逻辑。在集合解空间中，“节点”可以是单个元素、一个子集或一个状态。

| 解的形态 | 算法 | 节点操作（模板） | 时机 |
|:---|:---|:---|:---|
| **单元素解** | 哈希查找 | 检查是否存在，插入 | 遍历输入时 |
| **子集解** | 回溯 | 记录当前子集，递归选择 | 前序（做选择），后序（撤销） |
| **子集解** | 位运算 | 根据掩码构建子集 | 迭代时 |
| **子集解** | DP | 更新 DP 表 | 按物品循环 |
| **关系解** | 集合运算 | 插入、查找 | 遍历时 |
| **划分解** | 并查集 | 查找根，合并 | 操作时 |


## 四、辅助变量

辅助变量用于记录状态、剪枝、缓存结果，保证无冗余。

| 解的形态 | 算法 | 辅助变量（模板） | 功能分类 |
|:---|:---|:---|:---|
| **单元素解** | 哈希查找 | `unordered_set` / `unordered_map` | 优化缓存 |
| **子集解** | 回溯 | `vector<int> path`，结果集 `res` | 路径记录、结果存储 |
| **子集解** | 位运算 | 结果集 `res` | 结果存储 |
| **子集解** | 背包 DP | `vector<int> dp` | 优化缓存 |
| **关系解** | 集合运算 | `unordered_set` | 优化缓存 |
| **划分解** | 并查集 | `vector<int> parent, rank` | 状态标记 |

### 变量生命周期与存放位置
集合算法中，辅助变量常为函数局部变量或类的成员。

| 类型 | 描述 | 示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次函数执行中存在 | 循环索引、临时集合 | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 回溯中的 `path`, `used` | 递归参数（引用）或 lambda 捕获 |
| **类内长期** | 跨多次公共方法调用持久 | 并查集的 `parent` | 类的成员变量 |


## 五、各子类算法详细模板

### 一、单元素解

**定义**：解是单个元素的存在性，即需要判断某个元素是否属于集合。

#### 暴力枚举（线性查找）

##### 通用模板（所有集合）

**遍历框架（仅控制变量）**
```
for each element in 集合:
    节点操作：比较当前元素与目标
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：元素 x
// 辅助变量：无
for x in 集合:
    if x == target: return true
return false
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
for (int x : nums) {
    // 节点操作：比较 x 与 target
}
```

**完整模板（含辅助变量）**
```cpp
bool linearSearch(const vector<int>& nums, int target) {
    for (int x : nums) {                         // 遍历控制变量：x
        if (x == target) return true;            // 节点操作：比较
    }
    return false;
}
// 辅助变量：无
```

---

#### 优化算法1：哈希表查找

**优化说明**：利用**哈希表**将查找从 O(n) 降为 O(1)，适用于多次查找。

##### 通用模板

**遍历框架（仅控制变量）**
```
构建哈希表：for x in 集合: hash.insert(x)
查找：hash.contains(target)
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（构建时）
// 遍历控制变量：x
// 辅助变量：hash（优化缓存）
hash = 空哈希表
for x in 集合:
    hash.insert(x)                    // 节点操作：插入
return hash.contains(target)          // 节点操作：查找
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
unordered_set<int> s;
for (int x : nums) {
    s.insert(x);
}
```

**完整模板（含辅助变量）**
```cpp
bool hashSearch(const vector<int>& nums, int target) {
    unordered_set<int> s;                         // 辅助变量：优化缓存
    for (int x : nums) {                          // 遍历控制变量：x
        s.insert(x);                              // 节点操作：插入
    }
    return s.count(target);                       // 节点操作：查找
}
```


### 二、子集解

**定义**：解是原集合的一个子集，即需要从集合中选出若干元素。

#### 暴力枚举（回溯）

##### 通用模板（所有集合）

**遍历框架（仅控制变量）**
```
递归函数(pos, path):
    if pos == n:
        记录 path
        return
    不选当前元素：递归(pos+1, path)
    选当前元素：path.push(nums[pos]); 递归(pos+1, path); path.pop()
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（决策树）
// 遍历控制变量：pos（当前索引），path（当前路径）
// 辅助变量：res（结果存储）
def dfs(pos, path):
    if pos == n:
        res.add(path)               // 节点操作：记录结果
        return
    dfs(pos+1, path)                // 节点操作：不选
    path.append(nums[pos])
    dfs(pos+1, path)                // 节点操作：选
    path.pop()
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
void dfs(int pos, vector<int>& path, vector<vector<int>>& res) {
    if (pos == n) {
        res.push_back(path);
        return;
    }
    dfs(pos+1, path, res);
    path.push_back(nums[pos]);
    dfs(pos+1, path, res);
    path.pop_back();
}
```

**完整模板（含辅助变量）**
```cpp
vector<vector<int>> subsets(const vector<int>& nums) {
    vector<vector<int>> res;                     // 辅助变量：结果存储
    vector<int> path;                            // 辅助变量：路径记录
    function<void(int)> dfs = [&](int pos) {     // 遍历控制变量：pos
        if (pos == nums.size()) {
            res.push_back(path);                 // 节点操作：记录叶子
            return;
        }
        dfs(pos + 1);                            // 节点操作：不选
        path.push_back(nums[pos]);               // 节点操作：做选择
        dfs(pos + 1);                            // 节点操作：递归
        path.pop_back();                         // 节点操作：撤销选择
    };
    dfs(0);
    return res;
}
```

---

#### 优化算法1：位运算枚举

**优化说明**：利用**位掩码**表示子集，将指数级枚举简化为整数迭代，适用于 n ≤ 20。

##### 通用模板

**遍历框架（仅控制变量）**
```
for mask = 0 to (1<<n)-1:
    根据 mask 的二进制位构建子集
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（掩码）
// 遍历控制变量：mask
// 辅助变量：res（结果存储）
for mask = 0 to (1<<n)-1:
    subset = []
    for i = 0 to n-1:
        if mask & (1<<i): subset.append(nums[i])   // 节点操作：构建子集
    res.add(subset)
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
for (int mask = 0; mask < (1 << n); ++mask) {
    // 根据 mask 构建子集
}
```

**完整模板（含辅助变量）**
```cpp
vector<vector<int>> subsetsBit(const vector<int>& nums) {
    int n = nums.size();
    vector<vector<int>> res;                      // 辅助变量：结果存储
    for (int mask = 0; mask < (1 << n); ++mask) { // 遍历控制变量：mask
        vector<int> subset;
        for (int i = 0; i < n; ++i) {             // 遍历控制变量：i
            if (mask >> i & 1) {                  // 节点操作：检查位
                subset.push_back(nums[i]);         // 节点操作：构建子集
            }
        }
        res.push_back(subset);
    }
    return res;
}
```

---

#### 优化算法2：子集和（0-1背包）

**优化说明**：利用**动态规划**，将指数级枚举降为 O(n·W)，适用于求子集和可行性或最大价值。

##### 通用模板

**遍历框架（仅控制变量）**
```
dp[0] = true
for x in nums:
    for j = W down to x:
        if dp[j - x]: dp[j] = true
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环（物品 + 容量）
// 遍历控制变量：x, j
// 辅助变量：dp（优化缓存）
dp[0] = true
for x in nums:
    for j = W down to x:
        if dp[j - x]: dp[j] = true          // 节点操作：状态转移
return dp[W]
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<bool> dp(W+1, false);
dp[0] = true;
for (int x : nums) {
    for (int j = W; j >= x; --j) {
        if (dp[j - x]) dp[j] = true;
    }
}
```

**完整模板（含辅助变量）**
```cpp
bool subsetSum(const vector<int>& nums, int target) {
    vector<bool> dp(target + 1, false);       // 辅助变量：优化缓存
    dp[0] = true;
    for (int x : nums) {                      // 遍历控制变量：x
        for (int j = target; j >= x; --j) {   // 遍历控制变量：j
            if (dp[j - x]) {                  // 节点操作：状态转移
                dp[j] = true;
            }
        }
    }
    return dp[target];
}
```

---

#### 优化算法3：0-1背包（最大价值）

**优化说明**：利用**动态规划**，将指数级枚举降为 O(n·W)，求最大价值。

##### 通用模板

**遍历框架（仅控制变量）**
```
dp[j] = 0
for i = 0 to n-1:
    for j = W down to w[i]:
        dp[j] = max(dp[j], dp[j - w[i]] + v[i])
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：dp（优化缓存）
dp[0..W] = 0
for i = 0 to n-1:
    for j = W down to w[i]:
        dp[j] = max(dp[j], dp[j - w[i]] + v[i])   // 节点操作：状态转移
return dp[W]
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> dp(W+1, 0);
for (int i = 0; i < n; ++i) {
    for (int j = W; j >= w[i]; --j) {
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

**完整模板（含辅助变量）**
```cpp
int knapsack(const vector<int>& w, const vector<int>& v, int W) {
    int n = w.size();
    vector<int> dp(W + 1, 0);                  // 辅助变量：优化缓存
    for (int i = 0; i < n; ++i) {              // 遍历控制变量：i
        for (int j = W; j >= w[i]; --j) {      // 遍历控制变量：j
            dp[j] = max(dp[j], dp[j - w[i]] + v[i]); // 节点操作：状态转移
        }
    }
    return dp[W];
}
```

---

#### 优化算法4：完全背包

**优化说明**：每个物品可选无限次，将容量遍历方向改为正序。

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n-1:
    for j = w[i] to W:
        dp[j] = max(dp[j], dp[j - w[i]] + v[i])
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环（正序容量）
// 遍历控制变量：i, j
// 辅助变量：dp（优化缓存）
dp[0..W] = 0
for i = 0 to n-1:
    for j = w[i] to W:
        dp[j] = max(dp[j], dp[j - w[i]] + v[i])   // 节点操作：状态转移
return dp[W]
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> dp(W+1, 0);
for (int i = 0; i < n; ++i) {
    for (int j = w[i]; j <= W; ++j) {
        dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

**完整模板（含辅助变量）**
```cpp
int completePack(const vector<int>& w, const vector<int>& v, int W) {
    int n = w.size();
    vector<int> dp(W + 1, 0);                  // 辅助变量：优化缓存
    for (int i = 0; i < n; ++i) {              // 遍历控制变量：i
        for (int j = w[i]; j <= W; ++j) {      // 遍历控制变量：j
            dp[j] = max(dp[j], dp[j - w[i]] + v[i]); // 节点操作：状态转移
        }
    }
    return dp[W];
}
```


### 三、关系解

**定义**：解是两个集合之间的关系，如交集、并集、差集。

#### 暴力枚举（双重循环）

##### 通用模板（所有集合）

**遍历框架（仅控制变量）**
```
for a in 集合A:
    for b in 集合B:
        节点操作：比较 a 与 b
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：a, b
// 辅助变量：结果集
for a in A:
    for b in B:
        if a == b: res.add(a)        // 节点操作：判断
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> res;
for (int a : A) {
    for (int b : B) {
        if (a == b) res.push_back(a);
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> intersectionBrute(const vector<int>& A, const vector<int>& B) {
    vector<int> res;                                 // 辅助变量：结果存储
    for (int a : A) {                                // 遍历控制变量：a
        for (int b : B) {                            // 遍历控制变量：b
            if (a == b) {                            // 节点操作：比较
                res.push_back(a);
                break;
            }
        }
    }
    return res;
}
```

---

#### 优化算法1：哈希表交集

**优化说明**：利用**哈希表**将查找从 O(n) 降为 O(1)，整体 O(n+m)。

##### 通用模板

**遍历框架（仅控制变量）**
```
构建哈希表：for x in A: hash.insert(x)
遍历 B：for x in B: if hash.contains(x): res.add(x)
```

**完整模板（含辅助变量）**
```
// 遍历框架：两次单层循环
// 遍历控制变量：x
// 辅助变量：hash（优化缓存），res（结果存储）
hash = 空哈希表
for x in A:
    hash.insert(x)                     // 节点操作：插入
for x in B:
    if hash.contains(x): res.add(x)    // 节点操作：查找
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
unordered_set<int> s;
for (int x : A) s.insert(x);
vector<int> res;
for (int x : B) {
    if (s.count(x)) res.push_back(x);
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> intersection(const vector<int>& A, const vector<int>& B) {
    unordered_set<int> s;                            // 辅助变量：优化缓存
    for (int x : A) s.insert(x);                     // 遍历控制变量：x
    vector<int> res;                                 // 辅助变量：结果存储
    for (int x : B) {                                // 遍历控制变量：x
        if (s.count(x)) res.push_back(x);            // 节点操作：查找
    }
    return res;
}
```


### 四、划分解

**定义**：解是将集合划分为若干组（等价类），如并查集维护的连通分量。

#### 优化算法1：并查集（Union-Find）

**优化说明**：利用**树形结构**维护集合，通过**路径压缩**和**按秩合并**，将动态划分操作降为近似 O(α(n))。

##### 通用模板

**遍历框架（仅控制变量）**
```
查找：if parent[x] != x: parent[x] = find(parent[x]); return parent[x]
合并：x = find(x); y = find(y); if x != y: parent[x] = y (按秩优化)
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（查找时路径压缩）
// 遍历控制变量：元素 x
// 辅助变量：parent, rank（类成员）
int find(x):
    if parent[x] != x: parent[x] = find(parent[x])   // 节点操作：路径压缩
    return parent[x]
void unite(x, y):
    x = find(x); y = find(y)
    if x == y: return
    if rank[x] < rank[y]: parent[x] = y              // 节点操作：按秩合并
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


## 六、总结

集合解空间是算法设计中最基础的应用之一，涉及从简单查找到复杂子集优化的广泛问题。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种集合算法。本框架涵盖了哈希查找、子集枚举、背包问题、集合运算、并查集等典型算法，并展示了每种算法如何从暴力枚举出发，利用集合的特性（哈希、重叠子问题、树形结构）进行优化，实现从指数级到多项式级的跨越。掌握这套框架，能够帮助你快速设计出清晰、高效的集合算法。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。