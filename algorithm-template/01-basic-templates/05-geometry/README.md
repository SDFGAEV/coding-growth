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

### 两大基本策略
| 策略 | 核心思想 | 典型几何算法 |
|:---|:---|:---|
| **遍历** | 直接搜索几何对象 | 暴力点对距离、线段相交检测 |
| **分解** | 将空间划分为子空间，分别处理 | 扫描线、分治（最近点对）、凸包（排序后扫描） |


## 一、几何解空间的解的形态

几何解空间中，解可以表现为以下数学形态：

| 解的形态 | 描述 | 典型算法 | 解空间结构 |
|:---|:---|:---|:---|
| **点关系解** | 两点之间的距离、方位、点积、叉积 | 距离计算、向量运算 | 点对集合 |
| **线关系解** | 直线/线段之间的相交、平行、垂直关系 | 线段相交判断、直线交点 | 线对集合 |
| **多边形解** | 多边形的面积、凸包、点定位、三角剖分 | 凸包（Graham scan、Andrew）、多边形面积、点在多边形内 | 多边形顶点序列 |
| **最近点对解** | 点集中距离最近的一对点 | 最近点对（分治算法） | 点对集合 |
| **扫描线解** | 多个矩形面积并、区间重叠、平面扫描 | 矩形面积并、扫描线填充 | 事件序列 |
| **最远点对解** | 点集中距离最远的一对点（旋转卡壳） | 旋转卡壳 | 点对集合 |
| **凸包解** | 点集的最小凸多边形 | Graham scan、Andrew算法 | 凸多边形顶点序列 |

**说明**：几何问题的解空间往往是连续的，但通过离散化（如点集、事件）可以转化为组合问题。扫描线法通过排序事件将二维问题降为一维处理。


## 二、遍历框架

几何解空间的遍历框架多样，取决于问题的性质。

| 框架 | 适用场景 | 特点 |
|:---|:---|:---|
| **暴力枚举** | 小规模点对关系 | 简单直接，O(n²) |
| **排序+扫描** | 凸包、最近点对、扫描线 | 先排序，后线性扫描 |
| **分治** | 最近点对 | 将平面分割，递归处理 |
| **旋转卡壳** | 最远点对、凸包性质 | 利用凸包单调性旋转 |
| **扫描线** | 矩形面积并、线段相交 | 按x坐标排序，维护y方向状态 |


## 三、遍历控制变量

在几何解空间中，遍历控制变量通常是点的索引、事件的位置、扫描线的位置等。

| 算法类型 | 典型遍历控制变量 |
|:---|:---|
| 凸包（Graham scan） | 极角排序后的点索引 |
| 凸包（Andrew） | x坐标排序后的点索引 |
| 最近点对（分治） | 点索引区间 |
| 扫描线 | 事件列表中的当前位置 |
| 旋转卡壳 | 凸包上的点索引 |


## 四、节点操作

节点操作是在每个被访问的几何对象上执行的具体逻辑。

| 算法 | 节点操作 | 时机 |
|:---|:---|:---|
| 凸包（Graham scan） | 判断栈顶两点与新点的转向 | 扫描每个点时 |
| 线段相交检测 | 跨立实验、快速排斥 | 检查每对线段时 |
| 最近点对（分治） | 合并时检查分界线附近点 | 递归返回后 |
| 扫描线 | 添加/删除线段，更新当前交点 | 处理每个事件时 |
| 旋转卡壳 | 计算当前点对距离，旋转 | 循环遍历凸包顶点 |


## 五、辅助变量

辅助变量用于记录状态、剪枝、缓存结果，保证无冗余。

### 功能分类
| 类型 | 作用 | 几何算法示例 |
|:---|:---|:---|
| **结果存储** | 收集最终答案 | 凸包点列表、最近点对距离 |
| **状态标记** | 标记已处理对象 | 扫描线中的活动线段集合 |
| **临时计算** | 暂存中间结果 | 叉积、距离 |
| **优化缓存** | 加速重复计算 | 按x排序后的点数组 |

### 变量生命周期与存放位置
几何算法通常以函数形式实现，辅助变量多为局部数组或临时变量。

| 类型 | 描述 | 几何算法示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次算法执行中存在 | 排序后的点数组、栈 | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 分治中的临时点数组 | 递归参数（引用） |
| **类内长期** | 跨多次公共方法调用持久 | 几何对象集合 | 类的成员变量 |


## 六、基础几何工具

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

// 三点叉积：判断 p1p2 到 p1p3 的转向
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


## 七、具体算法

### 一、点关系解

