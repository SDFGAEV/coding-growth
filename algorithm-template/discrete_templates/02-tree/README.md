# 树形解空间：算法认知框架

> 本文档是“算法认知框架”系列的第二部分，聚焦于**树形解空间**的算法设计与实现。

## 核心理念

### 树形解空间定义
树形解空间由**树节点**构成，节点之间具有**一对多的层次关系**。解可以是树中的单个节点、一条路径、一棵子树或某种特定的树结构。

**基础元素**：树节点（二叉树、多叉树、决策树等）。

### 算法本质（在树形解空间中的体现）
**算法 = 在解空间上的聪明穷举 + 辅助变量优化**，对于树形解空间：
- **无遗漏**：必须考虑所有可能的解（如所有节点、所有路径、所有子树等）
- **无冗余**：利用树的结构特性（递归性质、子树独立性）避免重复计算

### 解空间的人为构造
解空间本身是问题的所有可能解的集合，其结构是抽象的。但在设计算法时，我们可以人为地为解空间赋予一种便于遍历的结构。**同一问题的解空间可以有多种不同的构造方式，选择不同的结构会导致不同的算法策略。**

例如：
- **排列解**：原始解空间是指数级的（所有排列），我们可以将其组织成**决策树**（树形结构），通过回溯深度优先遍历，此时解空间构造为树形。
- **树上的路径解**：树中任意两点间路径是唯一的，但所有路径集合仍可组织成树形结构（如通过根节点分解），利用后序遍历合并信息。

因此，在分析树形解空间时，我们关注的是将解空间构造为树形结构后的遍历和优化。


## 一、树形解空间的解的形态

树形解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构（人为构造） |
|:---|:---|:---|:---|
| **单点解** | 单个节点 | 节点查找、节点统计、二叉树遍历 | 节点集合（线性结构） |
| **路径解** | 节点序列，相邻节点有边相连 | 根到叶子路径、任意两点路径、直径、LCA、**全排列**、**组合** | 路径集合（决策树） |
| **子树解** | 以某节点为根的子树（含后代） | 子树和、树的重心、树形DP | 子树集合（递归树） |
| **结构解** | 树的特定结构或性质 | BST操作、平衡树、树链剖分、动态树、并查集 | 树结构空间 |

**说明**：路径解包括物理树上的路径和**决策树上的路径**（如全排列、组合等），后者解空间是指数级的，但通过回溯或动态规划求解。


## 二、遍历框架与遍历控制变量

树形解空间的遍历框架主要由**递归**主导，也可使用**迭代（栈、队列）**模拟。

| 解的形态 | 数据结构 | 遍历框架（模板） | 遍历控制变量（模板） |
|:---|:---|:---|:---|
| **单点解（遍历）** | 二叉树 | 递归（前序/中序/后序） | 当前节点指针 `root` |
| **单点解（层序）** | 二叉树 | 迭代（队列） | 队列中节点 |
| **路径解（根到叶子）** | 树 | 递归（回溯） | 当前节点，当前路径 `path` |
| **路径解（LCA）** | 树 | 递归（后序） | 当前节点 |
| **路径解（直径）** | 树 | 递归（后序） | 当前节点，左右子树深度 |
| **子树解（树形DP）** | 树 | 递归（后序） | 当前节点 |
| **结构解（BST查找）** | BST | 递归/迭代 | 当前节点 |
| **结构解（并查集查找）** | 并查集 | 递归/迭代 | 元素 `x` |


## 三、节点操作

节点操作是在每个被访问的节点上执行的具体逻辑。根据节点分支数，操作时机可分为：

### 1. 二叉树节点（双分支）
- **前序**：刚进入节点，左右子节点均未处理
- **中序**：左子节点处理完毕，右子节点尚未处理（仅当有固定顺序时）
- **后序**：左右子节点均已处理完毕，即将离开节点

### 2. 多叉节点（多分支）
- **前序**：刚进入节点
- **循环内**（对于每个子节点）：
  - 进入子节点前：剪枝、准备
  - 递归进入子节点
  - 离开子节点后：清理、状态恢复
- **后序**：所有子节点处理完毕

