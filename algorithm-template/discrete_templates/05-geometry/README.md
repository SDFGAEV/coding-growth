# 几何解空间：算法认知框架

> 本文档是“算法认知框架”系列的第六部分，聚焦于**几何解空间**的算法设计与实现。

## 核心理念

### 几何解空间定义
几何解空间由**几何对象**（点、线、线段、多边形、圆等）构成，解可以是这些对象之间的关系、性质或构造。几何问题通常涉及位置、距离、角度、相交、包含等空间关系。

**基础元素**：点、向量、直线、线段、多边形、圆等。

### 算法本质（在几何解空间中的体现）
**算法 = 在解空间上的聪明穷举 + 辅助变量优化**，对于几何解空间：
- **无遗漏**：必须考虑所有可能的几何对象或关系（如所有点对、所有线段交点等）
- **无冗余**：利用几何性质（凸性、单调性、空间划分）避免重复计算

### 解空间的人为构造
解空间本身是问题的所有可能解的集合，其结构是抽象的。但在设计算法时，我们可以人为地为解空间赋予一种便于遍历的结构。**同一问题的解空间可以有多种不同的构造方式，选择不同的结构会导致不同的算法策略。**

例如：
- **凸包**：原始解空间是所有子集（指数级），我们可以将点按极角排序，构造为**凸多边形**（线性结构），通过一次扫描得到凸包。
- **最近点对**：原始解空间是所有点对（O(n²)），我们可以利用**分治**将平面分割，在合并时只检查分界线附近常数个点，降为 O(n log n)。

因此，在分析几何解空间时，我们关注的是将解空间构造为便于遍历的结构后的搜索和优化。


## 一、几何解空间的解的形态

几何解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构（人为构造） |
|:---|:---|:---|:---|
| **点关系解** | 两点之间的距离、方位、点积、叉积 | 距离计算、向量运算 | 点对集合（二维） |
| **线关系解** | 直线/线段之间的相交、平行、垂直关系 | 线段相交判断、直线交点 | 线对集合 |
| **多边形解** | 多边形的面积、凸包、点定位、三角剖分 | 凸包（Graham scan、Andrew）、多边形面积、点在多边形内 | 多边形顶点序列（线性） |
| **最近点对解** | 点集中距离最近的一对点 | 最近点对（分治算法） | 点对集合（二维） |
| **最远点对解** | 点集中距离最远的一对点（旋转卡壳） | 旋转卡壳 | 凸包上的点对 |
| **扫描线解** | 多个矩形面积并、区间重叠、平面扫描 | 矩形面积并、扫描线填充 | 事件序列（线性） |
| **凸包解** | 点集的最小凸多边形 | Graham scan、Andrew算法 | 凸多边形顶点序列（线性） |


## 二、基础几何工具

在实现几何算法前，需要定义基本的数据结构和运算。

