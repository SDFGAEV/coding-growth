# 线性解空间：算法认知框架

> 本文档是“算法认知框架”系列的第一部分，聚焦于**线性解空间**的算法设计与实现。

## 核心理念

### 线性解空间定义
线性解空间由**有序元素**（如数组元素、字符、链表节点）构成，元素之间具有**线性顺序**关系。解是这些元素的某种组合或关系，且顺序在问题中起关键作用。

**基础元素**：数组、字符串、链表等线性结构中的个体。

### 算法本质（在线性解空间中的体现）
**算法 = 在解空间上的聪明穷举 + 辅助变量优化**，对于线性解空间：
- **无遗漏**：必须考虑所有可能的解（如所有子数组、所有数对等）
- **无冗余**：利用线性结构的特性（有序性、连续性）避免重复计算

### 解空间的人为构造
解空间本身是问题的所有可能解的集合，其结构是抽象的。但在设计算法时，我们可以人为地为解空间赋予一种便于遍历的结构。**同一问题的解空间可以有多种不同的构造方式，选择不同的结构会导致不同的算法策略。**

例如：
- **点对解**：原始解空间是二维的（所有可能的 `(i, j)` 组合）。我们可以将其组织成线性结构，通过双指针在线性时间内遍历（利用有序性）。
- **排列解**：原始解空间是指数级的，但我们可以将其组织成**决策树**（树形结构），通过回溯深度优先遍历。
- **子序列解**：原始解空间是组合空间（每个元素选或不选），可以组织成**DAG**（动态规划）或**决策树**（回溯）。

因此，在分析线性解空间时，我们关注的是将解空间构造为线性结构后的遍历和优化。


## 一、线性解空间的解的形态

线性解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构（人为构造） |
|:---|:---|:---|:---|
| **单点解** | 单个位置上的元素 | 顺序查找、二分查找、数组访问 | 一维索引（线性） |
| **点对解** | 两个元素的关系（和、差、顺序等） | 两数之和、三数之和、逆序对 | 二维索引，但可构造为线性（双指针） |
| **连续段解** | 由起点和终点确定的连续区间 | 滑动窗口、前缀和、差分、区间最值、线段树、树状数组、分块、莫队、区间DP | 二维区间，但可构造为线性（滑动窗口） |
| **子序列解** | 保持原序但不连续的元素集合 | 最长递增子序列、最长公共子序列、编辑距离 | 组合空间，但可构造为DAG（DP）或决策树（回溯） |
| **模式匹配解** | 文本串中模式串出现的位置或匹配区间 | KMP、扩展KMP、Manacher、字符串哈希、Z算法 | 一维索引，利用模式串内部信息优化 |


## 二、遍历框架与遍历控制变量

线性解空间的遍历框架主要由**迭代**主导，也可使用**递归**（如分治）。根据构造的线性结构不同，遍历控制变量也有所不同。

| 解的形态 | 数据结构 | 遍历框架（模板） | 遍历控制变量（模板） |
|:---|:---|:---|:---|
| **单点解** | 数组 | `for i in 0..n-1` | `i` |
| **单点解** | 有序数组（二分） | `while left <= right` | `left, right, mid` |
| **点对解** | 数组（无序） | `for i` + 哈希表辅助 | `i` + `hash` |
| **点对解** | 有序数组（双指针） | `while left < right` | `left, right` |
| **点对解** | 数组（分治） | 递归区间 `[l,r]` | `l, r` |
| **连续段解** | 数组（滑动窗口） | `for right` + `while` 收缩 | `left, right` |
| **连续段解** | 数组（前缀和） | `for i` 预处理 | `i` |
| **连续段解** | 数组（分块） | 分块预处理，查询时遍历 | 块索引、块内偏移 |
| **子序列解** | 数组（DP） | 双层循环 | `i, j` |
| **子序列解** | 数组（贪心+二分） | `for x` + 二分 | `x`, `tails` 数组 |
| **模式匹配解** | 字符串（KMP） | `for i` 遍历文本，`j` 根据 `next` 跳转 | `i, j`, `next` 数组 |
| **模式匹配解** | 字符串（哈希） | 预处理哈希，查询 O(1) | `i` |


## 三、节点操作

节点操作是在每个被访问的“节点”上执行的具体逻辑。在线性解空间中，节点通常指**当前元素**或**当前区间**。

| 解的形态 | 数据结构 | 节点操作（模板） | 时机 |
|:---|:---|:---|:---|
| **单点解** | 数组 | 比较 `nums[i] == target` | 前序 |
| **点对解（双指针）** | 有序数组 | 计算 `sum = nums[left]+nums[right]`，移动指针 | 前序 |
| **点对解（哈希）** | 无序数组 | 检查 `hash.count(complement)`，插入 `hash[nums[i]]` | 前序 |
| **连续段解（滑动窗口）** | 数组 | 加入 `nums[right]`，当条件满足时收缩左边界，更新结果 | 前序 |
| **连续段解（前缀和）** | 数组 | `prefix[i] = prefix[i-1] + nums[i-1]` | 前序（构建） |
| **连续段解（分治）** | 数组 | 合并两个有序子数组，统计信息 | 后序 |
| **子序列解（DP）** | 数组 | 若 `nums[j] < nums[i]`，则 `dp[i] = max(dp[i], dp[j]+1)` | 前序 |
| **模式匹配解（KMP）** | 字符串 | 比较字符，失配时 `j = next[j-1]` | 前序 |


## 四、辅助变量

辅助变量用于记录状态、剪枝、缓存结果，保证无冗余。