| 解的形态 | 数据结构 | 节点操作（模板） | 时机 |
|:---|:---|:---|:---|
| **单点解** | 二叉树 | 访问节点值 | 前/中/后序 |
| **路径解（根到叶子）** | 树 | 记录路径，叶子时保存 | 前序（做选择），后序（撤销） |
| **路径解（直径）** | 树 | 更新直径，返回深度 | 后序 |
| **子树解（树形DP）** | 树 | 合并子节点结果 | 后序 |
| **结构解（BST查找）** | BST | 比较键值 | 前序 |
| **结构解（并查集）** | 并查集 | 路径压缩，合并 | 查找时 |


## 四、辅助变量

辅助变量用于记录状态、剪枝、缓存结果，保证无冗余。

| 解的形态 | 算法 | 辅助变量（模板） | 功能分类 |
|:---|:---|:---|:---|
| **单点解** | 二叉树遍历 | `vector<int> res` | 结果存储 |
| **路径解** | 根到叶子路径 | `vector<int> path`（递归参数） | 路径记录 |
| **路径解** | 直径 | `int diameter`（全局） | 结果存储 |
| **子树解** | 树形DP | 返回值（如 `pair<int,int>`） | 优化缓存 |
| **结构解** | 并查集 | `vector<int> parent, rank` | 状态标记 |
| **结构解** | 树链剖分 | `dfn, top, size, son` 数组 | 优化缓存 |
| **路径解（决策树）** | 全排列 | `vector<bool> used`, `vector<int> path`, `res` | 状态标记、路径记录、结果存储 |

### 变量生命周期与存放位置
树形递归算法中，辅助变量常作为**递归参数（引用）**传递，或作为**全局/成员变量**（需注意重置）。

| 类型 | 描述 | 树形算法示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次递归调用中存在 | 循环变量、临时值 | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 结果集 `res`、当前路径 `path` | 递归参数（引用）或 lambda 捕获 |
| **类内长期** | 跨多次公共方法调用持久 | 平衡树的根节点、并查集的 `parent` | 类的成员变量 |


## 五、各子类算法详细模板

### 一、单点解

**定义**：解是树中的单个节点，即需要访问或统计节点信息。

#### 暴力枚举（遍历所有节点）

##### 通用模板（所有树形结构）

**遍历框架（仅控制变量）**
```
递归函数(node):
    if node == null: return
    节点操作：处理当前节点
    递归进入子节点
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（深度优先）
// 遍历控制变量：当前节点指针
// 辅助变量：结果集（可选）
void dfs(node):
    if node == null: return
    // 节点操作：处理当前节点（前序）
    for child in node.children:
        dfs(child)
```

##### 数据结构：二叉树（前序遍历）

**遍历框架（仅控制变量）**
```cpp
void preorder(TreeNode* root) {
    if (!root) return;
    // 节点操作：处理 root->val
    preorder(root->left);
    preorder(root->right);
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> res;                                 // 辅助变量：结果存储
    function<void(TreeNode*)> dfs = [&](TreeNode* node) { // 遍历控制变量：node
        if (!node) return;
        res.push_back(node->val);                    // 节点操作：访问
        dfs(node->left);
        dfs(node->right);
    };
    dfs(root);
    return res;
}
```

##### 数据结构：二叉树（层序遍历）

**遍历框架（仅控制变量）**
```cpp
queue<TreeNode*> q;
q.push(root);
while (!q.empty()) {
    int size = q.size();
    for (int i = 0; i < size; ++i) {
        TreeNode* node = q.front(); q.pop();
        // 节点操作：访问 node->val
        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;                         // 辅助变量：结果存储
    if (!root) return res;
    queue<TreeNode*> q;                              // 遍历控制变量：队列中的节点
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        vector<int> level;                           // 辅助变量：临时存储当前层
        for (int i = 0; i < size; ++i) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);              // 节点操作：访问
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

---

### 二、路径解

**定义**：解是树中若干节点组成的序列，相邻节点有边相连。

#### 暴力枚举（枚举所有根到叶子路径）

##### 通用模板（所有树形结构）

**遍历框架（仅控制变量）**
```
递归函数(node, path):
    if node == null: return
    path.push(node)
    if node is leaf:
        记录 path
    else:
        for child in node.children:
            递归(child, path)
    path.pop()
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（回溯）
// 遍历控制变量：当前节点，当前路径 path
// 辅助变量：结果集 res
void dfs(node, path):
    if node == null: return
    path.push(node)                         // 节点操作：做选择
    if node is leaf:
        res.add(path)                       // 节点操作：记录叶子路径
    else:
        for child in node.children:
            dfs(child, path)
    path.pop()                              // 节点操作：撤销选择