```cpp
struct Point {
    double x, y;
    Point(double x = 0, double y = 0) : x(x), y(y) {}
    Point operator+(const Point& p) const { return Point(x + p.x, y + p.y); }
    Point operator-(const Point& p) const { return Point(x - p.x, y - p.y); }
    Point operator*(double k) const { return Point(x * k, y * k); }
    Point operator/(double k) const { return Point(x / k, y / k); }
    double dot(const Point& p) const { return x * p.x + y * p.y; }
    double cross(const Point& p) const { return x * p.y - y * p.x; }
    double len2() const { return x * x + y * y; }
    double len() const { return sqrt(len2()); }
    bool operator<(const Point& p) const { return x != p.x ? x < p.x : y < p.y; }
};

// 三点叉积：判断 p1p2 到 p1p3 的转向（正=左转，负=右转，0=共线）
double cross(const Point& p1, const Point& p2, const Point& p3) {
    return (p2 - p1).cross(p3 - p1);
}

// 点在线段上的判断（含端点）
bool onSegment(const Point& p, const Point& a, const Point& b) {
    return (a - p).cross(b - p) == 0 && (p.x - a.x) * (p.x - b.x) <= 0 && (p.y - a.y) * (p.y - b.y) <= 0;
}

// 线段相交判断（含端点）
bool segmentsIntersect(const Point& a1, const Point& a2, const Point& b1, const Point& b2) {
    double c1 = cross(a1, a2, b1), c2 = cross(a1, a2, b2);
    double c3 = cross(b1, b2, a1), c4 = cross(b1, b2, a2);
    if (c1 == 0 && c2 == 0 && c3 == 0 && c4 == 0) {
        // 共线情况，检查投影
        if (max(a1.x, a2.x) < min(b1.x, b2.x) || max(b1.x, b2.x) < min(a1.x, a2.x)) return false;
        if (max(a1.y, a2.y) < min(b1.y, b2.y) || max(b1.y, b2.y) < min(a1.y, a2.y)) return false;
        return true;
    }
    return (c1 * c2 <= 0) && (c3 * c4 <= 0);
}
```


## 三、遍历框架与遍历控制变量

几何解空间的遍历框架多样，取决于问题的性质。

| 解的形态 | 数据结构 | 遍历框架（模板） | 遍历控制变量（模板） |
|:---|:---|:---|:---|
| **点关系解** | 点集 | 双层循环 | `i`, `j` |
| **点关系解** | 点集 | 分治 | 递归区间 `[l, r]` |
| **线关系解** | 线段集 | 双重循环或扫描线 | 线段索引，事件位置 |
| **凸包解** | 点集 | 排序 + 栈扫描 | 排序后的点索引 |
| **多边形解** | 多边形 | 遍历顶点 | 顶点索引 `i` |
| **最近点对** | 点集 | 分治 | 点索引区间 |
| **扫描线解** | 矩形集 | 排序 + 遍历事件 | 事件列表中的位置 |


## 四、节点操作

节点操作是在每个被访问的几何对象上执行的具体逻辑。

| 解的形态 | 算法 | 节点操作（模板） | 时机 |
|:---|:---|:---|:---|
| **点关系解** | 暴力枚举 | 计算距离或叉积 | 循环内 |
| **凸包解** | Graham scan | 判断栈顶三点转向 | 扫描每个点时 |
| **凸包解** | Andrew | 判断转向并维护栈 | 扫描每个点时 |
| **线段相交** | 暴力 | 跨立实验 | 检查每对线段时 |
| **最近点对** | 分治 | 合并时检查分界线附近点 | 递归返回后 |
| **扫描线** | 矩形面积并 | 添加/删除线段，更新覆盖长度 | 处理每个事件时 |
| **旋转卡壳** | 凸包直径 | 计算当前点对距离，旋转 | 循环遍历凸包顶点 |


## 五、辅助变量

辅助变量用于记录状态、剪枝、缓存结果，保证无冗余。

| 解的形态 | 算法 | 辅助变量（模板） | 功能分类 |
|:---|:---|:---|:---|
| **点关系解** | 暴力枚举 | 无 | - |
| **凸包解** | Graham scan | `vector<Point> stk` | 路径记录 |
| **凸包解** | Andrew | `vector<Point> lower, upper` | 临时存储 |
| **最近点对** | 分治 | `vector<Point> temp` | 临时计算 |
| **扫描线** | 矩形面积并 | 线段树（覆盖长度） | 优化缓存 |
| **扫描线** | 矩形面积并 | `vector<Event>` | 事件列表 |
| **多边形解** | 点在多边形内 | 无（或计数器） | 临时计算 |


## 六、各子类算法详细模板

### 一、点关系解

**定义**：解是两个点之间的关系，如距离、方位、叉积等。