### 功能分类
| 解的形态 | 算法 | 辅助变量（模板） | 功能分类 |
|:---|:---|:---|:---|
| **单点解** | 顺序查找 | 无 | - |
| **单点解** | 二分查找 | 无 | - |
| **点对解** | 两数之和（哈希） | `unordered_map<int,int>` | 优化缓存 |
| **点对解** | 双指针 | 无 | - |
| **点对解** | 归并逆序对 | `vector<int> temp` | 临时计算 |
| **连续段解** | 滑动窗口 | `unordered_set` / `unordered_map` | 状态标记 |
| **连续段解** | 前缀和 | `vector<int> prefix` | 优化缓存 |
| **连续段解** | 线段树 | `vector<int> tree, lazy` | 优化缓存 |
| **子序列解** | LIS (DP) | `vector<int> dp` | 优化缓存 |
| **子序列解** | LIS (贪心) | `vector<int> tails` | 优化缓存 |
| **模式匹配解** | KMP | `vector<int> next` | 优化缓存 |
| **模式匹配解** | 字符串哈希 | `vector<long long> h, pow` | 优化缓存 |

### 变量生命周期与存放位置
线性算法通常只涉及一次函数调用，辅助变量多为局部变量，但某些情况也需要考虑递归内共享或类内长期。

| 类型 | 描述 | 示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次函数执行中存在 | 滑动窗口中的 `left`, `right`, `window` | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 归并排序中的临时数组 `temp` | 递归参数（引用） |
| **类内长期** | 跨多次公共方法调用持久 | 前缀和类中的 `prefix` 数组 | 类的成员变量 |


## 五、各子类算法详细模板

### 一、单点解

**定义**：解是单个位置上的元素，即需要在给定线性结构中找出特定位置或验证某个值是否存在。

#### 暴力枚举（顺序查找）

##### 通用模板（所有线性数据结构）

**遍历框架（仅控制变量）**
```
初始化遍历控制变量指向第一个元素
while 还有元素:
    节点操作：比较当前元素与目标
    移动遍历控制变量到下一个元素
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：指向当前元素的指针/索引
// 辅助变量：无
while (遍历控制变量未到达末尾) {
    // 节点操作：比较
    if (当前元素 == target) 返回当前位置
    移动遍历控制变量到下一个位置
}
返回 未找到
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
for (int i = 0; i < n; ++i) {   // 遍历控制变量：i
    // 节点操作：比较 nums[i] 与 target
}
```

**完整模板（含辅助变量）**
```cpp
int linearSearch(const vector<int>& nums, int target) {
    for (int i = 0; i < nums.size(); ++i) {   // 遍历控制变量：i
        if (nums[i] == target) return i;       // 节点操作：比较
    }
    return -1;
}
// 辅助变量：无
```

##### 数据结构：链表

**遍历框架（仅控制变量）**
```cpp
ListNode* cur = head;               // 遍历控制变量：cur
while (cur) {                       // 遍历框架：迭代
    // 节点操作：比较 cur->val 与 target
    cur = cur->next;                // 移动遍历控制变量
}
```

**完整模板（含辅助变量）**
```cpp
ListNode* linearSearch(ListNode* head, int target) {
    ListNode* cur = head;                       // 遍历控制变量：cur
    while (cur) {                               // 遍历框架：迭代
        if (cur->val == target) return cur;     // 节点操作：比较
        cur = cur->next;                        // 移动遍历控制变量
    }
    return nullptr;
}
// 辅助变量：无
```

---

#### 优化算法1：二分查找

**优化说明**：利用序列的**有序性**，每次比较后可以排除一半的解空间，将 O(n) 降为 O(log n)。

##### 通用模板（有序线性数据结构）

**遍历框架（仅控制变量）**
```
初始化 left = 第一个位置，right = 最后一个位置
while left <= right:
    mid = (left + right) / 2
    节点操作：比较中间元素与目标
    根据比较结果更新 left 或 right
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（每次缩小一半）
// 遍历控制变量：left, right, mid
// 辅助变量：无
left = 0, right = n-1
while left <= right:
    mid = left + (right-left)/2
    if 中间元素 == target: return mid
    else if 中间元素 < target: left = mid+1
    else: right = mid-1
return -1
```

##### 数据结构：有序数组

**遍历框架（仅控制变量）**
```cpp
int left = 0, right = n - 1;
while (left <= right) {
    int mid = left + (right - left) / 2;
    // 节点操作：比较 nums[mid] 与 target
}
```

**完整模板（含辅助变量）**
```cpp
int binarySearch(const vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;          // 遍历控制变量：left, right
    while (left <= right) {                         // 遍历框架：迭代
        int mid = left + (right - left) / 2;         // 节点操作：计算中点
        if (nums[mid] == target) return mid;        // 节点操作：比较
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
// 辅助变量：无
```

##### 数据结构：有序链表（不适用直接二分，需转换为数组或使用跳表，此处略）

---

#### 优化算法2：二分查找答案（值域二分）

**优化说明**：将问题转化为可行性判断，利用**单调性**在值域上进行二分搜索，避免直接遍历所有可能值，将 O(值域) 降为 O(log 值域)。

##### 通用模板（可行域为连续整数的问题）

**遍历框架（仅控制变量）**
```
left = L, right = R
while left < right:
    mid = (left + right) / 2
    节点操作：调用 check(mid) 判断可行性
    根据结果更新 left 或 right
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（值域二分）
// 遍历控制变量：left, right, mid
// 辅助变量：check 函数（根据问题定义）
left = L, right = R
while left < right:
    mid = left + (right-left)/2
    if check(mid):          // 节点操作：可行性判断
        right = mid
    else:
        left = mid + 1
return left
```

##### 数据结构：数值区间（无具体容器）

**遍历框架（仅控制变量）**
```cpp
int left = L, right = R;
while (left < right) {
    int mid = left + (right - left) / 2;
    // 节点操作：调用 check(mid)
}
```