```

##### 数据结构：二叉树（根到叶子路径）

**遍历框架（仅控制变量）**
```cpp
void dfs(TreeNode* node, vector<int>& path, vector<vector<int>>& res) {
    if (!node) return;
    path.push_back(node->val);
    if (!node->left && !node->right) {
        res.push_back(path);
    } else {
        dfs(node->left, path, res);
        dfs(node->right, path, res);
    }
    path.pop_back();
}
```

**完整模板（含辅助变量）**
```cpp
vector<vector<int>> binaryTreePaths(TreeNode* root) {
    vector<vector<int>> res;                     // 辅助变量：结果存储
    vector<int> path;                            // 辅助变量：路径记录
    function<void(TreeNode*)> dfs = [&](TreeNode* node) { // 遍历控制变量：node
        if (!node) return;
        path.push_back(node->val);               // 节点操作：做选择
        if (!node->left && !node->right) {
            res.push_back(path);                 // 节点操作：记录叶子路径
        } else {
            dfs(node->left);
            dfs(node->right);
        }
        path.pop_back();                         // 节点操作：撤销选择
    };
    dfs(root);
    return res;
}
```

---

#### 优化算法1：树的直径（最长路径长度）

**优化说明**：利用**后序遍历**，对每个节点计算左右子树的最大深度，更新经过该节点的最长路径，最后返回全局最大值。时间复杂度 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(node):
    if node == null: return 0
    left = 递归(node.left)
    right = 递归(node.right)
    diameter = max(diameter, left + right)   // 节点操作：更新全局答案
    return max(left, right) + 1
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（后序）
// 遍历控制变量：当前节点
// 辅助变量：全局直径
int diameter = 0
int dfs(node):
    if node == null: return 0
    left = dfs(node.left)
    right = dfs(node.right)
    diameter = max(diameter, left + right)   // 节点操作：更新结果
    return max(left, right) + 1
```

##### 数据结构：二叉树

**遍历框架（仅控制变量）**
```cpp
int dfs(TreeNode* node) {
    if (!node) return 0;
    int left = dfs(node->left);
    int right = dfs(node->right);
    // 更新直径
    return max(left, right) + 1;
}
```

**完整模板（含辅助变量）**
```cpp
int diameterOfBinaryTree(TreeNode* root) {
    int diameter = 0;                                // 辅助变量：结果存储
    function<int(TreeNode*)> dfs = [&](TreeNode* node) -> int { // 遍历控制变量：node
        if (!node) return 0;
        int left = dfs(node->left);
        int right = dfs(node->right);
        diameter = max(diameter, left + right);       // 节点操作：更新直径
        return max(left, right) + 1;
    };
    dfs(root);
    return diameter;
}
```

---

#### 优化算法2：最近公共祖先（LCA）

**优化说明**：利用**后序遍历**，递归查找 p 和 q 的位置，若当前节点就是其中一个或左右子树各有一个，则当前节点即为 LCA。时间复杂度 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(node, p, q):
    if node == null: return null
    if node == p or node == q: return node
    left = 递归(node.left)
    right = 递归(node.right)
    if left and right: return node
    return left ? left : right
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（后序）
// 遍历控制变量：当前节点
// 辅助变量：无
TreeNode* lca(node):
    if node == null: return null
    if node == p or node == q: return node
    left = lca(node.left)
    right = lca(node.right)
    if left and right: return node          // 节点操作：找到 LCA
    return left ? left : right
```

##### 数据结构：二叉树

**遍历框架（仅控制变量）**
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* left = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left && right) return root;
    return left ? left : right;
}
```

**完整模板（含辅助变量）**
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;   // 递归出口
    TreeNode* left = lowestCommonAncestor(root->left, p, q);   // 遍历控制变量：左子树
    TreeNode* right = lowestCommonAncestor(root->right, p, q); // 遍历控制变量：右子树
    if (left && right) return root;                       // 节点操作：当前节点为 LCA
    return left ? left : right;                           // 节点操作：返回非空的那一边
}
// 辅助变量：无
```

---

#### 优化算法3：全排列（决策树路径）

**优化说明**：输入是线性结构（数组），将解空间构造为**决策树**，通过回溯深度优先遍历所有路径，利用 `used` 数组剪枝避免重复选择，时间复杂度 O(n!)。

##### 数据结构：数组

**遍历框架（仅控制变量）**
```
递归函数(path):
    if 路径长度 == n:
        记录结果
        return
    for i in 0..n-1:
        if not used[i]:
            used[i]=true; path.push(nums[i])
            递归(path)
            path.pop(); used[i]=false