#### 暴力枚举（所有点对）

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n-1:
    for j = i+1 to n-1:
        节点操作：计算距离或叉积
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：ans（结果存储）
ans = INF
for i = 0 to n-1:
    for j = i+1 to n-1:
        d = dist(p[i], p[j])               // 节点操作：计算距离
        ans = min(ans, d)
return ans
```

##### 数据结构：点集

**遍历框架（仅控制变量）**
```cpp
double ans = INF;
for (int i = 0; i < n; ++i) {
    for (int j = i+1; j < n; ++j) {
        double d = hypot(p[i].x - p[j].x, p[i].y - p[j].y);
        ans = min(ans, d);
    }
}
```

**完整模板（含辅助变量）**
```cpp
double closestPairBrute(const vector<Point>& pts) {
    int n = pts.size();
    double ans = 1e20;                                // 辅助变量：结果存储
    for (int i = 0; i < n; ++i) {                     // 遍历控制变量：i
        for (int j = i+1; j < n; ++j) {               // 遍历控制变量：j
            double d = hypot(pts[i].x - pts[j].x, pts[i].y - pts[j].y); // 节点操作：距离
            ans = min(ans, d);
        }
    }
    return ans;
}
```


### 二、线关系解

**定义**：解是直线/线段之间的关系，如相交、平行、交点等。

#### 暴力枚举（所有线段对）

##### 通用模板

**遍历框架（仅控制变量）**
```
for i = 0 to n-1:
    for j = i+1 to n-1:
        节点操作：检查线段是否相交
```

**完整模板（含辅助变量）**
```
// 遍历框架：双层循环
// 遍历控制变量：i, j
// 辅助变量：结果集
for i = 0 to n-1:
    for j = i+1 to n-1:
        if intersect(seg[i], seg[j]): 记录        // 节点操作：相交判断
```

##### 数据结构：线段集

**遍历框架（仅控制变量）**
```cpp
for (int i = 0; i < n; ++i) {
    for (int j = i+1; j < n; ++j) {
        if (segmentsIntersect(seg[i].first, seg[i].second, seg[j].first, seg[j].second)) {
            // 记录交点
        }
    }
}
```

**完整模板（含辅助变量）**
```cpp
vector<pair<int,int>> findAllIntersections(const vector<pair<Point,Point>>& segs) {
    int n = segs.size();
    vector<pair<int,int>> res;                       // 辅助变量：结果存储
    for (int i = 0; i < n; ++i) {                    // 遍历控制变量：i
        for (int j = i+1; j < n; ++j) {              // 遍历控制变量：j
            if (segmentsIntersect(segs[i].first, segs[i].second,
                                  segs[j].first, segs[j].second)) { // 节点操作：相交判断
                res.emplace_back(i, j);
            }
        }
    }
    return res;
}
```


### 三、凸包解

**定义**：解是点集的最小凸多边形。

#### 优化算法1：Graham scan

**优化说明**：利用**极角排序**，通过栈维护凸包，时间复杂度 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
找最下最左点 p0
按极角排序其余点
栈 push(p0), push(p1)
for i = 2 to n-1:
    while 栈大小>=2 and 栈顶两点与 pts[i] 不构成左转:
        栈 pop
    栈 push(pts[i])
```

**完整模板（含辅助变量）**
```
// 遍历框架：排序 + 栈扫描
// 遍历控制变量：i
// 辅助变量：栈 stk
```

##### 数据结构：点集

