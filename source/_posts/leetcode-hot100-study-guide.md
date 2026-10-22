---
title: LeetCode Hot 100 核心母题模式与 C++ STL 秒杀手册
date: 2026-10-22 23:15:00
tags:
  - LeetCode
  - C++
  - 算法
  - 刷题指南
categories:
  - 算法与数据结构
---

> 📋 **配套任务打卡表**：👉 [LeetCode Hot 100 两日极速通关清单与打卡表](/2026/10/22/leetcode-hot100-roadmap/)  
> **定位**：面向具备算法基础、有一年左右空档期（原周赛 1700+）的开发者。  
> **目标**：不讲冗余概念，直击 **条件反射信号（题眼）**、**高频解题套路模版**、**现代 C++ 高效写法** 与 **常见翻车天坑**，助你在两日内光速唤醒题感与肌肉记忆。

---

## 目录
1. [解题模式识别矩阵（条件反射速查）](#一解题模式识别矩阵条件反射速查)
2. [现代 C++ STL 刷题核心肌肉记忆](#二现代-c-stl-刷题核心肌肉记忆)
3. [17 大专题母题套路与核心代码骨架](#三17-大专题母题套路与核心代码骨架)
   - [1. 哈希](#1-哈希)
   - [2. 双指针](#2-双指针)
   - [3. 滑动窗口](#3-滑动窗口)
   - [4. 子串与前缀和](#4-子串与前缀和)
   - [5. 普通数组操作](#5-普通数组操作)
   - [6. 矩阵操作](#6-矩阵操作)
   - [7. 链表核心模版](#7-链表核心模版)
   - [8. 二叉树与递归](#8-二叉树与递归)
   - [9. 图论（网格与拓扑）](#9-图论网格与拓扑)
   - [10. 回溯算法模版](#10-回溯算法模版)
   - [11. 二分查找全场景](#11-二分查找全场景)
   - [12. 栈与单调栈](#12-栈与单调栈)
   - [13. 堆与优先队列](#13-堆与优先队列)
   - [14. 贪心策略](#14-贪心策略)
   - [15. 线性 / 背包动态规划](#15-线性--背包动态规划)
   - [16. 多维与区间动态规划](#16-多维与区间动态规划)
   - [17. 位运算与经典技巧](#17-位运算与经典技巧)
4. [十大避坑防暴雷清单（CheckList）](#四十大避坑防暴雷清单checklist)
5. [两日极速通关执行心法](#五两日极速通关执行心法)

---

## 一、解题模式识别矩阵（条件反射速查）

看到以下题面关键字，在 3 秒内映射到对应工具与套路：

| 题目特征 / 关键字 | 第一条件反射 | 经典备选 / 优化手段 |
| :--- | :--- | :--- |
| **无序数组求和为 $K$ / 连续子数组** | 前缀和 + 哈希表（预置 `mp[0] = 1`） | 滑动窗口（仅限全正数） |
| **无序数组求最长连续序列长度** | `unordered_set`，仅当 `num - 1` 不存在时展开 | 并查集（通常较重） |
| **有序数组两数/三数之和** | 双指针（对撞指针），外层排序去重 | 哈希表 |
| **连续子数组最长/最短满足某条件** | 不定长滑动窗口（右移进窗，左移缩窗） | 二分答案 + 校验 |
| **固定长度窗口的最大/最小/频数** | 定长滑动窗口（`i >= k - 1` 时收缩左边界） | 计数数组 (26/128) |
| **滑动窗口内的最大值 / 最小值** | 单调队列 (`std::deque` 存下标) | 堆（延迟删除） / `multiset` |
| **下一个更大 / 更小元素** | 单调栈（求下一个更大用单调递减栈） | 暴力向前搜 |
| **数组元素在 $[1, n]$ 范围内找缺失/重复** | 原地哈希（`nums[nums[i]-1] = nums[i]`） | 标记符号（负数标记） |
| **链表找中点 / 环形链表入口** | 快慢指针（慢步 1，快步 2；相遇后 head 同步走） | 哈希表（空间 $O(n)$） |
| **反转局部区间链表 / K 个一组反转** | 虚拟哨兵头 + 局部头插法 / 迭代穿针引线 | 递归（容易爆栈/边界繁琐） |
| **二叉树求最大深度 / 路径和** | 自底向上树形 DP（后序遍历，返回单侧贡献） | 全局变量传参更新 |
| **二叉树层级遍历 / 最短步数转移** | 队列 BFS（按层 `sz = q.size()` 遍历） | DFS 带 `depth` 参数 |
| **网格图连通块 / 封闭岛屿** | DFS / BFS 沉岛法（将 `'1'` 涂成 `'0'`） | 并查集 |
| **依赖关系先后执行顺序 / 环检测** | 拓扑排序（入度表 + BFS 队列） | 三色标记法 DFS |
| **求所有排列 / 子集 / 组合（$N \le 15$）** | 回溯算法（选/不选，或枚举起点；排序去重） | 二进制状态压缩枚举 |
| **有序数组 / 旋转有序 / 查找边界** | 二分查找（明确维护闭区间 `[left, right]`） | `std::lower_bound` |
| **数据流实时取中位数** | 对顶堆（大根堆维护前半段，小根堆维护后半段） | 平衡二叉搜索树 |
| **Top K 大 / 小元素** | `std::nth_element` (快速选择) / 维护大小为 $K$ 的堆 | 计数桶排序 |
| **最长递增子序列 (LIS)** | 贪心 + 二分（耐心排序法：`tails` 数组二分替换） | $O(n^2)$ 动态规划 |
| **重叠区间合并 / 安排最多互不冲突区间** | 按左端点/右端点排序 + 贪心扫描维护边界 | 差分数组 |
| **出现奇数次/唯一的数字** | 异或运算 $a \oplus a = 0$ | 计数哈希 |

---

## 二、现代 C++ STL 刷题核心肌肉记忆

### 1. 泛型递归 Lambda（C++14 起支持，告别类成员或全局函数）
无需声明 `std::function`（无任何函数指针或类型擦除开销，效率等同内联函数）：
```cpp
// 模版：通过 auto& self 显式传递自身
auto dfs = [&](auto& self, int u, int p) -> void {
    for (int v : adj[u]) {
        if (v == p) continue;
        self(self, v, u);
    }
};
dfs(dfs, root, -1);
```

### 2. 优先队列（堆）的三种高效姿势
```cpp
// 1. 基础小根堆
priority_queue<int, vector<int>, greater<int>> minHeap;

// 2. 带权 Pair 堆（默认按 pair.first 升序）
using pii = pair<int, int>;
priority_queue<pii, vector<pii>, greater<pii>> pq;

// 3. 自定义 Lambda 比较器 (注意：堆中比较函数返回 true 表示后者优先级更高，即降序排列)
auto cmp = [](const auto& a, const auto& b) {
    return a.weight > b.weight; // 小根堆：weight 小的在 top
};
priority_queue<Node, vector<Node>, decltype(cmp)> pqCustom(cmp);
```

### 3. 二分利器：`std::lower_bound` 与 `std::upper_bound`
- `lower_bound(beg, end, val)`：返回指向 **第一个 $\ge val$** 元素的迭代器。
- `upper_bound(beg, end, val)`：返回指向 **第一个 $> val$** 元素的迭代器。
- 获取下标：`int idx = lower_bound(nums.begin(), nums.end(), target) - nums.begin();`
- 查找区间数量：`[lower_bound, upper_bound)` 构成了所有等于 $val$ 的元素区间。

### 4. 字符串与字符频数数组（极致常数优化）
对纯英文小写字符题目，坚决不用 `unordered_map<char, int>`，使用固定大小数组：
```cpp
// 零堆内存开销，缓存极度友好
int cnt[26]{}; 
cnt[c - 'a']++;
// 判断两频数数组是否一致可直接用 std::equal 或 cnt1 == cnt2 (若封装为 array<int, 26>)
```

### 5. 无符号数下溢天坑（Unsigned Underflow）
- `vec.size()` 返回 `size_t` (无符号整数)！
- **天坑写法**：`for (int i = 0; i <= (int)vec.size() - 2; ++i)`
  若不加 `(int)` 强制转换且 `vec` 为空，`vec.size() - 2` 会回绕为超大正整数，导致死循环或越界崩溃。

---

## 三、17 大专题母题套路与核心代码骨架

### 1. 哈希

#### 母题 128. 最长连续序列
- **题眼**：要求 $O(n)$ 时间内找到最长连续整数序列。
- **破局点**：只有当 `num - 1` 不在哈希集中时，`num` 才能作为连续序列的起点；否则跳过，避免重复向后暴力统计。
```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> numSet(nums.begin(), nums.end());
    int maxLen = 0;
    for (int num : numSet) {
        if (!numSet.count(num - 1)) { // 仅从起点开始拓展
            int curNum = num;
            int curLen = 1;
            while (numSet.count(curNum + 1)) {
                curNum++;
                curLen++;
            }
            maxLen = max(maxLen, curLen);
        }
    }
    return maxLen;
}
```

---

### 2. 双指针

#### 母题 015. 三数之和
- **题眼**：排序 + 对撞双指针 + 双重去重。
- **破局点**：固定外层 $i$ 后，双指针 $L$ 和 $R$ 对撞；关键在去重逻辑必须跨过连续相等元素。
```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    int n = nums.size();
    vector<vector<int>> ans;
    for (int i = 0; i < n - 2; ++i) {
        if (nums[i] > 0) break; // 优化：首数大于 0 则三数之和必大于 0
        if (i > 0 && nums[i] == nums[i - 1]) continue; // 第一层去重
        int L = i + 1, R = n - 1;
        while (L < R) {
            int sum = nums[i] + nums[L] + nums[R];
            if (sum < 0) L++;
            else if (sum > 0) R--;
            else {
                ans.push_back({nums[i], nums[L], nums[R]});
                while (L < R && nums[L] == nums[L + 1]) L++; // L去重
                while (L < R && nums[R] == nums[R - 1]) R--; // R去重
                L++; R--;
            }
        }
    }
    return ans;
}
```

#### 母题 042. 接雨水 (双指针法)
- **破局点**：接雨水量由 `min(max_left, max_right) - height[i]` 决定。维护左右两侧的前缀最大值，哪侧小就结算哪侧并向内收缩。
```cpp
int trap(vector<int>& height) {
    int L = 0, R = height.size() - 1;
    int leftMax = 0, rightMax = 0, ans = 0;
    while (L < R) {
        leftMax = max(leftMax, height[L]);
        rightMax = max(rightMax, height[R]);
        if (leftMax < rightMax) {
            ans += leftMax - height[L];
            L++;
        } else {
            ans += rightMax - height[R];
            R--;
        }
    }
    return ans;
}
```

---

### 3. 滑动窗口

#### 滑动窗口万能模版（不定长最长/最短子串）
```cpp
int left = 0, right = 0;
while (right < n) {
    // 1. 扩大窗口：将 nums[right] 纳入状态
    char c = s[right++];
    window[c]++;
    
    // 2. 条件不满足时，收缩窗口
    while (/* 窗口需要收缩的条件 */) {
        char d = s[left++];
        window[d]--;
        // 更新收缩状态
    }
    
    // 3. 更新答案 (依据题意求最大/最小)
    ans = max(ans, right - left);
}
```

#### 母题 003. 无重复字符的最长子串
```cpp
int lengthOfLongestSubstring(string s) {
    int lastPos[256];
    memset(lastPos, -1, sizeof(lastPos));
    int maxLen = 0, left = 0;
    for (int right = 0; right < (int)s.size(); ++right) {
        if (lastPos[(unsigned char)s[right]] >= left) {
            left = lastPos[(unsigned char)s[right]] + 1; // 直接跳跃左指针
        }
        lastPos[(unsigned char)s[right]] = right;
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

### 4. 子串与前缀和

#### 母题 560. 和为 K 的子数组
- **题眼**：包含负数数组中找和为 $K$ 的连续子数组数目。不可用双指针/滑动窗口！
- **破局点**：$sum[i] - sum[j] = K \implies sum[j] = sum[i] - K$。
- **防暴雷点**：必须初始化 `prefixFreq[0] = 1`（处理前缀本身就等于 $K$ 的情况）。
```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefixFreq;
    prefixFreq[0] = 1;
    int curSum = 0, count = 0;
    for (int x : nums) {
        curSum += x;
        if (prefixFreq.count(curSum - k)) {
            count += prefixFreq[curSum - k];
        }
        prefixFreq[curSum]++;
    }
    return count;
}
```

#### 母题 239. 滑动窗口最大值（单调队列）
- **破局点**：单调双端队列 `deque` 存储下标，维护内部元素数值严格单调递减。
```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq; // 存下标
    vector<int> ans;
    for (int i = 0; i < (int)nums.size(); ++i) {
        // 1. 弹出队首过期的下标
        if (!dq.empty() && dq.front() <= i - k) {
            dq.pop_front();
        }
        // 2. 维护单调递减性质：凡是小于当前元素的队尾元素全弹出
        while (!dq.empty() && nums[dq.back()] <= nums[i]) {
            dq.pop_back();
        }
        dq.push_back(i);
        // 3. 达到窗口大小 k 时记录结果
        if (i >= k - 1) {
            ans.push_back(nums[dq.front()]);
        }
    }
    return ans;
}
```

---

### 5. 普通数组操作

#### 母题 041. 缺失的第一个正数（原地哈希）
- **破局点**：长度为 $N$ 的数组，缺失的最小正整数必在 $[1, N + 1]$ 之间。原地将值 $x \in [1, N]$ 置换到下标 $x - 1$ 的位置。
```cpp
int firstMissingPositive(vector<int>& nums) {
    int n = nums.size();
    for (int i = 0; i < n; ++i) {
        // 当 nums[i] 处于有效范围 [1, n]，且没有归位时，不断交换归位
        while (nums[i] >= 1 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            swap(nums[i], nums[nums[i] - 1]);
        }
    }
    for (int i = 0; i < n; ++i) {
        if (nums[i] != i + 1) return i + 1;
    }
    return n + 1;
}
```

#### 母题 189. 轮转数组
- **破局点**：三次翻转法。$k = k \pmod n$。整体翻转 $\to$ 翻转前 $k$ 个 $\to$ 翻转后 $n - k$ 个。
```cpp
void rotate(vector<int>& nums, int k) {
    k %= nums.size();
    reverse(nums.begin(), nums.end());
    reverse(nums.begin(), nums.begin() + k);
    reverse(nums.begin() + k, nums.end());
}
```

---

### 6. 矩阵操作

#### 母题 054. 螺旋矩阵（四界收缩）
- **破局点**：定义 `top, bottom, left, right` 四个边界，顺时针打印一圈后收缩对应边界，并立即检查是否相交。
```cpp
vector<int> spiralOrder(vector<vector<int>>& matrix) {
    vector<int> ans;
    int top = 0, bottom = matrix.size() - 1;
    int left = 0, right = matrix[0].size() - 1;
    while (true) {
        for (int j = left; j <= right; ++j) ans.push_back(matrix[top][j]);
        if (++top > bottom) break;
        for (int i = top; i <= bottom; ++i) ans.push_back(matrix[i][right]);
        if (--right < left) break;
        for (int j = right; j >= left; --j) ans.push_back(matrix[bottom][j]);
        if (--bottom < top) break;
        for (int i = bottom; i >= top; --i) ans.push_back(matrix[i][left]);
        if (++left > right) break;
    }
    return ans;
}
```

---

### 7. 链表核心模版

#### 核心利器：虚拟头节点（Dummy Head）
只要链表头节点可能被修改/删除，第一件事先建立哨兵头：
```cpp
ListNode dummy(0, head);
ListNode* prev = &dummy;
```

#### 母题 206. 反转链表
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr) {
        ListNode* nxt = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }
    return prev;
}
```

#### 母题 025. K 个一组翻转链表
- **破局点**：先数够 $K$ 个，不够则不翻转直接返回；够 $K$ 个则局部翻转，前接后挂。
```cpp
ListNode* reverseKGroup(ListNode* head, int k) {
    ListNode dummy(0, head);
    ListNode* prev = &dummy;
    ListNode* end = &dummy;

    while (end->next) {
        for (int i = 0; i < k && end; ++i) end = end->next;
        if (!end) break;
        ListNode* start = prev->next;
        ListNode* nextGroup = end->next;
        end->next = nullptr; // 断开局部链表

        prev->next = reverseList(start); // 翻转这 k 个节点
        start->next = nextGroup; // 翻转后 start 变尾节点，缝合后半段
        prev = start;
        end = start;
    }
    return dummy.next;
}
```

#### 母题 142. 环形链表 II（快慢指针推导）
- **原理**：快指针走 $2s$，慢指针走 $s$，相遇时 $2s - s = nb \implies s = nb$。
- 入环点公式：头节点到入环点为 $a$ 步，则 $a + nb$ 处也是入环点。相遇后，慢指针从头开始，快指针原地出发，每次 1 步，再次相遇即为入环点。

---

### 8. 二叉树与递归

二叉树的核心心法：**思考当前节点（root）该做什么，何时做（前序/中序/后序），需要子树返回什么信息。**

#### 母题 236. 二叉树的最近公共祖先 (LCA)
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* left = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left && right) return root; // p 和 q 分布在左右子树，当前就是 LCA
    return left ? left : right;     // 都在一侧，返回非空侧
}
```

#### 母题 124. 二叉树中的最大路径和（自底向上贡献模型）
```cpp
int maxPathSum(TreeNode* root) {
    int maxSum = INT_MIN;
    auto dfs = [&](auto& self, TreeNode* node) -> int {
        if (!node) return 0;
        int leftGain = max(0, self(self, node->left));   // 若贡献为负则舍弃
        int rightGain = max(0, self(self, node->right));
        
        // 拐弯穿过当前节点的最大路径，更新全局最优
        maxSum = max(maxSum, node->val + leftGain + rightGain);
        
        // 向父节点只能贡献单侧分支
        return node->val + max(leftGain, rightGain);
    };
    dfs(dfs, root);
    return maxSum;
}
```

---

### 9. 图论（网格与拓扑）

#### 母题 200. 岛屿数量（沉岛 DFS）
```cpp
int numIslands(vector<vector<char>>& grid) {
    int m = grid.size(), n = grid[0].size();
    auto dfs = [&](auto& self, int r, int c) -> void {
        if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != '1') return;
        grid[r][c] = '0'; // 沉岛标记
        self(self, r + 1, c);
        self(self, r - 1, c);
        self(self, r, c + 1);
        self(self, r, c - 1);
    };

    int count = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (grid[i][j] == '1') {
                count++;
                dfs(dfs, i, j);
            }
        }
    }
    return count;
}
```

#### 母题 207. 课程表（Kahn 算法拓扑排序）
- **破局点**：计算入度 `inDegree`，入度为 0 的节点入队 BFS，出队时消减邻接点入度。若出队总数等于总课程数，则无环。
```cpp
bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(numCourses);
    vector<int> inDegree(numCourses, 0);
    for (const auto& pre : prerequisites) {
        adj[pre[1]].push_back(pre[0]);
        inDegree[pre[0]]++;
    }
    queue<int> q;
    for (int i = 0; i < numCourses; ++i) {
        if (inDegree[i] == 0) q.push(i);
    }
    int visited = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        visited++;
        for (int v : adj[u]) {
            if (--inDegree[v] == 0) q.push(v);
        }
    }
    return visited == numCourses;
}
```

---

### 10. 回溯算法模版

回溯两大母体形态：
1. **子集/组合型**：传递 `startIndex`，去重时先排序，`if (i > startIndex && nums[i] == nums[i - 1]) continue;`
2. **排列型**：传递 `used` 数组，去重时 `if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;`

#### 子集与组合总和去重模版（以 039/040 为例）
```cpp
void backtrack(const vector<int>& nums, int start, int target, vector<int>& path, vector<vector<int>>& ans) {
    if (target == 0) {
        ans.push_back(path);
        return;
    }
    for (int i = start; i < (int)nums.size(); ++i) {
        if (nums[i] > target) break; // 剪枝
        if (i > start && nums[i] == nums[i - 1]) continue; // 同层去重！
        path.push_back(nums[i]);
        backtrack(nums, i + 1, target - nums[i], path, ans); // 若元素可复用则传 i
        path.pop_back(); // 撤销选择
    }
}
```

---

### 11. 二分查找全场景

#### 黄金法则：闭区间 `[left, right]` 模版（绝不死循环）
```cpp
int left = 0, right = (int)nums.size() - 1;
int ans = -1;
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (check(mid)) {
        ans = mid;        // 记录可行解
        right = mid - 1;  // 继续往左逼近更小解 (找左边界)
    } else {
        left = mid + 1;
    }
}
```

#### 母题 033. 搜索旋转排序数组
- **破局点**：旋转数组折半后，**必有一半是严格单调有序的**！
```cpp
int search(vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        
        // 判断左半部分是否有序
        if (nums[left] <= nums[mid]) {
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1; // target 在有序左半段
            } else {
                left = mid + 1;
            }
        } else { // 右半部分有序
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1;  // target 在有序右半段
            } else {
                right = mid - 1;
            }
        }
    }
    return -1;
}
```

---

### 12. 栈与单调栈

#### 单调栈适用场景：寻找数组中每个元素“下一个更大”或“下一个更小”的元素。
- 找下一个更大 $\to$ 单调递减栈（栈底大，栈顶小，遇到比栈顶大的立即弹出结算）。
- 找下一个更小 $\to$ 单调递增栈。

#### 母题 084. 柱状图中最大的矩形（首尾加哨兵）
- **破局点**：首尾插入高度为 0 的哨兵，既能让最左元素有左边界，又能强制最后清空栈内全部残留柱子。
```cpp
int largestRectangleArea(vector<int>& heights) {
    // 左右各加一个高度为 0 的哨兵
    heights.insert(heights.begin(), 0);
    heights.push_back(0);
    stack<int> st; // 存下标，单调递增
    int maxArea = 0;
    
    for (int i = 0; i < (int)heights.size(); ++i) {
        while (!st.empty() && heights[i] < heights[st.top()]) {
            int h = heights[st.top()];
            st.pop();
            int w = i - st.top() - 1; // 此时栈顶为左边第一个更小元素的下标
            maxArea = max(maxArea, h * w);
        }
        st.push(i);
    }
    return maxArea;
}
```

---

### 13. 堆与优先队列

#### 母题 215. 数组中的第 K 个最大元素
- **解法 1（工程最快）**：C++ 内置快速选择算法 `std::nth_element`。平均时间复杂度 $O(n)$。
```cpp
int findKthLargest(vector<int>& nums, int k) {
    // nth_element 会使目标位置左边都不大于它，右边都不小于它
    nth_element(nums.begin(), nums.begin() + k - 1, nums.end(), greater<int>());
    return nums[k - 1];
}
```
- **解法 2（堆）**：维护大小为 $K$ 的小根堆，新元素大于堆顶时入堆换顶。

#### 母题 295. 数据流的中位数（对顶堆）
- **结构**：大根堆 `left`（存较小的一半），小根堆 `right`（存较大的一半）。
- **维持平衡**：`left.size() == right.size()` 或 `left.size() == right.size() + 1`。

---

### 14. 贪心策略

#### 母题 055 / 045. 跳跃游戏
- **破局点**：维护当前能到达的全局最远边界 `maxReach`。
```cpp
// 055. 跳跃游戏 I
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < (int)nums.size(); ++i) {
        if (i > maxReach) return false;
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}
```

---

### 15. 线性 / 背包动态规划

#### 母题 300. 最长递增子序列 (LIS)
- **破局点**：$O(n \log n)$ 贪心 + 二分（耐心排序法）。
- 维护一个数组 `tails`，`tails[i]` 表示长度为 $i+1$ 的递增子序列末尾元素的最小值。
```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) {
            tails.push_back(x);
        } else {
            *it = x; // 贪心替换为更小末尾
        }
    }
    return tails.size();
}
```

#### 母题 322. 零钱兑换（完全背包求最小硬币数）
```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0;
    for (int coin : coins) {
        for (int i = coin; i <= amount; ++i) {
            dp[i] = min(dp[i], dp[i - coin] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

---

### 16. 多维与区间动态规划

#### 母题 1143. 最长公共子序列 (LCS)
- 状态：$dp[i][j]$ 表示 $s1[0..i-1]$ 与 $s2[0..j-1]$ 的 LCS 长度。
- 转移方程：
  - 若 $s1[i-1] == s2[j-1]$：$dp[i][j] = dp[i-1][j-1] + 1$
  - 否则：$dp[i][j] = \max(dp[i-1][j], dp[i][j-1])$

#### 母题 072. 编辑距离
- 状态：$dp[i][j]$ 将 $word1[0..i-1]$ 转为 $word2[0..j-1]$ 的最少操作数。
- 转移方程：
  - 若相等：$dp[i][j] = dp[i-1][j-1]$
  - 若不相等：$1 + \min(\{dp[i-1][j] \text{ (删除)}, dp[i][j-1] \text{ (插入)}, dp[i-1][j-1] \text{ (替换)}\})$

---

### 17. 位运算与经典技巧

#### 经典位运算技巧速查
- 清除最低位 1：`n = n & (n - 1)`（统计二进制 1 的个数常用）
- 提取最低位 1：`lowbit = n & (-n)`
- 判断 2 的幂：`n > 0 && (n & (n - 1)) == 0`

#### 母题 169. 多数元素（Boyer-Moore 摩尔投票法）
- **破局点**：票数抵消。相同加 1，不同减 1，为 0 时重置候选人。时间 $O(n)$，空间 $O(1)$。
```cpp
int majorityElement(vector<int>& nums) {
    int candidate = 0, count = 0;
    for (int num : nums) {
        if (count == 0) candidate = num;
        count += (num == candidate) ? 1 : -1;
    }
    return candidate;
}
```

#### 母题 075. 颜色分类（荷兰国旗三路快排）
```cpp
void sortColors(vector<int>& nums) {
    int p0 = 0, curr = 0, p2 = (int)nums.size() - 1;
    while (curr <= p2) {
        if (nums[curr] == 0) {
            swap(nums[curr++], nums[p0++]);
        } else if (nums[curr] == 2) {
            swap(nums[curr], nums[p2--]); // 注意：curr 不自增，交换回来的数还需判断
        } else {
            curr++;
        }
    }
}
```

---

## 四、十大避坑防暴雷清单（CheckList）

写完代码提交前，花 5 秒钟肉眼扫描以下 10 点：

1. [ ] **整数越界溢出**：
   - 二分中点：使用 `mid = left + (right - left) / 2`，绝不用 `(left + right) / 2`。
   - 累乘、平方和、多步相加是否会爆 `INT_MAX`，必要时开 `long long`。
2. [ ] **无符号大小计算下溢**：
   - 检查是否有 `for (int i = 0; i < (int)vec.size() - 1; ...)`。未加 `(int)` 时，如果 `vec` 为空，`0 - 1` 将变成超大正数。
3. [ ] **前缀和哈希表漏置 0**：
   - `subarraySum` 等前缀和问题，必须初始化 `mp[0] = 1`，否则会漏掉从下标 0 开始的合法子数组。
4. [ ] **空指针防护**：
   - 链表和二叉树访问 `node->val` 或 `node->next` 前，是否已先判断 `node != nullptr`。
5. [ ] **优先队列比较器方向混淆**：
   - C++ `std::greater<T>` 在堆中产生的是**小根堆**；返回 `true` 的节点会被排到堆底。
6. [ ] **回溯漏撤销现场**：
   - `path.push_back()` 是否在递归返回后严格对应 `path.pop_back()`；`vis[i] = true` 后面是否有 `vis[i] = false`。
7. [ ] **滑动窗口左右边界边界条件**：
   - 保证窗口缩小循环 `while (window invalid)` 中有 `left <= right` 保护或逻辑上必然终止。
8. [ ] **二维矩阵行列大小混淆**：
   - 行数 `int m = matrix.size();`，列数 `int n = matrix[0].size();`。在判断 `matrix.empty()` 之前严禁直接取 `matrix[0]`。
9. [ ] **图论与多源 BFS 初始化**：
   - 多源 BFS 入队时，所有起点必须在入队瞬间立即标记 `vis = true` 或记录 step，不能等出队时才标记，否则会导致重复入队超时（TLE）。
10. [ ] **局部变量 vs 全局变量残留**：
    - LeetCode 会连续调用你的类实例执行多组用例，**绝对不要使用全局/静态变量**；若写在类私有成员中，每次进入入口函数先执行 `clear()`。

---

## 五、两日极速通关执行心法

```text
[打开题目] ──> [读题 1 分钟，提炼核心关键字]
       │
       ▼
[模式矩阵能否在 5 分钟内敲定思路?]
  ├─ [能闭环] ──> [现代 C++ 快速码出 (5 分钟)] ──> [扫一眼 10 大 CheckList] ──> [提交 AC] ──> [在打卡清单打勾]
  │
  └─ [超过 8 分钟卡壳] ──> [立即看题解 (灵神/精选题解)] ──> [提炼核心套路与 STL 肌肉技巧] ──> [关闭题解，独立手写 AC] ──> [在打卡清单打勾]
```

### 刷题心法总结：
- **不恋战**：你已有 1700+ 的底子，卡题不是智商不够，而是特定套路的直觉暂时生锈。超过 8 分钟直接看题解，不要内耗。
- **抄题解不叫抄，叫重新校准神经元**：看完题解后，**立刻关掉页面闭卷手写一遍**，这才是真正的记忆巩固。
- **抓住母题**：Hot 100 中很多是同源题（例如 15 题三数之和与 11 题盛水容器、206 反转链表与 25 题 K 个一组反转），母题吃透，变种题目一秒看破。