```

**完整模板（含辅助变量）**
```cpp
class Permutations {
    vector<vector<int>> res;                     // 辅助变量：结果存储
    vector<int> path;                            // 辅助变量：路径记录
    vector<bool> used;                           // 辅助变量：状态标记

    void backtrack(vector<int>& nums) {
        if (path.size() == nums.size()) {        // 递归出口
            res.push_back(path);                 // 节点操作：记录叶子
            return;
        }
        for (int i = 0; i < nums.size(); ++i) {  // 遍历控制变量：i
            if (used[i]) continue;               // 节点操作：剪枝
            path.push_back(nums[i]);             // 节点操作：做选择
            used[i] = true;
            backtrack(nums);
            path.pop_back();                     // 节点操作：撤销选择
            used[i] = false;
        }
    }

public:
    vector<vector<int>> permute(vector<int>& nums) {
        used.assign(nums.size(), false);
        backtrack(nums);
        return res;
    }
};
```

---

### 三、子树解

**定义**：解是以某节点为根的子树（包含该节点及其所有后代），通常需要计算子树信息。

#### 暴力枚举（枚举所有子树）

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(node):
    if node == null: return
    计算以 node 为根的子树信息
    递归左子树
    递归右子树
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（前序或后序）
// 遍历控制变量：当前节点
// 辅助变量：结果集
void dfs(node):
    if node == null: return
    // 节点操作：计算当前子树信息
    计算结果
    dfs(node.left)
    dfs(node.right)
```

##### 数据结构：二叉树（子树和）

**遍历框架（仅控制变量）**
```cpp
int subtreeSum(TreeNode* root) {
    if (!root) return 0;
    int left = subtreeSum(root->left);
    int right = subtreeSum(root->right);
    int sum = left + right + root->val;
    // 记录或使用 sum
    return sum;
}
```

**完整模板（含辅助变量）**
```cpp
int subtreeSum(TreeNode* root) {
    if (!root) return 0;                                   // 递归出口
    int left = subtreeSum(root->left);                     // 遍历控制变量：左子树
    int right = subtreeSum(root->right);                   // 遍历控制变量：右子树
    int sum = left + right + root->val;                    // 节点操作：计算和
    // 此处可更新全局结果
    return sum;
}
// 辅助变量：无
```

---

#### 优化算法1：树形DP（一般形式）

**优化说明**：利用**后序遍历**，每个节点返回其子树的相关信息（如最大值、最小值、和等），通过合并子节点结果得到当前节点结果。时间复杂度 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(node):
    if node == null: return 默认值
    left = 递归(node.left)
    right = 递归(node.right)
    节点操作：根据 left, right 和 node 的值计算当前结果
    return 当前结果
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（后序）
// 遍历控制变量：当前节点
// 辅助变量：无（返回值）
Result dfs(node):
    if node == null: return 默认值
    left = dfs(node.left)
    right = dfs(node.right)
    cur = 合并(left, right, node)      // 节点操作：合并子结果
    return cur
```

##### 数据结构：二叉树（示例：子树最大路径和，此处仅展示通用结构）

```cpp
// 树形DP的通用形式
pair<int,int> dfs(TreeNode* node) {
    if (!node) return {0, -inf};
    auto left = dfs(node->left);
    auto right = dfs(node->right);
    // 根据具体问题合并
    return result;
}
```

---

### 四、结构解

**定义**：解是树的特定结构或性质，如同构、平衡性、剖分、动态变化等。

#### 暴力枚举（遍历所有节点）

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(node):
    if node == null: return
    节点操作：检查当前节点是否满足结构性质
    递归左子树
    递归右子树
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（前序）
// 遍历控制变量：当前节点
// 辅助变量：结果集
void dfs(node):
    if node == null: return
    // 节点操作：判断当前节点结构
    判断逻辑
    dfs(node.left)
    dfs(node.right)
```

##### 数据结构：二叉树（示例：检查两棵树是否相同）

**遍历框架（仅控制变量）**
```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p && !q) return true;
    if (!p || !q) return false;
    if (p->val != q->val) return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
}
```

