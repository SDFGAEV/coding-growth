线性结构（一对一）
解空间为线性序列，元素之间有先后关系，常用迭代遍历。

1.1 数组（连续存储）
1.1.1 顺序遍历
遍历框架：迭代（for 循环）

遍历控制变量：索引 i

节点操作：访问元素（如累加、打印）

辅助变量：视需求而定（如累加和）

cpp
for (int i = 0; i < n; ++i) {
    // 节点操作：处理 nums[i]
}
1.1.2 二分查找
遍历框架：迭代（while）

遍历控制变量：left, right, mid

节点操作：比较 nums[mid] 与目标

辅助变量：无

cpp
int left = 0, right = n-1;
while (left <= right) {
    int mid = left + (right-left)/2;
    if (nums[mid] == target) return mid;
    else if (nums[mid] < target) left = mid+1;
    else right = mid-1;
}
1.1.3 双指针（左右指针）
遍历框架：迭代（while）

遍历控制变量：left, right

节点操作：比较并移动

辅助变量：临时 sum

cpp
int left = 0, right = n-1;
while (left < right) {
    int sum = nums[left] + nums[right];
    if (sum == target) return {left+1, right+1};
    else if (sum < target) left++;
    else right--;
}
1.1.4 双指针（快慢指针）
遍历框架：迭代（for/while）

遍历控制变量：slow, fast

节点操作：赋值或交换

辅助变量：慢指针记录位置

cpp
int slow = 0;
for (int fast = 0; fast < n; ++fast) {
    if (nums[fast] != 0) {
        nums[slow++] = nums[fast];
    }
}
1.1.5 滑动窗口
遍历框架：迭代（右指针扩展，左指针收缩）

遍历控制变量：left, right

节点操作：加入右元素，收缩左元素，更新结果

辅助变量：窗口状态（如哈希表）

cpp
unordered_set<char> window;
int left = 0, maxLen = 0;
for (int right = 0; right < s.size(); ++right) {
    while (window.count(s[right])) {
        window.erase(s[left]);
        left++;
    }
    window.insert(s[right]);
    maxLen = max(maxLen, right-left+1);
}
1.1.6 前缀和
遍历框架：迭代（构建时）

遍历控制变量：i

节点操作：prefix[i] = prefix[i-1] + nums[i-1]

辅助变量：prefix 数组

cpp
vector<int> prefix(n+1);
for (int i = 1; i <= n; ++i) {
    prefix[i] = prefix[i-1] + nums[i-1];
}
1.1.7 差分
遍历框架：迭代（更新时 O(1)，还原时遍历）

遍历控制变量：i

节点操作：更新 diff[l] += c; diff[r+1] -= c; 还原时累加

辅助变量：diff 数组

cpp
vector<int> diff(n+1);
diff[l] += c; diff[r+1] -= c;
// 还原
vector<int> res(n);
res[0] = diff[0];
for (int i = 1; i < n; ++i) {
    res[i] = res[i-1] + diff[i];
}
1.1.8 排序算法（归并排序）
遍历框架：分治递归

遍历控制变量：left, right

节点操作：合并两个有序子数组（双指针）

辅助变量：临时数组 temp

cpp
void mergeSort(vector<int>& nums, int l, int r, vector<int>& temp) {
    if (l >= r) return;
    int mid = l + (r-l)/2;
    mergeSort(nums, l, mid, temp);
    mergeSort(nums, mid+1, r, temp);
    // 合并
    int i = l, j = mid+1, k = l;
    while (i <= mid && j <= r) {
        temp[k++] = nums[i] < nums[j] ? nums[i++] : nums[j++];
    }
    while (i <= mid) temp[k++] = nums[i++];
    while (j <= r) temp[k++] = nums[j++];
    for (i = l; i <= r; ++i) nums[i] = temp[i];
}
1.1.9 排序算法（快速排序）
遍历框架：分治递归

遍历控制变量：left, right

节点操作：分区（双指针交换）

辅助变量：基准值 pivot

cpp
int partition(vector<int>& nums, int l, int r) {
    int pivot = nums[l];
    int i = l, j = r;
    while (i < j) {
        while (i < j && nums[j] >= pivot) j--;
        nums[i] = nums[j];
        while (i < j && nums[i] <= pivot) i++;
        nums[j] = nums[i];
    }
    nums[i] = pivot;
    return i;
}
void quickSort(vector<int>& nums, int l, int r) {
    if (l >= r) return;
    int p = partition(nums, l, r);
    quickSort(nums, l, p-1);
    quickSort(nums, p+1, r);
}
1.2 链表（链式存储）
1.2.1 链表遍历
遍历框架：迭代