**完整模板（含辅助变量）**
```cpp
// 判断函数，需根据具体问题实现
bool check(int x) {
    // 根据问题定义
    return true / false;
}

int binarySearchAnswer(int L, int R) {
    int left = L, right = R;                         // 遍历控制变量：left, right
    while (left < right) {                           // 遍历框架：迭代
        int mid = left + (right - left) / 2;          // 节点操作：计算中点
        if (check(mid)) right = mid;                 // 节点操作：可行性判断
        else left = mid + 1;
    }
    return left;
}
// 辅助变量：check 函数
```


### 二、点对解

**定义**：解涉及两个元素的位置关系，例如两数之和、三数之和、逆序对等。

#### 暴力枚举

##### 通用模板（所有线性数据结构）

**遍历框架（仅控制变量）**
```
for i = 0 to n-2:
    for j = i+1 to n-1:
        节点操作：处理数对 (i, j)
```

**完整模板（含辅助变量）**
```
// 遍历框架：两层嵌套循环
// 遍历控制变量：i, j
// 辅助变量：无
for i = 0 to n-2:
    for j = i+1 to n-1:
        // 节点操作：检查条件
        if 满足条件:
            记录或处理
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
for (int i = 0; i < n-1; ++i) {
    for (int j = i+1; j < n; ++j) {
        // 节点操作：处理 nums[i] 和 nums[j]
    }
}
```

**完整模板（含辅助变量）**
```cpp
// 示例：两数之和
vector<pair<int,int>> bruteTwoSum(const vector<int>& nums, int target) {
    vector<pair<int,int>> res;                      // 辅助变量：结果存储
    for (int i = 0; i < nums.size(); ++i) {         // 遍历控制变量：i
        for (int j = i+1; j < nums.size(); ++j) {   // 遍历控制变量：j
            if (nums[i] + nums[j] == target) {      // 节点操作：检查
                res.emplace_back(i, j);
            }
        }
    }
    return res;
}
```

---

#### 优化算法1：哈希表优化（无序）

**优化说明**：利用**哈希表**存储已遍历元素，将查找补数从 O(n) 降为 O(1)，整体 O(n)。这是典型的**空间换时间**。

##### 通用模板（无序数据）

**遍历框架（仅控制变量）**
```
初始化哈希表
for i = 0 to n-1:
    complement = target - nums[i]
    节点操作：检查 complement 是否在哈希表中
    节点操作：将 nums[i] 插入哈希表
```

**完整模板（含辅助变量）**
```
// 遍历框架：单层循环
// 遍历控制变量：i
// 辅助变量：hash（优化缓存）
hash = 空哈希表
for i = 0 to n-1:
    complement = target - nums[i]
    if hash.contains(complement):   // 节点操作：查找
        返回结果
    hash.insert(nums[i], i)          // 节点操作：插入
```

##### 数据结构：无序数组

**遍历框架（仅控制变量）**
```cpp
unordered_map<int,int> hash;
for (int i = 0; i < n; ++i) {
    int complement = target - nums[i];
    // 节点操作：检查 hash 中是否有 complement
    // 节点操作：插入 nums[i]
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> twoSum(const vector<int>& nums, int target) {
    unordered_map<int,int> hash;                     // 辅助变量：优化缓存
    for (int i = 0; i < nums.size(); ++i) {          // 遍历控制变量：i
        int complement = target - nums[i];
        if (hash.count(complement)) {                // 节点操作：查找
            return {hash[complement], i};
        }
        hash[nums[i]] = i;                           // 节点操作：插入
    }
    return {};
}
```

---

#### 优化算法2：双指针优化（有序）

**优化说明**：利用序列的**有序性**，采用左右指针，根据当前和与目标的大小关系移动指针，每次可以排除一行或一列的数对，将 O(n²) 降为 O(n)。

##### 通用模板（有序数据）

**遍历框架（仅控制变量）**
```
left = 0, right = n-1
while left < right:
    节点操作：计算当前和
    根据比较结果移动 left 或 right
```

**完整模板（含辅助变量）**
```
// 遍历框架：双指针迭代
// 遍历控制变量：left, right
// 辅助变量：无
left = 0, right = n-1
while left < right:
    sum = nums[left] + nums[right]          // 节点操作：计算
    if sum == target: return {left, right}
    else if sum < target: left++
    else: right--
```

##### 数据结构：有序数组

**遍历框架（仅控制变量）**
```cpp
int left = 0, right = n-1;
while (left < right) {
    int sum = nums[left] + nums[right];
    // 节点操作：比较
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> twoSumII(const vector<int>& numbers, int target) {
    int left = 0, right = numbers.size() - 1;      // 遍历控制变量：left, right
    while (left < right) {                         // 遍历框架：迭代
        int sum = numbers[left] + numbers[right];  // 节点操作：计算
        if (sum == target) return {left + 1, right + 1};
        else if (sum < target) left++;
        else right--;
    }
    return {};
}
// 辅助变量：无
```

---

#### 优化算法3：分治优化（归并思想）