#### 1.1 两点距离
- **解空间结构**：所有点对
- **遍历框架**：直接计算
- **遍历控制变量**：无
- **节点操作**：`sqrt((x1-x2)² + (y1-y2)²)`
- **辅助变量**：无
```cpp
double distance(const Point& a, const Point& b) {
    return (a - b).len();
}
```


### 二、线关系解

#### 2.1 线段相交判断
- **解空间结构**：所有线段对
- **遍历框架**：暴力枚举或扫描线
- **遍历控制变量**：线段索引
- **节点操作**：跨立实验
- **辅助变量**：无
```cpp
bool anyIntersection(vector<pair<Point, Point>>& segments) {
    int n = segments.size();
    for (int i = 0; i < n; ++i) {
        for (int j = i+1; j < n; ++j) {
            if (segmentsIntersect(segments[i].first, segments[i].second,
                                  segments[j].first, segments[j].second)) {
                return true;
            }
        }
    }
    return false;
}
```


### 三、凸包解

#### 3.1 Graham scan 凸包
- **解空间结构**：点集的凸多边形
- **遍历框架**：排序 + 栈扫描
- **遍历控制变量**：按极角排序后的点索引
- **节点操作**：判断栈顶两点与当前点的转向，决定出栈/入栈
- **辅助变量**：栈
```cpp
vector<Point> convexHullGraham(vector<Point>& pts) {
    int n = pts.size();
    if (n <= 1) return pts;
    // 找最下最左的点
    int k = 0;
    for (int i = 1; i < n; ++i) {
        if (pts[i].y < pts[k].y || (pts[i].y == pts[k].y && pts[i].x < pts[k].x)) k = i;
    }
    swap(pts[0], pts[k]);
    // 按极角排序
    sort(pts.begin() + 1, pts.end(), [&](const Point& a, const Point& b) {
        double cp = cross(pts[0], a, b);
        if (cp != 0) return cp > 0;
        return (a - pts[0]).len2() < (b - pts[0]).len2();
    });
    vector<Point> stk;
    stk.push_back(pts[0]);
    stk.push_back(pts[1]);
    for (int i = 2; i < n; ++i) {
        while (stk.size() >= 2 && cross(stk[stk.size()-2], stk.back(), pts[i]) <= 0) {
            stk.pop_back();
        }
        stk.push_back(pts[i]);
    }
    return stk;
}
```

#### 3.2 Andrew 算法凸包
- **解空间结构**：点集的凸多边形
- **遍历框架**：排序 + 两次扫描
- **遍历控制变量**：按x坐标排序后的点索引
- **节点操作**：判断转向，构造上凸包和下凸包
- **辅助变量**：栈
```cpp
vector<Point> convexHullAndrew(vector<Point>& pts) {
    sort(pts.begin(), pts.end());
    vector<Point> lower, upper;
    for (const Point& p : pts) {
        while (lower.size() >= 2 && cross(lower[lower.size()-2], lower.back(), p) <= 0) lower.pop_back();
        lower.push_back(p);
    }
    for (int i = pts.size()-1; i >= 0; --i) {
        const Point& p = pts[i];
        while (upper.size() >= 2 && cross(upper[upper.size()-2], upper.back(), p) <= 0) upper.pop_back();
        upper.push_back(p);
    }
    lower.insert(lower.end(), upper.begin()+1, upper.end()-1);
    return lower;
}
```


### 四、最近点对解

#### 4.1 分治法求最近点对
- **解空间结构**：所有点对
- **遍历框架**：分治
- **遍历控制变量**：点索引区间 `[l, r]`
- **节点操作**：递归求左右最小，合并时检查分界线附近点
- **辅助变量**：按y排序的临时数组
```cpp
double closestPair(vector<Point>& pts, int l, int r) {
    if (l >= r) return 1e20;
    int mid = (l + r) / 2;
    double d = min(closestPair(pts, l, mid), closestPair(pts, mid+1, r));
    // 合并左右
    vector<Point> tmp;
    for (int i = l; i <= r; ++i) {
        if (fabs(pts[i].x - pts[mid].x) < d) tmp.push_back(pts[i]);
    }
    sort(tmp.begin(), tmp.end(), [](const Point& a, const Point& b) { return a.y < b.y; });
    for (int i = 0; i < tmp.size(); ++i) {
        for (int j = i+1; j < tmp.size() && tmp[j].y - tmp[i].y < d; ++j) {
            d = min(d, (tmp[i] - tmp[j]).len());
        }
    }
    return d;
}
double closestPair(vector<Point>& pts) {
    sort(pts.begin(), pts.end(), [](const Point& a, const Point& b) { return a.x < b.x; });
    return closestPair(pts, 0, pts.size()-1);
}
```


### 五、最远点对解（旋转卡壳）