遍历控制变量：指针 cur

节点操作：访问 cur->val

辅助变量：无

cpp
ListNode* cur = head;
while (cur) {
    // 节点操作
    cur = cur->next;
}
1.2.2 反转链表
遍历框架：迭代

遍历控制变量：cur

节点操作：修改指针指向

辅助变量：prev, next

cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;
    while (cur) {
        ListNode* next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
1.2.3 链表中点（快慢指针）
遍历框架：迭代

遍历控制变量：slow, fast

节点操作：移动指针

辅助变量：无

cpp
ListNode* middleNode(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
1.2.4 环形链表检测
遍历框架：迭代

遍历控制变量：slow, fast

节点操作：移动指针并比较

辅助变量：无

cpp
bool hasCycle(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
1.3 栈
解空间结构：线性（受限），但本身常用作辅助结构。

典型算法：括号匹配、表达式求值、单调栈。

1.3.1 括号匹配
遍历框架：迭代

遍历控制变量：i

节点操作：遇到左括号入栈，右括号匹配

辅助变量：栈

cpp
bool isValid(string s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;
            char top = st.top();
            if ( (c==')' && top=='(') || (c==']' && top=='[') || (c=='}' && top=='{') ) st.pop();
            else return false;
        }
    }
    return st.empty();
}
1.3.2 单调栈（下一个更大元素）
遍历框架：迭代

遍历控制变量：i

节点操作：当前元素与栈顶比较，弹出小于当前值的元素

辅助变量：栈

cpp
vector<int> nextGreaterElement(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;
    for (int i = 0; i < n; ++i) {
        while (!st.empty() && nums[i] > nums[st.top()]) {
            res[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return res;
}
1.4 队列
常用于 BFS 或滑动窗口。

1.4.1 单调队列（滑动窗口最大值）
遍历框架：迭代

遍历控制变量：i

节点操作：维护双端队列，保证队首最大

辅助变量：deque

cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;
    vector<int> res;
    for (int i = 0; i < nums.size(); ++i) {
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
        dq.push_back(i);
        if (dq.front() == i - k) dq.pop_front();
        if (i >= k-1) res.push_back(nums[dq.front()]);
    }
    return res;
}
1.5 字符串
字符串可视为字符数组，线性结构。

1.5.1 KMP 匹配
遍历框架：迭代（主串指针 i 不回退）

遍历控制变量：i（主串索引），j（模式串索引）

节点操作：比较字符，失配时 j 回退到 next[j]

辅助变量：next 数组

cpp
vector<int> getNext(string pat) {
    int m = pat.size();
    vector<int> next(m, 0);
    int j = 0;
    for (int i = 1; i < m; ++i) {
        while (j > 0 && pat[i] != pat[j]) j = next[j-1];
        if (pat[i] == pat[j]) j++;
        next[i] = j;
    }
    return next;
}
vector<int> kmp(string txt, string pat) {
    auto next = getNext(pat);
    vector<int> res;
    int j = 0;
    for (int i = 0; i < txt.size(); ++i) {
        while (j > 0 && txt[i] != pat[j]) j = next[j-1];
        if (txt[i] == pat[j]) j++;
        if (j == pat.size()) {
            res.push_back(i - pat.size() + 1);
            j = next[j-1];
        }
    }
    return res;
}
1.5.2 Manacher 最长回文子串
遍历框架：迭代

遍历控制变量：中心 i，当前最右边界 r 和中心 c

节点操作：利用对称性初始化臂长，然后中心扩展

辅助变量：p 数组

cpp
string manacher(string s) {
    string t = "#";
    for (char c : s) { t += c; t += '#'; }
    int n = t.size();
    vector<int> p(n, 0);
    int center = 0, right = 0;
    for (int i = 1; i < n-1; ++i) {
        int mirror = 2*center - i;
        if (i < right) p[i] = min(right - i, p[mirror]);
        while (i-p[i]-1 >= 0 && i+p[i]+1 < n && t[i-p[i]-1] == t[i+p[i]+1]) p[i]++;
        if (i + p[i] > right) {
            center = i;
            right = i + p[i];
        }
    }
    // 找到最大 p[i] 对应的原串子串
    int pos = max_element(p.begin(), p.end()) - p.begin();
    int start = (pos - p[pos]) / 2;
    return s.substr(start, p[pos]);
}