**优化说明**：利用**分治**思想，在归并排序的过程中统计逆序对，将 O(n²) 降为 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(l, r):
    if l >= r: return 0
    mid = (l+r)/2
    count = 递归(l, mid) + 递归(mid+1, r)
    合并并统计跨区间的逆序对
    return count
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归（分治）
// 遍历控制变量：区间 [l, r]
// 辅助变量：temp（临时数组，递归内共享）
int mergeCount(l, r, temp):
    if l >= r: return 0
    mid = (l+r)/2
    count = mergeCount(l, mid, temp) + mergeCount(mid+1, r, temp)
    // 合并两个有序子数组并统计逆序对
    i = l, j = mid+1, k = l
    while i <= mid and j <= r:
        if nums[i] <= nums[j]:
            temp[k++] = nums[i++]
        else:
            count += mid - i + 1      // 节点操作：统计
            temp[k++] = nums[j++]
    // 复制剩余元素
    // 将 temp 复制回 nums
    return count
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
int mergeCount(vector<int>& nums, int l, int r, vector<int>& temp) {
    if (l >= r) return 0;
    int mid = l + (r - l) / 2;
    // ...
}
```

**完整模板（含辅助变量）**
```cpp
int mergeCount(vector<int>& nums, int l, int r, vector<int>& temp) {
    if (l >= r) return 0;                                         // 递归出口
    int mid = l + (r - l) / 2;
    int count = mergeCount(nums, l, mid, temp) + mergeCount(nums, mid+1, r, temp);
    int i = l, j = mid+1, k = l;
    while (i <= mid && j <= r) {                                  // 遍历框架：合并
        if (nums[i] <= nums[j]) {
            temp[k++] = nums[i++];
        } else {
            count += mid - i + 1;                                 // 节点操作：统计逆序对
            temp[k++] = nums[j++];
        }
    }
    while (i <= mid) temp[k++] = nums[i++];
    while (j <= r) temp[k++] = nums[j++];
    for (int i = l; i <= r; ++i) nums[i] = temp[i];
    return count;
}
int reversePairs(vector<int>& nums) {
    vector<int> temp(nums.size());                               // 辅助变量：临时数组
    return mergeCount(nums, 0, nums.size()-1, temp);
}
```


### 三、连续段解

**定义**：解是连续的一段区间，由起始和结束位置确定。问题通常涉及子数组的和、长度、最值等。

#### 暴力枚举

##### 通用模板

**遍历框架（仅控制变量）**
```
for l = 0 to n-1:
    for r = l to n-1:
        节点操作：计算或检查区间 [l, r]
```

**完整模板（含辅助变量）**
```
// 遍历框架：两层循环（枚举起点和终点）
// 遍历控制变量：l, r
// 辅助变量：当前区间和（可选，可累加）
for l = 0 to n-1:
    cur = 0
    for r = l to n-1:
        cur += nums[r]                // 节点操作：累加
        更新结果
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
for (int l = 0; l < n; ++l) {
    for (int r = l; r < n; ++r) {
        // 节点操作：处理区间 [l, r]
    }
}
```

**完整模板（含辅助变量）**
```cpp
// 示例：最大子数组和（暴力）
int maxSubArrayBrute(const vector<int>& nums) {
    int ans = INT_MIN;
    for (int l = 0; l < nums.size(); ++l) {          // 遍历控制变量：l
        int sum = 0;
        for (int r = l; r < nums.size(); ++r) {      // 遍历控制变量：r
            sum += nums[r];                           // 节点操作：累加
            ans = max(ans, sum);                     // 节点操作：更新结果
        }
    }
    return ans;
}
// 辅助变量：sum（临时计算），ans（结果存储）
```

---

#### 优化算法1：滑动窗口

**优化说明**：利用**连续性**和**单调性**，通过左右指针维护窗口，每个元素最多进出窗口一次，将 O(n²) 降为 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
left = 0
for right = 0 to n-1:
    节点操作：将 nums[right] 加入窗口
    while 窗口不满足条件:
        节点操作：将 nums[left] 移出窗口
        left++
    节点操作：更新结果
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（右指针扩展，左指针收缩）
// 遍历控制变量：left, right
// 辅助变量：窗口状态（如哈希表、计数）
left = 0
for right = 0 to n-1:
    加入 nums[right] 到窗口                    // 节点操作
    while 窗口不满足条件:
        移除 nums[left] 从窗口                // 节点操作
        left++
    更新结果                                 // 节点操作
```

##### 数据结构：数组/字符串

**遍历框架（仅控制变量）**
```cpp
int left = 0;
for (int right = 0; right < n; ++right) {
    // 加入 nums[right]
    while (条件) {
        // 移除 nums[left]
        left++;
    }
    // 更新结果
}
```

**完整模板（含辅助变量）**
```cpp
// 示例：最长无重复字符子串
int lengthOfLongestSubstring(string s) {
    unordered_set<char> window;                 // 辅助变量：窗口状态（状态标记）
    int left = 0, maxLen = 0;                   // 遍历控制变量：left, right
    for (int right = 0; right < s.size(); ++right) {   // 遍历框架：迭代
        while (window.count(s[right])) {        // 节点操作：收缩窗口
            window.erase(s[left]);
            left++;
        }
        window.insert(s[right]);                // 节点操作：扩大窗口
        maxLen = max(maxLen, right - left + 1); // 节点操作：更新结果
    }
    return maxLen;
}
```

---

#### 优化算法2：前缀和

**优化说明**：通过**预处理**前缀和，将区间和查询从 O(n) 降为 O(1)，适用于多次查询场景。

##### 通用模板

**遍历框架（仅控制变量）**
```
预处理：
    prefix[0] = 0
    for i = 1 to n:
        prefix[i] = prefix[i-1] + nums[i-1]
查询：
    返回 prefix[r+1] - prefix[l]
```

**完整模板（含辅助变量）**
```
// 预处理阶段
// 遍历控制变量：i
// 辅助变量：prefix（优化缓存）
prefix[0] = 0
for i = 1 to n:
    prefix[i] = prefix[i-1] + nums[i-1]     // 节点操作：累加

// 查询阶段
// 节点操作：公式计算
return prefix[r+1] - prefix[l]
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> prefix(n+1, 0);
for (int i = 1; i <= n; ++i) {
    prefix[i] = prefix[i-1] + nums[i-1];
}
```

**完整模板（含辅助变量）**
```cpp
class PrefixSum {
private:
    vector<int> prefix;                         // 辅助变量：优化缓存
public:
    PrefixSum(vector<int>& nums) {
        int n = nums.size();
        prefix.resize(n + 1, 0);
        for (int i = 1; i <= n; ++i) {          // 遍历控制变量：i
            prefix[i] = prefix[i-1] + nums[i-1]; // 节点操作：累加
        }
    }
    int query(int l, int r) {
        return prefix[r+1] - prefix[l];          // 节点操作：公式计算
    }
};
```