**遍历框架（仅控制变量）**
```cpp
vector<Point> convexHullGraham(vector<Point> pts) {
    int n = pts.size();
    if (n <= 1) return pts;
    // 找最下最左点
    int k = 0;
    for (int i = 1; i < n; ++i) {
        if (pts[i].y < pts[k].y || (pts[i].y == pts[k].y && pts[i].x < pts[k].x)) k = i;
    }
    swap(pts[0], pts[k]);
    // 极角排序
    sort(pts.begin()+1, pts.end(), [&](const Point& a, const Point& b) {
        double cp = cross(pts[0], a, b);
        if (cp != 0) return cp > 0;
        return (a - pts[0]).len2() < (b - pts[0]).len2();
    });
    vector<Point> stk;
    stk.push_back(pts[0]);
    stk.push_back(pts[1]);
    for (int i = 2; i < n; ++i) {                     // 遍历控制变量：i
        while (stk.size() >= 2 && cross(stk[stk.size()-2], stk.back(), pts[i]) <= 0) {
            stk.pop_back();                           // 节点操作：出栈
        }
        stk.push_back(pts[i]);                        // 节点操作：入栈
    }
    return stk;
}
```

---

#### 优化算法2：Andrew 算法

**优化说明**：利用**x坐标排序**，分别构造上下凸包，时间复杂度 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
按 x 排序点集
构造下凸包：从左到右扫描
构造上凸包：从右到左扫描
合并（去掉重复端点）
```

**完整模板（含辅助变量）**
```
// 遍历框架：排序 + 扫描
// 遍历控制变量：i
// 辅助变量：lower, upper 栈
```

##### 数据结构：点集

**遍历框架（仅控制变量）**
```cpp
vector<Point> convexHullAndrew(vector<Point> pts) {
    sort(pts.begin(), pts.end());
    vector<Point> lower, upper;
    for (int i = 0; i < pts.size(); ++i) {            // 遍历控制变量：i
        while (lower.size() >= 2 && cross(lower[lower.size()-2], lower.back(), pts[i]) <= 0) {
            lower.pop_back();
        }
        lower.push_back(pts[i]);
    }
    for (int i = pts.size()-1; i >= 0; --i) {         // 遍历控制变量：i
        while (upper.size() >= 2 && cross(upper[upper.size()-2], upper.back(), pts[i]) <= 0) {
            upper.pop_back();
        }
        upper.push_back(pts[i]);
    }
    lower.pop_back(); upper.pop_back();
    lower.insert(lower.end(), upper.begin(), upper.end());
    return lower;
}
```


### 四、最近点对解

#### 优化算法1：分治法

**优化说明**：利用**分治**思想，将点集按 x 坐标分割，合并时只检查分界线附近常数个点，将 O(n²) 降为 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
按 x 排序
分治(l, r):
    if r-l <= 3: 暴力求解
    mid = (l+r)/2
    d = min(分治(l,mid), 分治(mid+1,r))
    取 x 坐标在 [mid.x-d, mid.x+d] 的点，按 y 排序
    for i in 该区域:
        for j = i+1 to i+7:
            d = min(d, dist)
    return d
```

**完整模板（含辅助变量）**
```
// 遍历框架：分治递归
// 遍历控制变量：区间 [l, r]
// 辅助变量：临时点数组 temp
```

##### 数据结构：点集

**遍历框架（仅控制变量）**
```cpp
double closestPair(vector<Point>& pts, int l, int r) {
    if (r - l <= 3) {
        double d = 1e20;
        for (int i = l; i <= r; ++i)
            for (int j = i+1; j <= r; ++j)
                d = min(d, (pts[i] - pts[j]).len());
        return d;
    }
    int mid = (l + r) / 2;
    double d = min(closestPair(pts, l, mid), closestPair(pts, mid+1, r));
    vector<Point> strip;
    for (int i = l; i <= r; ++i) {
        if (fabs(pts[i].x - pts[mid].x) < d) strip.push_back(pts[i]);
    }
    sort(strip.begin(), strip.end(), [](const Point& a, const Point& b) { return a.y < b.y; });
    for (int i = 0; i < strip.size(); ++i) {
        for (int j = i+1; j < strip.size() && strip[j].y - strip[i].y < d; ++j) {
            d = min(d, (strip[i] - strip[j]).len());
        }
    }
    return d;
}
```