**完整模板（含辅助变量）**
```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p && !q) return true;                           // 递归出口
    if (!p || !q) return false;                          // 节点操作：比较结构
    if (p->val != q->val) return false;                  // 节点操作：比较值
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right); // 遍历控制变量
}
// 辅助变量：无
```

---

#### 优化算法1：二叉搜索树查找

**优化说明**：利用**BST性质**（左子树 < 根 < 右子树），每次比较可排除一半子树，将 O(n) 降为 O(log n)（平衡时）。

##### 通用模板

**遍历框架（仅控制变量）**
```
while node != null and node.val != target:
    if target < node.val: node = node.left
    else: node = node.right
return node
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：当前节点指针
// 辅助变量：无
while node != null and node.val != target:
    if target < node.val: node = node.left
    else: node = node.right
return node
```

##### 数据结构：二叉搜索树

**遍历框架（仅控制变量）**
```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    while (root && root->val != val) {
        if (val < root->val) root = root->left;
        else root = root->right;
    }
    return root;
}
```

**完整模板（含辅助变量）**
```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    TreeNode* cur = root;                               // 遍历控制变量：cur
    while (cur && cur->val != val) {                    // 遍历框架：迭代
        if (val < cur->val) cur = cur->left;            // 节点操作：比较
        else cur = cur->right;
    }
    return cur;
}
// 辅助变量：无
```

---

#### 优化算法2：二叉搜索树插入

**优化说明**：同样利用BST性质，递归找到插入位置后创建新节点。时间复杂度 O(log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(node, val):
    if node == null: return new Node(val)
    if val < node.val: node.left = 递归(node.left, val)
    else: node.right = 递归(node.right, val)
    return node
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归
// 遍历控制变量：当前节点
// 辅助变量：无
TreeNode* insert(node, val):
    if node == null: return new TreeNode(val)
    if val < node.val: node.left = insert(node.left, val)
    else: node.right = insert(node.right, val)
    return node
```

##### 数据结构：二叉搜索树

**遍历框架（仅控制变量）**
```cpp
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left = insertIntoBST(root->left, val);
    else root->right = insertIntoBST(root->right, val);
    return root;
}
```

**完整模板（含辅助变量）**
```cpp
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);                 // 递归出口
    if (val < root->val)                                 // 节点操作：比较
        root->left = insertIntoBST(root->left, val);     // 遍历控制变量：左子树
    else
        root->right = insertIntoBST(root->right, val);   // 遍历控制变量：右子树
    return root;
}
// 辅助变量：无
```

---

#### 优化算法3：并查集（Union-Find）

**优化说明**：利用**树形结构**维护集合，通过**路径压缩**和**按秩合并**，将查找和合并操作近似 O(α(n))。

##### 通用模板

**遍历框架（仅控制变量）**
```
查找：
    if parent[x] != x: parent[x] = find(parent[x])
    return parent[x]
合并：
    x = find(x); y = find(y)
    if x == y: return
    if rank[x] < rank[y]: parent[x] = y
    else if rank[x] > rank[y]: parent[y] = x
    else: parent[y] = x; rank[x]++
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
```

**完整模板（含辅助变量）**
```cpp
class UnionFind {
    vector<int> parent, rank;                     // 辅助变量：状态标记
public:
    UnionFind(int n) {
        parent.resize(n);
        rank.resize(n, 0);
        for (int i = 0; i < n; ++i) parent[i] = i;
    }
    int find(int x) {                             // 遍历控制变量：x
        if (parent[x] != x) parent[x] = find(parent[x]); // 节点操作：路径压缩
        return parent[x];
    }
    void unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return;
        if (rank[x] < rank[y]) parent[x] = y;      // 节点操作：按秩合并
        else if (rank[x] > rank[y]) parent[y] = x;
        else { parent[y] = x; rank[x]++; }
    }
};
```


## 六、总结

树形解空间是算法设计中非常重要的领域，涵盖从基础遍历到复杂动态树结构的丰富内容。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种树形算法。本框架覆盖了二叉树遍历、路径问题、树形DP、BST、平衡树、树链剖分、并查集、回溯等典型算法，并展示了每种算法如何从暴力枚举出发，利用树的结构特性（递归性质、子树独立性、有序性）进行优化，实现从指数级到多项式级的跨越。掌握这套框架，能够帮助你快速设计出清晰、高效的树形算法，并为更复杂的图论问题打下坚实基础。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。