---

#### 优化算法3：差分

**优化说明**：将区间更新从 O(n) 降为 O(1)，最后 O(n) 还原，适用于多次区间增量场景。

##### 通用模板

**遍历框架（仅控制变量）**
```
更新：diff[l] += c; diff[r+1] -= c;
还原：
    res[0] = diff[0]
    for i = 1 to n-1:
        res[i] = res[i-1] + diff[i]
```

**完整模板（含辅助变量）**
```
// 更新操作：O(1)
diff[l] += c
diff[r+1] -= c

// 还原阶段
// 遍历控制变量：i
// 辅助变量：diff（优化缓存）
res[0] = diff[0]
for i = 1 to n-1:
    res[i] = res[i-1] + diff[i]          // 节点操作：累加
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> diff(n+1, 0);
// 多次更新
diff[l] += c; diff[r+1] -= c;
// 还原
vector<int> res(n);
res[0] = diff[0];
for (int i = 1; i < n; ++i) {
    res[i] = res[i-1] + diff[i];
}
```

**完整模板（含辅助变量）**
```cpp
class Difference {
private:
    vector<int> diff;                           // 辅助变量：优化缓存
public:
    Difference(int n) : diff(n + 1, 0) {}
    void increment(int l, int r, int c) {
        diff[l] += c;                           // 节点操作：更新
        diff[r + 1] -= c;
    }
    vector<int> result() {
        int n = diff.size() - 1;
        vector<int> res(n);
        res[0] = diff[0];
        for (int i = 1; i < n; ++i) {           // 遍历控制变量：i
            res[i] = res[i-1] + diff[i];        // 节点操作：累加
        }
        return res;
    }
};
```

---

#### 优化算法4：区间最值（ST表）

**优化说明**：利用**倍增思想**预处理，将静态区间最值查询从 O(n) 降为 O(1)，适用于多次 RMQ 查询。

##### 通用模板

**遍历框架（仅控制变量）**
```
预处理：
    for i = 0 to n-1: st[i][0] = nums[i]
    for j = 1 to K-1:
        for i = 0 to n-(1<<j):
            st[i][j] = min(st[i][j-1], st[i+(1<<(j-1))][j-1])
查询：
    k = log2(r-l+1)
    return min(st[l][k], st[r-(1<<k)+1][k])
```

**完整模板（含辅助变量）**
```
// 预处理阶段
// 遍历控制变量：i, j
// 辅助变量：st（优化缓存），log（预处理）
for i = 0 to n-1:
    st[i][0] = nums[i]
for j = 1 to K-1:
    for i = 0 to n-(1<<j):
        st[i][j] = min(st[i][j-1], st[i+(1<<(j-1))][j-1])   // 节点操作：合并

// 查询阶段
k = log[r-l+1]
return min(st[l][k], st[r-(1<<k)+1][k])                       // 节点操作：组合
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
int K = log2(n) + 1;
vector<vector<int>> st(n, vector<int>(K));
for (int i = 0; i < n; ++i) st[i][0] = nums[i];
for (int j = 1; j < K; ++j) {
    for (int i = 0; i + (1<<j) <= n; ++i) {
        st[i][j] = min(st[i][j-1], st[i + (1<<(j-1))][j-1]);
    }
}
```

**完整模板（含辅助变量）**
```cpp
class SparseTable {
private:
    vector<vector<int>> st;                     // 辅助变量：优化缓存
    vector<int> log;
public:
    SparseTable(vector<int>& nums) {
        int n = nums.size();
        log.resize(n + 1);
        for (int i = 2; i <= n; ++i) log[i] = log[i/2] + 1;  // 预处理 log
        int K = log[n] + 1;
        st.assign(n, vector<int>(K));
        for (int i = 0; i < n; ++i) st[i][0] = nums[i];
        for (int j = 1; j < K; ++j) {                        // 遍历控制变量：j
            for (int i = 0; i + (1 << j) <= n; ++i) {        // 遍历控制变量：i
                st[i][j] = min(st[i][j-1], st[i + (1 << (j-1))][j-1]); // 节点操作：合并
            }
        }
    }
    int query(int l, int r) {
        int k = log[r - l + 1];                               // 节点操作：计算区间长度对应幂
        return min(st[l][k], st[r - (1 << k) + 1][k]);       // 节点操作：组合结果
    }
};
```

---

#### 优化算法5：线段树

**优化说明**：利用**树形结构**维护区间信息，将区间操作（查询、更新）降为 O(log n)。

##### 通用模板（区间求和）

**遍历框架（仅控制变量）**
```
构建：递归构建节点
查询：递归区间合并
更新：递归更新节点，懒标记下推
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归
// 遍历控制变量：节点索引，区间左右端点
// 辅助变量：tree（线段树数组），lazy（懒标记数组）
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
class SegmentTree {
private:
    vector<int> tree, lazy;
    int n;
    void build(const vector<int>& nums, int idx, int l, int r) {
        if (l == r) { tree[idx] = nums[l]; return; }
        int mid = (l + r) / 2;
        build(nums, idx*2, l, mid);
        build(nums, idx*2+1, mid+1, r);
        tree[idx] = tree[idx*2] + tree[idx*2+1];
    }
    void push(int idx, int l, int r) {
        if (lazy[idx] != 0) {
            int mid = (l + r) / 2;
            tree[idx*2] += lazy[idx] * (mid - l + 1);
            tree[idx*2+1] += lazy[idx] * (r - mid);
            lazy[idx*2] += lazy[idx];
            lazy[idx*2+1] += lazy[idx];
            lazy[idx] = 0;
        }
    }
    int query(int idx, int l, int r, int ql, int qr) {
        if (ql <= l && r <= qr) return tree[idx];
        push(idx, l, r);
        int mid = (l + r) / 2, res = 0;
        if (ql <= mid) res += query(idx*2, l, mid, ql, qr);
        if (qr > mid) res += query(idx*2+1, mid+1, r, ql, qr);
        return res;
    }
    void update(int idx, int l, int r, int ql, int qr, int val) {
        if (ql <= l && r <= qr) {
            tree[idx] += val * (r - l + 1);
            lazy[idx] += val;
            return;
        }
        push(idx, l, r);
        int mid = (l + r) / 2;
        if (ql <= mid) update(idx*2, l, mid, ql, qr, val);
        if (qr > mid) update(idx*2+1, mid+1, r, ql, qr, val);
        tree[idx] = tree[idx*2] + tree[idx*2+1];
    }
public:
    SegmentTree(const vector<int>& nums) {
        n = nums.size();
        tree.resize(4 * n);
        lazy.resize(4 * n, 0);
        build(nums, 1, 0, n-1);
    }
    int rangeSum(int l, int r) { return query(1, 0, n-1, l, r); }
    void rangeAdd(int l, int r, int val) { update(1, 0, n-1, l, r, val); }
};
```