**完整模板（含辅助变量）**
```cpp
double closestPair(vector<Point>& pts) {
    sort(pts.begin(), pts.end(), [](const Point& a, const Point& b) { return a.x < b.x; });
    vector<Point> temp(pts.size());                  // 辅助变量：临时数组
    function<double(int,int)> rec = [&](int l, int r) -> double {
        if (r - l <= 3) {
            double d = 1e20;
            for (int i = l; i <= r; ++i)
                for (int j = i+1; j <= r; ++j)
                    d = min(d, (pts[i] - pts[j]).len());
            return d;
        }
        int mid = (l + r) / 2;
        double d = min(rec(l, mid), rec(mid+1, r));
        vector<Point> strip;
        for (int i = l; i <= r; ++i) {
            if (fabs(pts[i].x - pts[mid].x) < d) strip.push_back(pts[i]);
        }
        sort(strip.begin(), strip.end(), [](const Point& a, const Point& b) { return a.y < b.y; });
        for (int i = 0; i < strip.size(); ++i) {
            for (int j = i+1; j < strip.size() && strip[j].y - strip[i].y < d; ++j) {
                d = min(d, (strip[i] - strip[j]).len());
            }
        }
        return d;
    };
    return rec(0, pts.size()-1);
}
```


### 五、多边形解

#### 5.1 点在多边形内（射线法）

##### 通用模板

**遍历框架（仅控制变量）**
```
cnt = 0
for i = 0 to n-1:
    a = poly[i], b = poly[(i+1)%n]
    if 点在线段上: return true
    if ((a.y > p.y) != (b.y > p.y)):
        x = (b.x - a.x)*(p.y - a.y)/(b.y - a.y) + a.x
        if x > p.x: cnt++
return cnt % 2 == 1
```

**完整模板（含辅助变量）**
```
// 遍历框架：循环遍历多边形的边
// 遍历控制变量：i
// 辅助变量：cnt（计数器）
```

##### 数据结构：多边形

**遍历框架（仅控制变量）**
```cpp
bool pointInPolygon(const Point& p, const vector<Point>& poly) {
    int n = poly.size(), cnt = 0;
    for (int i = 0; i < n; ++i) {
        Point a = poly[i], b = poly[(i+1)%n];
        if (onSegment(p, a, b)) return true;
        if ((a.y > p.y) != (b.y > p.y)) {
            double x = (b.x - a.x) * (p.y - a.y) / (b.y - a.y) + a.x;
            if (x > p.x) cnt++;
        }
    }
    return cnt % 2;
}
```

---

#### 5.2 多边形面积

##### 通用模板

**遍历框架（仅控制变量）**
```
area = 0
for i = 0 to n-1:
    area += poly[i].x * poly[(i+1)%n].y - poly[i].y * poly[(i+1)%n].x
return abs(area)/2
```

**完整模板（含辅助变量）**
```
// 遍历框架：循环遍历顶点
// 遍历控制变量：i
// 辅助变量：area
```

##### 数据结构：多边形

**遍历框架（仅控制变量）**
```cpp
double polygonArea(const vector<Point>& poly) {
    double area = 0;
    int n = poly.size();
    for (int i = 0; i < n; ++i) {
        area += poly[i].x * poly[(i+1)%n].y - poly[i].y * poly[(i+1)%n].x;
    }
    return fabs(area) / 2;
}
```


### 六、最远点对解（旋转卡壳）

#### 优化算法1：凸包直径

