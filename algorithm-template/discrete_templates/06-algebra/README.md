# 数学解空间：算法认知框架

> 本文档是“算法认知框架”系列的第五部分，聚焦于**数学解空间**的算法设计与实现。

## 核心理念

### 数学解空间定义
数学解空间由**数值或数学对象**构成，包括整数、实数、模数、多项式、矩阵等。解可以是单个数值、两个数的关系、数列、方程的解、组合计数、数位统计、数论函数值、多项式运算结果或线性代数问题的解。

**基础元素**：整数、实数、模数、组合数、矩阵、多项式等。

### 算法本质（在数学解空间中的体现）
**算法 = 在解空间上的聪明穷举 + 辅助变量优化**，对于数学解空间：
- **无遗漏**：必须考虑所有可能的数值或数学对象（如所有素数、所有因数、所有数位等）
- **无冗余**：利用数学定理（如算术基本定理、同余性质、递推关系、数论函数性质）避免重复计算

### 解空间的人为构造
解空间本身是问题的所有可能解的集合，其结构是抽象的。但在设计算法时，我们可以人为地为解空间赋予一种便于遍历的结构。**同一问题的解空间可以有多种不同的构造方式，选择不同的结构会导致不同的算法策略。**

例如：
- **素数筛**：解空间是整数区间，我们可以将其组织成**线性结构**（埃氏筛）或利用**线性筛**避免重复标记。
- **组合数**：解空间是组合空间，我们可以组织成**递推表**（DP）或直接利用**公式**（阶乘+逆元）计算。
- **数位DP**：解空间是数字的各位，我们可以组织成**树形结构**（按位递归），利用记忆化避免重复。

因此，在分析数学解空间时，我们关注的是将解空间构造为便于遍历的结构后的搜索和优化。


## 一、数学解空间的解的形态

数学解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构（人为构造） |
|:---|:---|:---|:---|
| **单值解** | 单个数值 | 素数判定、阶乘、快速幂、数值计算 | 数值区间 |
| **数对解** | 两个数的关系（最大公约数、最小公倍数、同余方程） | GCD、扩展欧几里得、中国剩余定理（单个方程） | 整数对 |
| **数列解** | 一系列数值 | 素数筛、递推数列（斐波那契） | 数值序列 |
| **方程解** | 满足方程组的数值（线性方程组、同余方程组） | 高斯消元、中国剩余定理（CRT） | 解集（点/线/面） |
| **组合解** | 排列组合计数（组合数、卡特兰数、斯特林数、贝尔数） | 组合数计算、卡特兰数、斯特林数 | 组合空间 |
| **数位解** | 按位拆分的数值问题（数字统计、约束条件计数） | 数位DP | 按位展开的树形 |
| **数论函数解** | 数论函数的值（欧拉函数、莫比乌斯函数）及其反演结果 | 欧拉函数筛、莫比乌斯函数筛、莫比乌斯反演 | 函数空间 |
| **多项式解** | 多项式系数序列及其运算结果（乘法、卷积） | FFT/NTT、多项式乘法、生成函数 | 多项式环 |
| **线性代数解** | 矩阵、向量、线性方程组解、线性基 | 矩阵快速幂、高斯消元、线性基 | 向量空间 |


## 二、遍历框架与遍历控制变量

数学解空间的遍历框架多样，通常与数值范围或数学结构相关。

| 解的形态 | 数据结构 | 遍历框架（模板） | 遍历控制变量（模板） |
|:---|:---|:---|:---|
| **单值解** | 数值 | 迭代（试除） | `i`（从2到sqrt(n)） |
| **单值解** | 数值 | 快速幂（二进制） | 指数 `n` 的位 |
| **数对解** | 数值 | 辗转相除 | `a`, `b` |
| **数列解** | 数组 | 筛法（埃氏筛） | `i`, `j`（倍数） |
| **数列解** | 数组 | 线性筛 | `i`，素数列表 |
| **方程解** | 矩阵 | 三层循环（高斯消元） | `k`, `i`, `j` |
| **组合解** | 数组 | 双层循环（递推） | `i`, `j` |
| **组合解** | 数值 | 直接公式 | 无 |
| **数位解** | 字符串/数字 | 递归（数位DP） | `pos`, `limit`, `lead` |
| **数论函数解** | 数组 | 线性筛 | `i`，素数列表 |
| **多项式解** | 复数/整数数组 | 分治（FFT） | 长度 `len`，位逆序 |
| **线性代数解** | 矩阵 | 快速幂（二进制） | 指数 `n` 的位 |


## 三、节点操作

节点操作是在每个被访问的“节点”上执行的具体逻辑。在数学算法中，“节点”可以是当前数值、当前位、当前方程、当前多项式系数等。