---

#### 优化算法6：树状数组

**优化说明**：利用**二进制分解**，将单点更新和前缀和查询降为 O(log n)，代码简洁。

##### 通用模板

**遍历框架（仅控制变量）**
```
更新：for i = idx to n, i += lowbit(i): bit[i] += delta
查询：for i = idx to 0, i -= lowbit(i): sum += bit[i]
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（lowbit 跳跃）
// 遍历控制变量：索引 i
// 辅助变量：bit（树状数组）
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
void add(int idx, int delta) {
    for (int i = idx; i <= n; i += i & -i) bit[i] += delta;
}
int sum(int idx) {
    int res = 0;
    for (int i = idx; i > 0; i -= i & -i) res += bit[i];
    return res;
}
```

**完整模板（含辅助变量）**
```cpp
class Fenwick {
private:
    vector<int> bit;
    int n;
public:
    Fenwick(int n) : n(n), bit(n+1, 0) {}
    void add(int idx, int delta) {                         // 遍历控制变量：i
        for (int i = idx; i <= n; i += i & -i) bit[i] += delta; // 节点操作：更新
    }
    int sum(int idx) {                                     // 遍历控制变量：i
        int res = 0;
        for (int i = idx; i > 0; i -= i & -i) res += bit[i]; // 节点操作：累加
        return res;
    }
    int rangeSum(int l, int r) { return sum(r) - sum(l-1); }
};
```

---

#### 优化算法7：分块