**优化说明**：利用**旋转卡壳**，在凸包上通过旋转找到对踵点，将 O(n²) 降为 O(n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
hull = 凸包
n = hull.size()
if n == 2: return dist(hull[0], hull[1])
j = 1
ans = 0
for i = 0 to n-1:
    while cross(hull[i], hull[(i+1)%n], hull[(j+1)%n]) > cross(hull[i], hull[(i+1)%n], hull[j]):
        j = (j+1) % n
    ans = max(ans, (hull[i] - hull[j]).len(), (hull[(i+1)%n] - hull[(j+1)%n]).len())
return ans
```

**完整模板（含辅助变量）**
```
// 遍历框架：循环遍历凸包顶点
// 遍历控制变量：i, j
// 辅助变量：ans（结果存储）
```

##### 数据结构：凸包

**遍历框架（仅控制变量）**
```cpp
double rotatingCalipers(const vector<Point>& hull) {
    int n = hull.size();
    if (n <= 1) return 0;
    if (n == 2) return (hull[0] - hull[1]).len();
    int j = 1;
    double ans = 0;
    for (int i = 0; i < n; ++i) {                     // 遍历控制变量：i
        while (cross(hull[i], hull[(i+1)%n], hull[(j+1)%n]) > 
               cross(hull[i], hull[(i+1)%n], hull[j])) {
            j = (j + 1) % n;                          // 节点操作：旋转
        }
        ans = max(ans, (hull[i] - hull[j]).len());
        ans = max(ans, (hull[(i+1)%n] - hull[(j+1)%n]).len());
    }
    return ans;
}
```


### 七、扫描线解

#### 优化算法1：矩形面积并

**优化说明**：利用**扫描线** + **线段树**，将二维面积计算转化为一维区间覆盖长度累加，时间复杂度 O(n log n)。

##### 通用模板

**遍历框架（仅控制变量）**
```
构建事件列表（x 坐标，y1, y2，类型）
按 x 排序
遍历事件：
    更新覆盖长度
    面积 += (x_next - x_cur) * 当前覆盖长度
```

**完整模板（含辅助变量）**
```
// 遍历框架：排序 + 遍历事件
// 遍历控制变量：事件索引
// 辅助变量：事件列表，线段树
```

##### 数据结构：矩形集

**遍历框架（仅控制变量）**
```cpp
struct Event {
    double x, y1, y2;
    int type; // 1: 左边界, -1: 右边界
    bool operator<(const Event& other) const { return x < other.x; }
};
double rectangleUnion(vector<tuple<double,double,double,double>>& rects) {
    vector<double> ys;
    vector<Event> events;
    for (auto& [x1, y1, x2, y2] : rects) {
        ys.push_back(y1); ys.push_back(y2);
        events.push_back({x1, y1, y2, 1});
        events.push_back({x2, y1, y2, -1});
    }
    sort(ys.begin(), ys.end());
    ys.erase(unique(ys.begin(), ys.end()), ys.end());
    sort(events.begin(), events.end());
    // 线段树维护覆盖长度（略）
    double area = 0, lastX = events[0].x;
    for (int i = 0; i < events.size(); ++i) {
        if (i > 0) {
            double len = queryCoveredLength();        // 节点操作：查询当前覆盖长度
            area += (events[i].x - lastX) * len;
        }
        update(events[i].y1, events[i].y2, events[i].type); // 节点操作：更新覆盖
        lastX = events[i].x;
    }
    return area;
}
```


## 七、变量生命周期与存放位置（在几何算法中）

几何算法中，辅助变量常为局部数组或栈，递归时需注意共享临时空间。

| 类型 | 描述 | 几何算法示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次算法执行中存在 | 排序后的点数组、栈 | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 分治中的临时点数组 | 递归参数（引用） |
| **类内长期** | 跨多次公共方法调用持久 | 点集、多边形对象 | 类的成员变量 |


## 八、总结

几何解空间是计算几何的核心内容，涉及点、线、多边形等基本几何对象之间的关系与性质。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种几何算法。本框架涵盖了凸包、最近点对、线段相交、多边形面积、扫描线等典型几何算法，并展示了每种算法如何从暴力枚举出发，利用几何性质（凸性、单调性、分治）进行优化，实现从指数级或 O(n²) 到 O(n log n) 的跨越。掌握这套框架，能够帮助你快速设计出清晰、高效的几何算法。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。