| 解的形态 | 算法 | 节点操作（模板） | 时机 |
|:---|:---|:---|:---|
| **单值解** | 素数判定 | `if n % i == 0` 返回 false | 循环内 |
| **单值解** | 快速幂 | 若当前位为1则累乘，底数自乘 | 迭代循环内 |
| **数对解** | GCD | 取模交换 | 循环内 |
| **数列解** | 埃氏筛 | 若 `isPrime[i]` 则标记倍数 | 循环内 |
| **数列解** | 线性筛 | 标记合数，若 `i % p == 0` 跳出 | 循环内 |
| **方程解** | 高斯消元 | 选主元，消元，回代 | 循环内 |
| **组合解** | 递推 | `C[i][j] = C[i-1][j-1] + C[i-1][j]` | 循环内 |
| **数位解** | 数位DP | 枚举当前位数字，递归 | 递归前 |
| **数论函数解** | 线性筛 | 根据质因数更新函数值 | 循环内 |
| **多项式解** | FFT | 蝴蝶操作 | 分治合并时 |
| **线性代数解** | 矩阵乘法 | 乘积累加 | 循环内 |


## 四、辅助变量

辅助变量用于记录状态、缓存结果，保证无冗余。

| 解的形态 | 算法 | 辅助变量（模板） | 功能分类 |
|:---|:---|:---|:---|
| **单值解** | 素数判定 | 无 | - |
| **单值解** | 快速幂 | `ans`, `base` | 临时计算 |
| **数对解** | GCD | 无 | - |
| **数列解** | 埃氏筛 | `vector<bool> isPrime` | 状态标记 |
| **数列解** | 线性筛 | `vector<int> primes`, `vector<bool> isComposite` | 结果存储、状态标记 |
| **方程解** | 高斯消元 | `vector<vector<double>> a` | 优化缓存 |
| **组合解** | 递推 | `vector<vector<int>> C` | 优化缓存 |
| **组合解** | 公式 | `vector<long long> fact, invFact` | 优化缓存 |
| **数位解** | 数位DP | `vector<vector<int>> memo` | 优化缓存 |
| **数论函数解** | 线性筛 | `vector<int> phi, mu, primes` | 结果存储、状态标记 |
| **多项式解** | FFT | `vector<complex<double>>` | 临时计算 |
| **线性代数解** | 矩阵快速幂 | `Matrix res` | 优化缓存 |

### 变量生命周期与存放位置
数学算法通常只涉及一次函数调用，辅助变量多为局部变量或类成员（如筛法中的数组可复用）。

| 类型 | 描述 | 示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次函数执行中存在 | 快速幂中的 `ans`, `base` | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 数位DP中的 `memo` | 递归参数（引用）或 lambda 捕获 |
| **类内长期** | 跨多次公共方法调用持久 | 素数筛的结果数组、线性基 | 类的成员变量 |


## 五、各子类算法详细模板

### 一、单值解

**定义**：解是单个数值，如素数判定、阶乘、快速幂结果等。

#### 暴力枚举（试除法）

##### 通用模板（数值范围）

**遍历框架（仅控制变量）**
```
for i = 2 to sqrt(n):
    节点操作：检查 n % i == 0
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：无
for i = 2 to sqrt(n):
    if n % i == 0: return false   // 节点操作：取模
return true
```

##### 数据结构：整数

**遍历框架（仅控制变量）**
```cpp
for (int i = 2; i * i <= n; ++i) {
    // 节点操作：检查 n % i == 0
}
```

**完整模板（含辅助变量）**
```cpp
bool isPrime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; ++i) {   // 遍历控制变量：i
        if (n % i == 0) return false;    // 节点操作：取模
    }
    return true;
}
// 辅助变量：无
```

---

#### 优化算法1：快速幂