**优化说明**：将序列分为 √n 块，整块用预处理信息 O(1) 获取，零散部分暴力 O(√n)，整体 O(√n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
预处理：分块，维护每块信息
查询：计算块索引，整块累加，零散暴力
```

**完整模板（含辅助变量）**
```
// 遍历控制变量：块索引，块内偏移
// 辅助变量：分块数组，块大小
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
class Block {
private:
    vector<int> arr;
    vector<int> block_sum;
    int block_size;
public:
    Block(const vector<int>& nums) {
        arr = nums;
        int n = nums.size();
        block_size = sqrt(n);
        block_sum.resize((n + block_size - 1) / block_size, 0);
        for (int i = 0; i < n; ++i) block_sum[i / block_size] += nums[i];
    }
    int query(int l, int r) {
        int bl = l / block_size, br = r / block_size;
        if (bl == br) {
            int sum = 0;
            for (int i = l; i <= r; ++i) sum += arr[i];
            return sum;
        }
        int sum = 0;
        for (int i = l; i < (bl+1)*block_size; ++i) sum += arr[i];
        for (int i = bl+1; i < br; ++i) sum += block_sum[i];
        for (int i = br*block_size; i <= r; ++i) sum += arr[i];
        return sum;
    }
    void update(int pos, int val) {
        int bl = pos / block_size;
        block_sum[bl] += val - arr[pos];
        arr[pos] = val;
    }
};
```

---

#### 优化算法8：莫队算法

**优化说明**：离线处理区间查询，通过排序和双指针移动，将指针移动总次数控制在 O((n+q)√n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
排序查询，初始化左右指针，依次处理每个查询，移动指针并更新答案
```

**完整模板（含辅助变量）**
```
// 遍历控制变量：当前左右端点 curL, curR
// 辅助变量：查询数组，频率数组
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
struct Query { int l, r, idx; };
vector<int> mo(vector<Query>& queries, const vector<int>& arr) {
    int n = arr.size(), q = queries.size();
    int block = sqrt(n);
    sort(queries.begin(), queries.end(), [block](const Query& a, const Query& b) {
        if (a.l / block != b.l / block) return a.l < b.l;
        return (a.l / block) % 2 ? a.r < b.r : a.r > b.r;
    });
    vector<int> ans(q);
    vector<int> freq(1000001, 0);
    int curL = 0, curR = -1, curAns = 0;
    auto add = [&](int pos) { /* 添加 arr[pos] */ };
    auto remove = [&](int pos) { /* 移除 arr[pos] */ };
    for (const auto& q : queries) {
        while (curL > q.l) add(--curL);
        while (curR < q.r) add(++curR);
        while (curL < q.l) remove(curL++);
        while (curR > q.r) remove(curR--);
        ans[q.idx] = curAns;
    }
    return ans;
}
```

---

#### 优化算法9：区间DP

**优化说明**：利用**最优子结构**，将大区间的最优解分解为子区间的最优解，按区间长度递增顺序计算，时间复杂度 O(n³)。

##### 通用模板

**遍历框架（仅控制变量）**
```
for len = 2 to n:
    for i = 0 to n-len:
        j = i+len-1
        for k = i to j-1:
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + cost)
```

**完整模板（含辅助变量）**
```
// 遍历控制变量：len, i, j, k
// 辅助变量：dp 二维数组
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
int matrixChain(vector<int>& dims) {
    int n = dims.size() - 1;
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int len = 2; len <= n; ++len) {            // 遍历控制变量：len
        for (int i = 0; i <= n - len; ++i) {        // 遍历控制变量：i
            int j = i + len - 1;
            dp[i][j] = INT_MAX;
            for (int k = i; k < j; ++k) {            // 遍历控制变量：k
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + dims[i]*dims[k+1]*dims[j+1]); // 节点操作：合并
            }
        }
    }
    return dp[0][n-1];
}
```


### 四、子序列解

**定义**：解是从原序列中选出的一组元素，保持原序但不一定连续。典型问题：最长递增子序列、最长公共子序列等。

#### 暴力枚举（回溯）

##### 通用模板

**遍历框架（仅控制变量）**
```
递归(pos, path):
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
        // 记录
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
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> res;                 // 辅助变量：结果存储
    vector<int> path;                        // 辅助变量：路径记录
    function<void(int)> dfs = [&](int pos) { // 遍历控制变量：pos
        if (pos == nums.size()) {
            res.push_back(path);             // 节点操作：记录
            return;
        }
        dfs(pos + 1);                        // 节点操作：不选
        path.push_back(nums[pos]);           // 节点操作：做选择
        dfs(pos + 1);                        // 节点操作：递归
        path.pop_back();                     // 节点操作：撤销选择
    };
    dfs(0);
    return res;
}
```

---

#### 优化算法1：最长递增子序列（LIS）动态规划

**优化说明**：利用**最优子结构**和**重叠子问题**，定义状态 `dp[i]` 表示以 `nums[i]` 结尾的 LIS 长度，状态转移只需考虑前面所有比它小的元素，将指数级枚举降为 O(n²)。

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n-1:
    dp[i] = 1
    for j = 0 to i-1:
        if nums[j] < nums[i]:
            dp[i] = max(dp[i], dp[j]+1)
    ans = max(ans, dp[i])
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：dp（优化缓存），ans（结果存储）
for i = 0 to n-1:
    dp[i] = 1
    for j = 0 to i-1:
        if nums[j] < nums[i]:           // 节点操作：比较
            dp[i] = max(dp[i], dp[j]+1) // 节点操作：状态转移
    ans = max(ans, dp[i])               // 节点操作：更新结果
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> dp(n, 1);
int ans = 1;
for (int i = 0; i < n; ++i) {
    for (int j = 0; j < i; ++j) {
        if (nums[j] < nums[i]) {
            // dp[i] = max(dp[i], dp[j]+1)
        }
    }
    ans = max(ans, dp[i]);
}
```

**完整模板（含辅助变量）**
```cpp
int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, 1);                // 辅助变量：优化缓存
    int ans = 1;                         // 辅助变量：结果存储
    for (int i = 0; i < n; ++i) {        // 遍历控制变量：i
        for (int j = 0; j < i; ++j) {    // 遍历控制变量：j
            if (nums[j] < nums[i]) {     // 节点操作：比较
                dp[i] = max(dp[i], dp[j] + 1); // 节点操作：状态转移
            }
        }
        ans = max(ans, dp[i]);           // 节点操作：更新结果
    }
    return ans;
}
```

---

#### 优化算法2：最长递增子序列（LIS）贪心+二分

**进一步优化说明**：维护 `tails` 数组，其中 `tails[i]` 表示长度为 i+1 的递增子序列的最小末尾值，利用**单调性**进行二分查找，将 O(n²) 优化到 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
tails = []
for x in nums:
    在 tails 中二分查找第一个 >= x 的位置 pos
    if pos == tails.size(): tails.push_back(x)
    else: tails[pos] = x
return tails.size()
```

**完整模板（含辅助变量）**
```
// 遍历框架：单层循环 + 二分查找
// 遍历控制变量：x
// 辅助变量：tails（优化缓存）
tails = []
for x in nums:
    pos = lower_bound(tails, x)          // 节点操作：二分查找
    if pos == tails.size():
        tails.push_back(x)                // 节点操作：追加
    else:
        tails[pos] = x                    // 节点操作：替换
return tails.size()
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> tails;
for (int x : nums) {
    auto it = lower_bound(tails.begin(), tails.end(), x);
    // 根据 it 决定插入或替换
}
```

**完整模板（含辅助变量）**
```cpp
int lengthOfLIS_binary(vector<int>& nums) {
    vector<int> tails;                     // 辅助变量：优化缓存
    for (int x : nums) {                   // 遍历控制变量：x
        auto it = lower_bound(tails.begin(), tails.end(), x); // 节点操作：二分查找
        if (it == tails.end()) {
            tails.push_back(x);            // 节点操作：追加
        } else {
            *it = x;                       // 节点操作：替换
        }
    }
    return tails.size();
}
```

---

### 五、模式匹配解

**定义**：在文本串中寻找模式串出现的位置，解是起始索引（单点解）或匹配区间（连续段解），但利用模式串自身信息优化。

#### 暴力枚举

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n-m:
    j = 0
    while j < m and txt[i+j] == pat[j]:
        j++
    if j == m: 找到匹配
```

**完整模板（含辅助变量）**
```
// 遍历框架：外层循环枚举起始位置，内层逐字符匹配
// 遍历控制变量：i, j
// 辅助变量：无
for i = 0 to n-m:
    j = 0
    while j < m and txt[i+j] == pat[j]:   // 节点操作：比较
        j++
    if j == m:                            // 节点操作：检查
        记录匹配位置