#### 5.1 旋转卡壳求凸包直径
- **解空间结构**：凸包上的点对
- **遍历框架**：旋转卡壳
- **遍历控制变量**：凸包上的点索引
- **节点操作**：当对踵点距离不再增加时旋转
- **辅助变量**：无
```cpp
double rotatingCalipers(const vector<Point>& hull) {
    int n = hull.size();
    if (n <= 1) return 0;
    if (n == 2) return (hull[0] - hull[1]).len();
    int j = 1;
    double res = 0;
    for (int i = 0; i < n; ++i) {
        while (cross(hull[i], hull[(i+1)%n], hull[(j+1)%n]) > cross(hull[i], hull[(i+1)%n], hull[j])) {
            j = (j+1) % n;
        }
        res = max(res, max((hull[i] - hull[j]).len(), (hull[(i+1)%n] - hull[(j+1)%n]).len()));
    }
    return res;
}
```


### 六、扫描线解

#### 6.1 矩形面积并（扫描线 + 线段树）
- **解空间结构**：多个矩形的并集面积
- **遍历框架**：扫描线（按x排序事件）
- **遍历控制变量**：事件列表中的当前位置
- **节点操作**：添加或删除y区间，更新当前覆盖长度
- **辅助变量**：线段树（维护y轴覆盖长度），事件列表
```cpp
struct Event {
    double x, y1, y2;
    int type; // 1: 左边界, -1: 右边界
    bool operator<(const Event& other) const { return x < other.x; }
};
double rectangleUnion(vector<pair<Point, Point>>& rects) {
    vector<double> ys;
    vector<Event> events;
    for (auto& r : rects) {
        double x1 = r.first.x, y1 = r.first.y, x2 = r.second.x, y2 = r.second.y;
        ys.push_back(y1); ys.push_back(y2);
        events.push_back({x1, y1, y2, 1});
        events.push_back({x2, y1, y2, -1});
    }
    sort(ys.begin(), ys.end());
    ys.erase(unique(ys.begin(), ys.end()), ys.end());
    sort(events.begin(), events.end());
    
    // 线段树实现省略（需要支持区间覆盖计数和长度查询）
    // 遍历事件，更新当前覆盖长度，累加面积 (x_next - x_cur) * 覆盖长度
    double area = 0;
    double lastX = 0;
    for (int i = 0; i < events.size(); ++i) {
        if (i > 0 && events[i].x > lastX) {
            area += (events[i].x - lastX) * queryCoveredLength();
        }
        update(events[i].y1, events[i].y2, events[i].type); // 更新覆盖计数
        lastX = events[i].x;
    }
    return area;
}
```


### 七、多边形解

#### 7.1 点在多边形内（射线法）
- **解空间结构**：多边形与点的位置关系
- **遍历框架**：遍历多边形每条边
- **遍历控制变量**：边索引
- **节点操作**：判断射线与边是否相交，计数
- **辅助变量**：无
```cpp
bool pointInPolygon(const Point& p, const vector<Point>& poly) {
    int n = poly.size();
    int cnt = 0;
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

#### 7.2 多边形面积
- **解空间结构**：多边形的面积
- **遍历框架**：遍历所有顶点
- **遍历控制变量**：顶点索引
- **节点操作**：累加叉积
- **辅助变量**：无
```cpp
double polygonArea(const vector<Point>& poly) {
    double area = 0;
    int n = poly.size();
    for (int i = 0; i < n; ++i) {
        area += poly[i].cross(poly[(i+1)%n]);
    }
    return fabs(area) / 2;
}
```


## 八、变量生命周期与辅助变量存放位置（在几何算法中）

几何算法中，辅助变量常为局部数组或栈，递归时需注意共享临时空间。

| 类型 | 描述 | 几何算法示例 | 存放位置 |
|:---|:---|:---|:---|
| **一次调用内** | 仅在单次算法执行中存在 | 排序后的点数组、栈 | 函数局部变量 |
| **递归内共享** | 在一次递归过程中共享 | 分治中的临时点数组 | 递归参数（引用） |
| **类内长期** | 跨多次公共方法调用持久 | 点集、多边形对象 | 类的成员变量 |


## 九、总结

几何解空间是计算几何的核心内容，涉及点、线、多边形等基本几何对象之间的关系与性质。通过将算法分解为**解的形态、遍历框架、遍历控制变量、节点操作、辅助变量**五个要素，我们可以系统化地理解和实现各种几何算法。本框架涵盖了凸包、最近点对、线段相交、多边形面积、扫描线等典型几何算法，掌握它们能够帮助你应对大多数计算几何问题。

---

**贡献**：欢迎提出 issues 和 PRs，共同完善本框架。