**优化说明**：利用**指数二进制分解**，将 O(n) 的乘法次数降为 O(log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
while n > 0:
    if n & 1: ans = ans * base % mod
    base = base * base % mod
    n >>= 1
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（指数二进制位）
// 遍历控制变量：指数 n
// 辅助变量：ans（结果），base（当前底数幂）
ans = 1
base = a % mod
while n > 0:
    if n & 1: ans = ans * base % mod   // 节点操作：累乘
    base = base * base % mod           // 节点操作：自乘
    n >>= 1
return ans
```

##### 数据结构：整数

**遍历框架（仅控制变量）**
```cpp
long long ans = 1;
long long base = a % mod;
while (n) {
    if (n & 1) ans = ans * base % mod;
    base = base * base % mod;
    n >>= 1;
}
```

**完整模板（含辅助变量）**
```cpp
long long quickPow(long long a, long long n, long long mod) {
    long long ans = 1;                               // 辅助变量：结果
    long long base = a % mod;                        // 辅助变量：当前底数
    while (n) {                                      // 遍历控制变量：n
        if (n & 1) ans = ans * base % mod;           // 节点操作：累乘
        base = base * base % mod;                    // 节点操作：自乘
        n >>= 1;                                     // 移动遍历控制变量
    }
    return ans;
}
```


### 二、数对解

**定义**：解是两个数的关系，如最大公约数、最小公倍数、扩展欧几里得解。

#### 暴力枚举（试除）

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 1 to min(a,b):
    if a % i == 0 and b % i == 0: 记录 i
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：无
for i = 1 to min(a,b):
    if a % i == 0 and b % i == 0: gcd = i   // 节点操作：取模
return gcd
```

##### 数据结构：整数

**遍历框架（仅控制变量）**
```cpp
for (int i = 1; i <= min(a,b); ++i) {
    // 检查整除
}
```

**完整模板（含辅助变量）**
```cpp
int gcdBrute(int a, int b) {
    int g = 1;
    for (int i = 1; i <= min(a,b); ++i) {    // 遍历控制变量：i
        if (a % i == 0 && b % i == 0) {      // 节点操作：取模
            g = i;
        }
    }
    return g;
}
```

---

#### 优化算法1：辗转相除法（GCD）

**优化说明**：利用**欧几里得定理** `gcd(a,b) = gcd(b, a mod b)`，将 O(min(a,b)) 降为 O(log min(a,b))。

##### 通用模板

**遍历框架（仅控制变量）**
```
while b != 0:
    t = b
    b = a % b
    a = t
return a
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：a, b
// 辅助变量：无
while b != 0:
    t = b
    b = a % b            // 节点操作：取模
    a = t
return a
```

##### 数据结构：整数

**遍历框架（仅控制变量）**
```cpp
while (b) {
    int t = b;
    b = a % b;
    a = t;
}
```

**完整模板（含辅助变量）**
```cpp
int gcd(int a, int b) {
    while (b) {                              // 遍历控制变量：a, b
        int t = b;                           // 辅助变量：临时
        b = a % b;                           // 节点操作：取模
        a = t;
    }
    return a;
}
```

---

#### 优化算法2：扩展欧几里得算法

**优化说明**：在求 GCD 的同时求解 `ax + by = gcd(a,b)` 的整数解，利用递归后回溯更新系数。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(a, b, &x, &y):
    if b == 0: x=1, y=0; return a
    d = exgcd(b, a%b, y, x)
    y -= a/b * x
    return d
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归
// 遍历控制变量：a, b
// 辅助变量：x, y（引用参数）
int exgcd(a, b, &x, &y):
    if b == 0:
        x = 1; y = 0; return a
    d = exgcd(b, a%b, y, x)                // 节点操作：递归
    y -= a/b * x                           // 节点操作：回溯更新系数
    return d
```

##### 数据结构：整数

**遍历框架（仅控制变量）**
```cpp
int exgcd(int a, int b, int &x, int &y) {
    if (b == 0) { x = 1; y = 0; return a; }
    int d = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}
```

**完整模板（含辅助变量）**
```cpp
int exgcd(int a, int b, int &x, int &y) {
    if (b == 0) {                            // 递归出口
        x = 1; y = 0; return a;
    }
    int d = exgcd(b, a % b, y, x);           // 遍历控制变量：递归
    y -= a / b * x;                          // 节点操作：回溯
    return d;
}
// 辅助变量：x, y（引用参数）
```


### 三、数列解

**定义**：解是一系列数值，如素数列表、斐波那契数列等。

#### 暴力枚举（逐个判断）

##### 通用模板（数列生成）

**遍历框架（仅控制变量）**
```
for i = 2 to n:
    检查 i 是否为素数
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：isPrime 数组
for i = 2 to n:
    if isPrime(i): 加入列表            // 节点操作：判断
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> primes;
for (int i = 2; i <= n; ++i) {
    bool flag = true;
    for (int j = 2; j * j <= i; ++j) {
        if (i % j == 0) { flag = false; break; }
    }
    if (flag) primes.push_back(i);
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> primeListBrute(int n) {
    vector<int> primes;                          // 辅助变量：结果存储
    for (int i = 2; i <= n; ++i) {               // 遍历控制变量：i
        bool isPrime = true;
        for (int j = 2; j * j <= i; ++j) {       // 内层循环
            if (i % j == 0) { isPrime = false; break; }
        }
        if (isPrime) primes.push_back(i);        // 节点操作：加入
    }
    return primes;
}
```

---

#### 优化算法1：埃氏筛

**优化说明**：利用**倍数的性质**，从 `i*i` 开始标记合数，避免重复标记，时间复杂度 O(n log log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
isPrime[0..n] = true
for i = 2 to sqrt(n):
    if isPrime[i]:
        for j = i*i to n step i:
            isPrime[j] = false
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：isPrime（状态标记）
isPrime[0..n] = true
isPrime[0]=isPrime[1]=false
for i = 2 to sqrt(n):
    if isPrime[i]:
        for j = i*i to n step i:
            isPrime[j] = false           // 节点操作：标记合数
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<bool> isPrime(n+1, true);
isPrime[0] = isPrime[1] = false;
for (int i = 2; i * i <= n; ++i) {
    if (isPrime[i]) {
        for (int j = i * i; j <= n; j += i) {
            isPrime[j] = false;
        }
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<bool> sieve(int n) {
    vector<bool> isPrime(n+1, true);          // 辅助变量：状态标记
    isPrime[0] = isPrime[1] = false;
    for (int i = 2; i * i <= n; ++i) {        // 遍历控制变量：i
        if (isPrime[i]) {
            for (int j = i * i; j <= n; j += i) { // 遍历控制变量：j
                isPrime[j] = false;            // 节点操作：标记合数
            }
        }
    }
    return isPrime;
}
```

---

#### 优化算法2：欧拉筛（线性筛）

**优化说明**：每个合数只被其最小质因数标记一次，时间复杂度 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
isComp[0..n] = false
primes = []
for i = 2 to n:
    if not isComp[i]: primes.push(i)
    for p in primes:
        if i*p > n: break
        isComp[i*p] = true
        if i % p == 0: break
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：isComp（状态标记），primes（结果存储）
isComp[0..n] = false
primes = []
for i = 2 to n:
    if not isComp[i]: primes.push_back(i)      // 节点操作：加入素数
    for p in primes:
        if i*p > n: break
        isComp[i*p] = true                     // 节点操作：标记合数
        if i % p == 0: break                   // 节点操作：关键剪枝
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> primes;
vector<bool> isComp(n+1, false);
for (int i = 2; i <= n; ++i) {
    if (!isComp[i]) primes.push_back(i);
    for (int p : primes) {
        if (i * p > n) break;
        isComp[i * p] = true;
        if (i % p == 0) break;
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> eulerSieve(int n) {
    vector<int> primes;                          // 辅助变量：结果存储
    vector<bool> isComp(n+1, false);             // 辅助变量：状态标记
    for (int i = 2; i <= n; ++i) {               // 遍历控制变量：i
        if (!isComp[i]) primes.push_back(i);     // 节点操作：加入素数
        for (int p : primes) {                   // 遍历控制变量：p
            if (i * p > n) break;
            isComp[i * p] = true;                // 节点操作：标记合数
            if (i % p == 0) break;               // 节点操作：关键剪枝
        }
    }
    return primes;
}
```


### 四、方程解

**定义**：解是满足方程组的数值，如线性方程组、同余方程组。

#### 暴力枚举（代入检验）

##### 通用模板（小范围）

**遍历框架（仅控制变量）**
```
for x in 值域:
    检查是否满足所有方程
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：x
// 辅助变量：无
for x in 范围:
    if 满足方程: 记录 x           // 节点操作：检查
```

##### 数据结构：无

---

#### 优化算法1：高斯消元（线性方程组）

**优化说明**：利用**矩阵行变换**，将增广矩阵化为行阶梯形，时间复杂度 O(n³)。

##### 通用模板

**遍历框架（仅控制变量）**
```
for k = 0 to n-1:
    选主元，交换行
    归一化第 k 行
    for i = k+1 to n-1:
        消元
```

**完整模板（含辅助变量）**
```
// 遍历框架：三层循环
// 遍历控制变量：k, i, j
// 辅助变量：矩阵 a，向量 b
for k = 0 to n-1:
    选主元，交换行
    pivot = a[k][k]
    for j = k to n: a[k][j] /= pivot; b[k] /= pivot   // 节点操作：归一化
    for i = k+1 to n-1:
        factor = a[i][k]
        for j = k to n: a[i][j] -= factor * a[k][j]   // 节点操作：消元
        b[i] -= factor * b[k]
回代
```

##### 数据结构：矩阵（二维数组）

**遍历框架（仅控制变量）**
```cpp
for (int k = 0; k < n; ++k) {
    // 选主元
    // 归一化
    for (int i = k+1; i < n; ++i) {
        double factor = a[i][k];
        for (int j = k; j < n; ++j) {
            a[i][j] -= factor * a[k][j];
        }
        b[i] -= factor * b[k];
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<double> gauss(vector<vector<double>> a, vector<double> b) {
    int n = a.size();
    for (int k = 0; k < n; ++k) {                     // 遍历控制变量：k
        // 选主元
        int maxRow = k;
        for (int i = k+1; i < n; ++i) {
            if (abs(a[i][k]) > abs(a[maxRow][k])) maxRow = i;
        }
        swap(a[k], a[maxRow]);
        swap(b[k], b[maxRow]);
        // 归一化
        double pivot = a[k][k];
        for (int j = k; j < n; ++j) a[k][j] /= pivot; // 节点操作：归一化
        b[k] /= pivot;
        // 消元
        for (int i = k+1; i < n; ++i) {               // 遍历控制变量：i
            double factor = a[i][k];
            for (int j = k; j < n; ++j) {             // 遍历控制变量：j
                a[i][j] -= factor * a[k][j];          // 节点操作：消元
            }
            b[i] -= factor * b[k];
        }
    }
    // 回代
    vector<double> x(n);
    for (int i = n-1; i >= 0; --i) {
        x[i] = b[i];
        for (int j = i+1; j < n; ++j) {
            x[i] -= a[i][j] * x[j];
        }
    }
    return x;
}
```

---

#### 优化算法2：中国剩余定理（CRT）

**优化说明**：利用**模线性方程组**的合并公式，逐步合并同余方程，时间复杂度 O(n log M)。

##### 通用模板

**遍历框架（仅控制变量）**
```
res = a[0], mod = m[0]
for i = 1 to n-1:
    合并方程 res ≡ a[i] (mod m[i])
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：res, mod
res = a[0], mod = m[0]
for i = 1 to n-1:
    // 解 res + mod * t ≡ a[i] (mod m[i])
    求 t 满足 mod * t ≡ a[i] - res (mod m[i])
    t = 解模线性方程
    res = res + mod * t
    mod = lcm(mod, m[i])
    res %= mod
return res
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
long long res = a[0], mod = m[0];
for (int i = 1; i < n; ++i) {
    // 合并
}
```

**完整模板（含辅助变量）**
```cpp
long long crt(const vector<long long>& a, const vector<long long>& m) {
    long long res = a[0], mod = m[0];
    for (int i = 1; i < a.size(); ++i) {                // 遍历控制变量：i
        // 解方程 mod * t ≡ a[i] - res (mod m[i])
        long long c = a[i] - res;
        long long x, y;
        long long g = exgcd(mod, m[i], x, y);           // 扩展欧几里得
        if (c % g != 0) return -1;                      // 无解
        long long t = (c / g * x) % (m[i] / g);
        res = res + mod * t;
        mod = mod / g * m[i];                           // lcm
        res = (res % mod + mod) % mod;
    }
    return res;
}
```


### 五、组合解

**定义**：解是组合计数结果，如组合数、卡特兰数等。

#### 暴力枚举（递推）

##### 通用模板

**遍历框架（仅控制变量）**
```
C[0][0] = 1
for i = 1 to n:
    C[i][0] = C[i][i] = 1
    for j = 1 to i-1:
        C[i][j] = C[i-1][j-1] + C[i-1][j]
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：C 二维数组
```

##### 数据结构：二维数组

**遍历框架（仅控制变量）**
```cpp
vector<vector<int>> C(n+1, vector<int>(n+1, 0));
C[0][0] = 1;
for (int i = 1; i <= n; ++i) {
    C[i][0] = C[i][i] = 1;
    for (int j = 1; j < i; ++j) {
        C[i][j] = C[i-1][j-1] + C[i-1][j];
    }
}
```

**完整模板（含辅助变量）**
```cpp
int combinationDP(int n, int k) {
    vector<vector<int>> C(n+1, vector<int>(n+1, 0));   // 辅助变量：优化缓存
    C[0][0] = 1;
    for (int i = 1; i <= n; ++i) {                     // 遍历控制变量：i
        C[i][0] = C[i][i] = 1;
        for (int j = 1; j < i; ++j) {                  // 遍历控制变量：j
            C[i][j] = C[i-1][j-1] + C[i-1][j];         // 节点操作：递推
        }
    }
    return C[n][k];
}
```

---

#### 优化算法1：公式计算（阶乘+逆元）

**优化说明**：利用**组合数公式** `C(n,k) = n! / (k! * (n-k)!)`，通过预处理阶乘和逆元，O(1) 查询，适用于模素数。

##### 通用模板

**遍历框架（仅控制变量）**
```
预处理阶乘 fact[i] = fact[i-1] * i % mod
预处理逆元 invFact[i] = invFact[i+1] * (i+1) % mod
查询：return fact[n] * invFact[k] % mod * invFact[n-k] % mod
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（预处理）
// 遍历控制变量：i
// 辅助变量：fact, invFact
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<long long> fact(n+1), invFact(n+1);
fact[0] = 1;
for (int i = 1; i <= n; ++i) fact[i] = fact[i-1] * i % mod;
invFact[n] = quickPow(fact[n], mod-2, mod);
for (int i = n-1; i >= 0; --i) invFact[i] = invFact[i+1] * (i+1) % mod;
```

**完整模板（含辅助变量）**
```cpp
class Combination {
    vector<long long> fact, invFact;
    int mod;
public:
    Combination(int n, int mod) : mod(mod) {
        fact.resize(n+1);
        invFact.resize(n+1);
        fact[0] = 1;
        for (int i = 1; i <= n; ++i) {                // 遍历控制变量：i
            fact[i] = fact[i-1] * i % mod;            // 节点操作：阶乘
        }
        invFact[n] = quickPow(fact[n], mod-2, mod);    // 费马小定理
        for (int i = n-1; i >= 0; --i) {              // 遍历控制变量：i
            invFact[i] = invFact[i+1] * (i+1) % mod;  // 节点操作：逆元
        }
    }
    long long comb(int n, int k) {
        if (k < 0 || k > n) return 0;
        return fact[n] * invFact[k] % mod * invFact[n-k] % mod; // 节点操作：公式
    }
};
```


### 六、数位解

**定义**：解与数字的各位有关，如统计数字1的个数、不含某数字的个数等。

#### 暴力枚举（遍历所有数字）

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n:
    检查 i 的各位
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：无
for i = 0 to n:
    if 满足条件: 计数++           // 节点操作：检查各位
```

##### 数据结构：无

---

#### 优化算法1：数位DP（记忆化搜索）

**优化说明**：利用**数位分解**和**记忆化**，将 O(n) 降为 O(位数 * 状态数)，适用于范围统计。

##### 通用模板

**遍历框架（仅控制变量）**
```
递归函数(pos, limit, lead):
    if pos == len: return 1
    if !limit && !lead && memo[pos][...] != -1: return memo[pos][...]
    up = limit ? s[pos] - '0' : 9
    res = 0
    for d = 0 to up:
        res += dfs(pos+1, limit && d==up, lead && d==0)
    if !limit && !lead: memo[pos][...] = res
    return res
```

**完整模板（含辅助变量）**
```
// 遍历框架：递归 + 记忆化
// 遍历控制变量：pos, limit, lead
// 辅助变量：memo（优化缓存）
```

##### 数据结构：字符串/数字

**遍历框架（仅控制变量）**
```cpp
string s = to_string(n);
int len = s.size();
vector<vector<int>> memo(len, vector<int>(2, -1));
function<int(int, bool, bool)> dfs = [&](int pos, bool limit, bool lead) {
    if (pos == len) return 1;
    if (!limit && !lead && memo[pos][limit] != -1) return memo[pos][limit];
    int up = limit ? s[pos] - '0' : 9;
    int res = 0;
    for (int d = 0; d <= up; ++d) {
        res += dfs(pos+1, limit && d == up, lead && d == 0);
    }
    if (!limit && !lead) memo[pos][limit] = res;
    return res;
};
```

**完整模板（含辅助变量）**
```cpp
int countDigitOne(int n) {
    string s = to_string(n);
    int len = s.size();
    vector<vector<int>> memo(len, vector<int>(2, -1));      // 辅助变量：记忆化
    function<int(int, bool, bool, int)> dfs = [&](int pos, bool limit, bool lead, int cnt) -> int {
        if (pos == len) return cnt;                         // 节点操作：返回计数
        if (!limit && !lead && memo[pos][cnt] != -1) return memo[pos][cnt];
        int up = limit ? s[pos] - '0' : 9;
        int res = 0;
        for (int d = 0; d <= up; ++d) {                     // 遍历控制变量：d
            res += dfs(pos+1, limit && d == up, lead && d == 0, cnt + (d == 1));
        }
        if (!limit && !lead) memo[pos][cnt] = res;
        return res;
    };
    return dfs(0, true, true, 0);
}
```


### 七、数论函数解

**定义**：解是数论函数的值，如欧拉函数 φ(n)、莫比乌斯函数 μ(n) 等。

#### 暴力枚举（质因数分解）

##### 通用模板

**遍历框架（仅控制变量）**
```
res = n
for i = 2 to sqrt(n):
    if n % i == 0:
        while n % i == 0: n /= i
        res = res / i * (i-1)
if n > 1: res = res / n * (n-1)
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：无
```

##### 数据结构：整数

**遍历框架（仅控制变量）**
```cpp
int phi(int n) {
    int res = n;
    for (int i = 2; i * i <= n; ++i) {
        if (n % i == 0) {
            while (n % i == 0) n /= i;
            res = res / i * (i - 1);
        }
    }
    if (n > 1) res = res / n * (n - 1);
    return res;
}
```

---

#### 优化算法1：欧拉函数线性筛

**优化说明**：利用**线性筛**同时计算所有数的欧拉函数，时间复杂度 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
phi[1] = 1
for i = 2 to n:
    if not isComp[i]: primes.push(i); phi[i] = i-1
    for p in primes:
        if i*p > n: break
        isComp[i*p] = true
        if i % p == 0:
            phi[i*p] = phi[i] * p
            break
        else:
            phi[i*p] = phi[i] * (p-1)
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：i
// 辅助变量：phi 数组，primes，isComp
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> primes;
vector<bool> isComp(n+1, false);
vector<int> phi(n+1);
phi[1] = 1;
for (int i = 2; i <= n; ++i) {
    if (!isComp[i]) {
        primes.push_back(i);
        phi[i] = i - 1;
    }
    for (int p : primes) {
        if (i * p > n) break;
        isComp[i * p] = true;
        if (i % p == 0) {
            phi[i * p] = phi[i] * p;
            break;
        } else {
            phi[i * p] = phi[i] * (p - 1);
        }
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> eulerPhiSieve(int n) {
    vector<int> primes;                              // 辅助变量：素数列表
    vector<bool> isComp(n+1, false);                 // 辅助变量：状态标记
    vector<int> phi(n+1);                            // 辅助变量：结果存储
    phi[1] = 1;
    for (int i = 2; i <= n; ++i) {                   // 遍历控制变量：i
        if (!isComp[i]) {
            primes.push_back(i);
            phi[i] = i - 1;                          // 节点操作：素数 phi = i-1
        }
        for (int p : primes) {                       // 遍历控制变量：p
            if (i * p > n) break;
            isComp[i * p] = true;
            if (i % p == 0) {
                phi[i * p] = phi[i] * p;              // 节点操作：倍数情况
                break;
            } else {
                phi[i * p] = phi[i] * (p - 1);        // 节点操作：互质情况
            }
        }
    }
    return phi;
}
```


### 八、多项式解

**定义**：解是多项式系数序列或多项式运算结果，如卷积、乘法等。

#### 暴力枚举（双重循环）

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n-1:
    for j = 0 to m-1:
        c[i+j] += a[i] * b[j]
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：c 数组
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<int> c(n+m-1, 0);
for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m; ++j) {
        c[i+j] += a[i] * b[j];
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> multiplyBrute(const vector<int>& a, const vector<int>& b) {
    int n = a.size(), m = b.size();
    vector<int> c(n + m - 1, 0);                     // 辅助变量：结果存储
    for (int i = 0; i < n; ++i) {                    // 遍历控制变量：i
        for (int j = 0; j < m; ++j) {                // 遍历控制变量：j
            c[i + j] += a[i] * b[j];                 // 节点操作：累乘累加
        }
    }
    return c;
}
```

---

#### 优化算法1：FFT（快速傅里叶变换）

**优化说明**：利用**单位根**分治，将多项式乘法从 O(n²) 降为 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
将长度补到2的幂
FFT(a, false); FFT(b, false)
for i = 0 to len-1: a[i] *= b[i]
FFT(a, true)
```

**完整模板（含辅助变量）**
```
// 遍历框架：分治 + 循环
// 遍历控制变量：长度，位逆序，蝴蝶操作
// 辅助变量：复数数组，旋转因子
```

##### 数据结构：复数数组

**遍历框架（仅控制变量）**
```cpp
void fft(vector<complex<double>>& a, bool invert) {
    int n = a.size();
    for (int i = 1, j = 0; i < n; ++i) {
        int bit = n >> 1;
        for (; j & bit; bit >>= 1) j ^= bit;
        j ^= bit;
        if (i < j) swap(a[i], a[j]);
    }
    for (int len = 2; len <= n; len <<= 1) {
        double ang = 2 * M_PI / len * (invert ? -1 : 1);
        complex<double> wlen(cos(ang), sin(ang));
        for (int i = 0; i < n; i += len) {
            complex<double> w(1);
            for (int j = 0; j < len/2; ++j) {
                complex<double> u = a[i+j], v = a[i+j+len/2] * w;
                a[i+j] = u + v;
                a[i+j+len/2] = u - v;
                w *= wlen;
            }
        }
    }
    if (invert) for (int i = 0; i < n; ++i) a[i] /= n;
}
```

**完整模板（含辅助变量）**
```cpp
vector<int> multiplyFFT(const vector<int>& a, const vector<int>& b) {
    vector<complex<double>> fa(a.begin(), a.end()), fb(b.begin(), b.end());
    int n = 1;
    while (n < a.size() + b.size()) n <<= 1;
    fa.resize(n); fb.resize(n);
    fft(fa, false); fft(fb, false);                // 遍历框架：分治
    for (int i = 0; i < n; ++i) fa[i] *= fb[i];   // 节点操作：点乘
    fft(fa, true);
    vector<int> res(n);
    for (int i = 0; i < n; ++i) res[i] = round(fa[i].real());
    return res;
}
```


### 九、线性代数解

**定义**：解是矩阵、向量或线性基，如矩阵幂、线性基最大异或和等。

#### 优化算法1：矩阵快速幂

**优化说明**：利用**快速幂**思想，将矩阵乘法的幂次计算从 O(n³ log k) 优化为 O(n³ log k)。

##### 通用模板

**遍历框架（仅控制变量）**
```
res = 单位矩阵
while n > 0:
    if n & 1: res = res * a
    a = a * a
    n >>= 1
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代（指数二进制）
// 遍历控制变量：指数 n
// 辅助变量：res（结果），a（当前幂）
```

##### 数据结构：矩阵（二维数组）

**遍历框架（仅控制变量）**
```cpp
Matrix matMul(const Matrix& a, const Matrix& b) {
    int n = a.size();
    Matrix res(n, vector<long long>(n, 0));
    for (int i = 0; i < n; ++i) {
        for (int k = 0; k < n; ++k) {
            if (a[i][k] == 0) continue;
            for (int j = 0; j < n; ++j) {
                res[i][j] = (res[i][j] + a[i][k] * b[k][j]) % mod;
            }
        }
    }
    return res;
}
Matrix matPow(Matrix a, long long n, long long mod) {
    int sz = a.size();
    Matrix res(sz, vector<long long>(sz, 0));
    for (int i = 0; i < sz; ++i) res[i][i] = 1;
    while (n) {
        if (n & 1) res = matMul(res, a);
        a = matMul(a, a);
        n >>= 1;
    }
    return res;
}
```

**完整模板（含辅助变量）**
```cpp
using Matrix = vector<vector<long long>>;
Matrix matMul(const Matrix& a, const Matrix& b, long long mod) {
    int n = a.size();
    Matrix res(n, vector<long long>(n, 0));
    for (int i = 0; i < n; ++i) {                    // 遍历控制变量：i
        for (int k = 0; k < n; ++k) {                // 遍历控制变量：k
            if (a[i][k] == 0) continue;
            for (int j = 0; j < n; ++j) {            // 遍历控制变量：j
                res[i][j] = (res[i][j] + a[i][k] * b[k][j]) % mod; // 节点操作：乘积累加
            }
        }
    }
    return res;
}
Matrix matPow(Matrix a, long long n, long long mod) {
    int sz = a.size();
    Matrix res(sz, vector<long long>(sz, 0));        // 辅助变量：结果
    for (int i = 0; i < sz; ++i) res[i][i] = 1;
    while (n) {                                      // 遍历控制变量：n
        if (n & 1) res = matMul(res, a, mod);        // 节点操作：累乘
        a = matMul(a, a, mod);                       // 节点操作：自乘
        n >>= 1;                                     // 移动遍历控制变量
    }
    return res;
}
```

---

#### 优化算法2：线性基（异或空间）

**优化说明**：利用**线性基**维护向量空间，支持插入、求最大异或和等操作，时间复杂度 O(n·bits)。

##### 通用模板

**遍历框架（仅控制变量）**
```
插入：
    for i from 高到低:
        if (x >> i) & 1:
            if !basis[i]:
                basis[i] = x; break
            else: x ^= basis[i]
```

**完整模板（含辅助变量）**
```
// 遍历框架：迭代
// 遍历控制变量：位 i
// 辅助变量：basis（基数组）
```

##### 数据结构：数组

**遍历框架（仅控制变量）**
```cpp
vector<long long> basis(64, 0);
void insert(long long x) {
    for (int i = 63; i >= 0; --i) {
        if (!(x >> i)) continue;
        if (!basis[i]) { basis[i] = x; return; }
        x ^= basis[i];
    }
}
```

**完整模板（含辅助变量）**
```cpp
class LinearBase {
    vector<long long> basis;                         // 辅助变量：基
public:
    LinearBase() : basis(64, 0) {}
    void insert(long long x) {
        for (int i = 63; i >= 0; --i) {              // 遍历控制变量：i
            if (!(x >> i & 1)) continue;
            if (!basis[i]) {
                basis[i] = x;                        // 节点操作：插入
                return;
            }
            x ^= basis[i];                           // 节点操作：消元
        }
    }
    long long queryMax() {
        long long res = 0;
        for (int i = 63; i >= 0; --i) {
            if ((res ^ basis[i]) > res) res ^= basis[i]; // 节点操作：贪心取最大
        }
        return res;
    }
};
```


## 六、总结

数学解空间涵盖了从基础数论到复杂多项式运算的广泛问题。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种数学算法。本框架覆盖了素数筛、GCD、快速幂、扩展欧几里得、中国剩余定理、高斯消元、组合数、数位DP、数论函数、FFT、线性基等典型算法，并展示了每种算法如何从暴力枚举出发，利用数学性质（数论、递推、分治、线性代数）进行优化，实现从指数级或多项式级到高效算法的跨越。掌握这套框架，能够帮助你快速设计出清晰、高效的数学算法。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。