```

##### 数据结构：字符串

**遍历框架（仅控制变量）**
```cpp
for (int i = 0; i <= n - m; ++i) {
    int j = 0;
    while (j < m && txt[i+j] == pat[j]) ++j;
    if (j == m) return i;
}
```

**完整模板（含辅助变量）**
```cpp
int bruteMatch(const string& txt, const string& pat) {
    int n = txt.size(), m = pat.size();
    for (int i = 0; i <= n - m; ++i) {        // 遍历控制变量：i
        int j = 0;
        while (j < m && txt[i+j] == pat[j]) { // 节点操作：比较
            ++j;
        }
        if (j == m) return i;                 // 节点操作：检查
    }
    return -1;
}
// 辅助变量：无
```

---

#### 优化算法1：KMP 算法

**优化说明**：利用模式串自身的**前缀-后缀信息**（next 数组），在失配时直接将模式串指针跳转到已匹配部分的下一个可能位置，避免文本指针回溯，将 O(n·m) 降为 O(n+m)。

##### 通用模板

**遍历框架（仅控制变量）**
```
预处理 next 数组：
    j = 0
    for i = 1 to m-1:
        while j > 0 and pat[i] != pat[j]: j = next[j-1]
        if pat[i] == pat[j]: j++
        next[i] = j
匹配：
    j = 0
    for i = 0 to n-1:
        while j > 0 and txt[i] != pat[j]: j = next[j-1]
        if txt[i] == pat[j]: j++
        if j == m: 找到匹配; j = next[j-1]
```

**完整模板（含辅助变量）**
```
// 预处理阶段
// 遍历控制变量：i, j
// 辅助变量：next（优化缓存）
j = 0
for i = 1 to m-1:
    while j > 0 and pat[i] != pat[j]: j = next[j-1]   // 节点操作：回退
    if pat[i] == pat[j]: j++
    next[i] = j

// 匹配阶段
// 遍历控制变量：i, j
j = 0
for i = 0 to n-1:
    while j > 0 and txt[i] != pat[j]: j = next[j-1]   // 节点操作：回退
    if txt[i] == pat[j]: j++                          // 节点操作：比较
    if j == m:                                         // 节点操作：检查
        记录匹配位置
        j = next[j-1]                                 // 继续查找下一个
```

##### 数据结构：字符串

**遍历框架（仅控制变量）**
```cpp
// 求 next 数组
vector<int> next(m);
int j = 0;
for (int i = 1; i < m; ++i) {
    while (j > 0 && pat[i] != pat[j]) j = next[j-1];
    if (pat[i] == pat[j]) ++j;
    next[i] = j;
}
// 匹配
j = 0;
for (int i = 0; i < n; ++i) {
    while (j > 0 && txt[i] != pat[j]) j = next[j-1];
    if (txt[i] == pat[j]) ++j;
    if (j == m) {
        // 匹配成功
        j = next[j-1];
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> getNext(const string& pat) {
    int m = pat.size();
    vector<int> next(m, 0);                          // 辅助变量：优化缓存
    int j = 0;
    for (int i = 1; i < m; ++i) {                    // 遍历控制变量：i
        while (j > 0 && pat[i] != pat[j]) j = next[j-1]; // 节点操作：回退
        if (pat[i] == pat[j]) ++j;
        next[i] = j;                                 // 节点操作：记录
    }
    return next;
}
vector<int> KMP(const string& txt, const string& pat) {
    vector<int> res;                                 // 辅助变量：结果存储
    if (pat.empty()) return res;
    int n = txt.size(), m = pat.size();
    vector<int> next = getNext(pat);
    int j = 0;
    for (int i = 0; i < n; ++i) {                    // 遍历控制变量：i
        while (j > 0 && txt[i] != pat[j]) j = next[j-1]; // 节点操作：回退
        if (txt[i] == pat[j]) ++j;                   // 节点操作：比较
        if (j == m) {                                // 节点操作：检查
            res.push_back(i - m + 1);
            j = next[j-1];
        }
    }
    return res;
}
```

---

#### 优化算法2：字符串哈希

**优化说明**：利用**滚动哈希**预处理，可在 O(1) 时间内得到任意子串的哈希值，从而快速比较子串是否相等，常用于字符串匹配和去重。

##### 通用模板

**遍历框架（仅控制变量）**
```
预处理：
    h[0] = 0; pow[0] = 1
    for i = 1 to n:
        h[i] = h[i-1]*base + s[i-1]
        pow[i] = pow[i-1]*base
查询：
    return h[r+1] - h[l] * pow[r-l+1]
```

**完整模板（含辅助变量）**
```
// 遍历控制变量：i
// 辅助变量：h（哈希数组），pow（幂次数组）
```

##### 数据结构：字符串

**遍历框架（仅控制变量）**
```cpp
vector<unsigned long long> h(n+1), pow(n+1);
pow[0] = 1;
for (int i = 1; i <= n; ++i) {
    h[i] = h[i-1] * base + s[i-1];
    pow[i] = pow[i-1] * base;
}
```

**完整模板（含辅助变量）**
```cpp
class StringHash {
private:
    vector<unsigned long long> h, pow;
    static const unsigned long long base = 131;
public:
    StringHash(const string& s) {
        int n = s.size();
        h.resize(n+1);
        pow.resize(n+1);
        pow[0] = 1;
        for (int i = 1; i <= n; ++i) {          // 遍历控制变量：i
            h[i] = h[i-1] * base + s[i-1];      // 节点操作：计算哈希
            pow[i] = pow[i-1] * base;
        }
    }
    unsigned long long getHash(int l, int r) {
        return h[r+1] - h[l] * pow[r-l+1];       // 节点操作：公式计算
    }
};
```


## 六、总结

线性解空间是算法设计中最基础、最常用的解空间。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种线性算法。本框架涵盖了从基础遍历到复杂区间查询、模式匹配、子序列问题的典型算法，并展示了每种算法如何从暴力枚举出发，利用线性结构的特性（有序性、单调性、重叠子问题、哈希等）进行优化，实现从指数级或多项式级到高效算法的跨越。掌握这套框架，能够帮助你快速设计出清晰、高效的线性算法。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。