# CS101 Cheatsheet（紧凑整理版）

声明：此cheetsheet全部内容均由chatgpt生成，我本人只提供prompt并删改

> 本文档面向 **算法 / 数据结构 / Python 竞赛与面试**，目标是： **一眼定位问题类型 → 套模板 → 不踩复杂度坑**。

---

## 目录

1. Python 核心数据结构复杂度速查
2. 常见算法思想总览
3. 区间问题模板
4. 背包问题全集
5. 单调栈（必背）
6. 最长上升子序列（LIS）
7. DFS 搜索与回溯
8. BFS 广度优先搜索
9. 滑动窗口（双指针）
10. Python 常用库速查与性能注意事项
11. 算法题型速查对照表

---

## 一、Python 核心数据结构复杂度速查

### 1. list（动态数组）

- **优势**：随机访问快
- **劣势**：中间插删慢

| 操作              | 时间复杂度      | 说明   |
| --------------- | ---------- | ---- |
| a[i]            | O(1)       | 随机访问 |
| append / pop()  | O(1)       | 尾部操作 |
| insert / pop(i) | O(n)       | 整体移动 |
| x in a          | O(n)       | 线性查找 |
| sort()          | O(n log n) | 稳定排序 |
| 切片              | O(k)       | 会拷贝  |

⚠️ 坑：**不要用 list 当队列**

---

### 2. tuple（不可变数组）

- 更省空间
- 可作为 dict / set 的 key

| 操作     | 时间   |
| ------ | ---- |
| t[i]   | O(1) |
| x in t | O(n) |

---

### 3. deque（双端队列）

| 操作                   | 时间   | 说明 |
| -------------------- | ---- | -- |
| append / pop         | O(1) |    |
| appendleft / popleft | O(1) |    |
| 随机访问                 | O(n) | ❌  |

适用：**BFS、滑动窗口**

---

### 4. dict / set（哈希）

| 操作      | 时间     | 说明  |
| ------- | ------ | --- |
| 查找 / 插入 | O(1)   | 均摊  |
| 遍历      | O(n)   |     |
| 合并      | O(n+m) | 新对象 |

⚠️ key 必须可哈希

---

## 二、常见算法思想总览

| 思想  | 关键词     | 常见题目    |
| --- | ------- | ------- |
| 贪心  | 最少 / 最多 | 区间调度    |
| 二分  | 最小最大    | 答案二分    |
| 前缀和 | 多次区间    | 区间和     |
| 差分  | 区间修改    | 区间加     |
| DFS | 所有方案    | 排列 / 路径 |
| BFS | 最短步数    | 迷宫      |
| DP  | 最优子结构   | 背包      |
| 单调栈 | 左右最近    | 矩形面积    |

---

## 三、区间问题模板

### 1️⃣ 区间覆盖（最少区间）

```python
# 用最少的区间覆盖目标区间 [L, R]
def min_cover(intervals, L, R):
    # 按区间左端点排序
    intervals.sort()
    cur = L        # 当前已覆盖到的位置
    i = 0          # 区间指针
    ans = 0        # 使用的区间数量

    # 只要还没覆盖到 R 就继续
    while cur < R:
        far = cur  # 在当前可选区间中能延伸到的最远右端
        # 所有左端点 <= cur 的区间都是“可选区间"
        while i < len(intervals) and intervals[i][0] <= cur:
            far = max(far, intervals[i][1])
            i += 1
        # 如果一步都无法向右推进，说明无解
        if far == cur:
            return -1
        # 选择能延伸最远的区间
        cur = far
        ans += 1
    return ans
```

---

### 2️⃣ 区间合并

```python
# 合并所有有重叠的区间
def merge_intervals(intervals):
    # 先按左端点排序
    intervals.sort()
    res = []
    for l, r in intervals:
        # 如果当前区间与结果中最后一个区间不重叠
        if not res or l > res[-1][1]:
            res.append([l, r])
        else:
            # 否则合并区间，更新右端点
            res[-1][1] = max(res[-1][1], r)
    return res
```

---

## 四、背包问题全集（必考）

### 1️⃣ 0-1 背包

```python
# 0-1 背包：每个物品只能选一次
def knapsack_01(weights, values, W):
    # dp[c] 表示容量为 c 时的最大价值
    dp = [0] * (W + 1)

    for w, v in zip(weights, values):
        # 容量必须倒序遍历，防止同一物品被重复使用
        for cap in range(W, w - 1, -1):
            # 选 or 不选 当前物品
            dp[cap] = max(dp[cap], dp[cap - w] + v)
    return dp[W]
```

口诀：**0-1 → 容量倒序**

---

### 2️⃣ 完全背包

```python
# 完全背包：每个物品可以无限次使用
def knapsack_complete(weights, values, W):
    dp = [0] * (W + 1)

    for w, v in zip(weights, values):
        # 容量正序遍历，允许重复选当前物品
        for cap in range(w, W + 1):
            dp[cap] = max(dp[cap], dp[cap - w] + v)
    return dp[W]
```

口诀：**完全 → 容量正序**

---

### 3️⃣ 多重背包（二进制优化）

**问题描述**： 每种物品有数量限制 `cnt[i]`，重量 `w[i]`，价值 `v[i]`。

**核心思想**： 把 `cnt` 拆成若干个 `1, 2, 4, ...` 的幂次之和， 转化为若干个 **0-1 背包物品**。

```python
# 多重背包：二进制拆分优化
def knapsack_multiple(weights, values, counts, W):
    dp = [0] * (W + 1)

    for w, v, c in zip(weights, values, counts):
        k = 1
        while c > 0:
            num = min(k, c)
            ww = w * num
            vv = v * num
            # 当成 0-1 背包物品处理
            for cap in range(W, ww - 1, -1):
                dp[cap] = max(dp[cap], dp[cap - ww] + vv)
            c -= num
            k <<= 1
    return dp[W]
```

👉 要点：

- 数量 `c` 拆成 **log c 个物品**
- 多重背包 → 0-1 背包

口诀：**“多重拆二进制，再做 0-1”**

---

## 五、单调栈（秒杀区间类）

### 经典用途

- 下一个更大 / 更小元素
- 最大矩形面积
- 子数组最值贡献

```python
# 对每个元素，找到右侧第一个比它大的元素
def next_greater(a):
    n = len(a)
    res = [-1] * n     # 默认不存在更大元素
    st = []            # 单调递减栈，存下标

    for i, x in enumerate(a):
        # 当前元素比栈顶大，说明它是栈顶元素的 next greater
        while st and a[st[-1]] < x:
            res[st.pop()] = x
        st.append(i)
    return res
```

---

### 典型例题：盛水容器（接雨水）

**问题描述**： 给定柱状图高度，计算可以接多少雨水（LeetCode 42）。

```python
# 单调栈解法：计算接雨水总量
def trap(height):
    st = []        # 存下标，保持递减
    water = 0

    for i, h in enumerate(height):
        # 当前柱子更高，可以形成凹槽
        while st and height[st[-1]] < h:
            mid = st.pop()     # 凹槽底部
            if not st:
                break
            left = st[-1]     # 左边界
            # 宽度 * 高度
            w = i - left - 1
            hgt = min(height[left], h) - height[mid]
            water += w * hgt
        st.append(i)
    return water
```

👉 要点：

- 栈维护 **递减高度**
- 每次弹出计算一个“凹槽”

口诀：**“弹中算水，左右定高”**

---

## 六、最长上升子序列（LIS）

```python
from bisect import bisect_left

# O(n log n) 求最长上升子序列长度
def length_of_LIS(nums):
    tails = []
    # tails[k] = 长度为 k+1 的上升子序列的最小结尾
    for x in nums:
        # 找到第一个 >= x 的位置
        i = bisect_left(tails, x)
        if i == len(tails):
            # x 比所有结尾都大，可以直接接在后面
            tails.append(x)
        else:
            # 用更小的结尾替换，方便后续延伸
            tails[i] = x
    return len(tails)
```

一句话：**维护结尾最小值**

---

## 七、DFS 搜索与回溯

### 模板

```python
# DFS 通用模板（回溯思想）
def dfs(state):
    # 终止条件：当前状态已经是一个解
    if 终止条件:
        return

    # 枚举当前状态下的所有选择
    for choice in choices:
        # 1. 做选择（修改状态）
        做选择
        # 2. 递归进入下一层
        dfs(new_state)
        # 3. 撤销选择（回溯）
        撤销选择
```

适用：全排列、子集、路径、连通块

---

### 典型例题 1️⃣：马走日（Knight Tour 计数 / 可达性）

**问题描述**： 棋盘上马从起点出发，每一步走“日”字，问是否能走到终点 / 一共多少种走法（不重复格子）。

```python
# 判断马是否能从 (sx, sy) 走到 (tx, ty)
# n x m 棋盘，不允许重复访问格子

dirs = [(2,1),(2,-1),(-2,1),(-2,-1),(1,2),(1,-2),(-1,2),(-1,-2)]

visited = [[False]*m for _ in range(n)]
found = False

def dfs(x, y):
    global found
    if found:
        return
    # 到达目标
    if (x, y) == (tx, ty):
        found = True
        return
    for dx, dy in dirs:
        nx, ny = x + dx, y + dy
        # 边界 + 未访问
        if 0 <= nx < n and 0 <= ny < m and not visited[nx][ny]:
            visited[nx][ny] = True
            dfs(nx, ny)
            visited[nx][ny] = False
```

👉 要点：**方向数组 + visited + 回溯**

---

### 典型例题 2️⃣：八皇后（N 皇后）

**问题描述**： 在 n×n 棋盘上放 n 个皇后，使任意两个皇后不同行、不同列、不同对角线。

```python
# 返回 N 皇后的解的数量

def solveNQueens(n):
    col = [False] * n          # 列是否被占
    diag1 = [False] * (2*n)    # 主对角线 (row + col)
    diag2 = [False] * (2*n)    # 副对角线 (row - col + n)
    ans = 0

    def dfs(r):
        nonlocal ans
        # 成功放完 n 行
        if r == n:
            ans += 1
            return
        for c in range(n):
            if not col[c] and not diag1[r+c] and not diag2[r-c+n]:
                col[c] = diag1[r+c] = diag2[r-c+n] = True
                dfs(r + 1)
                col[c] = diag1[r+c] = diag2[r-c+n] = False

    dfs(0)
    return ans
```

👉 要点：

- **一行一行放**（天然避免同行冲突）
- 列 / 对角线用布尔数组 O(1) 判断

口诀：**“行递归，列枚举，三数组判冲突”**

---

### 八皇后补充做法：按列回溯（等价写法）

```python
# 按“列”递归放皇后（与按行完全对称）
def solveNQueens_col(n):
    row = [False] * n
    diag1 = [False] * (2*n)    # r + c
    diag2 = [False] * (2*n)    # r - c + n
    ans = 0

    def dfs(c):
        nonlocal ans
        if c == n:
            ans += 1
            return
        for r in range(n):
            if not row[r] and not diag1[r+c] and not diag2[r-c+n]:
                row[r] = diag1[r+c] = diag2[r-c+n] = True
                dfs(c + 1)
                row[r] = diag1[r+c] = diag2[r-c+n] = False

    dfs(0)
    return ans
```

👉 理解要点：

- **行回溯 / 列回溯完全对称**
- 选“行 or 列”只是状态定义不同

口诀：**“定一维，枚举另一维”**

---

### 典型例题 3️⃣：全排列（含去重）

**问题描述**： 给定一个数组，输出所有可能的排列（元素可能重复）。

```python
# 返回 nums 的所有全排列（去重）
def permute_unique(nums):
    nums.sort()              # 排序，方便去重
    used = [False] * len(nums)
    res = []
    path = []

    def dfs():
        # 当前排列长度等于 n，记录答案
        if len(path) == len(nums):
            res.append(path[:])
            return
        for i in range(len(nums)):
            # 已使用过的元素不能再用
            if used[i]:
                continue
            # 去重：相同数字，必须按顺序使用
            if i > 0 and nums[i] == nums[i-1] and not used[i-1]:
                continue
            used[i] = True
            path.append(nums[i])
            dfs()
            path.pop()
            used[i] = False

    dfs()
    return res
```

👉 要点：

- `used` 控制是否选过
- **排序 + 相邻剪枝** 实现去重

口诀：**“排好序，用数组，前没用就跳过”**

---

## 八、BFS 广度优先搜索

```python
from collections import deque

# BFS 模板：求最短步数（无权图）
def bfs(start):
    q = deque([start])
    visited = {start}   # 防止重复访问
    step = 0            # 当前层数 / 步数

    while q:
        for _ in range(len(q)):
            cur = q.popleft()
            if cur == target:
                return step
            for nxt in neighbors(cur):
                if nxt not in visited:
                    visited.add(nxt)
                    q.append(nxt)
        step += 1
```

适用：**最短步数 / 无权图**

---

### 典型例题：迷宫最短路（BFS）

```python
from collections import deque

# grid: 0 可走, 1 障碍
def shortest_path(grid, sx, sy, tx, ty):
    n, m = len(grid), len(grid[0])
    q = deque([(sx, sy)])
    dist = [[-1]*m for _ in range(n)]
    dist[sx][sy] = 0
    dirs = [(1,0),(-1,0),(0,1),(0,-1)]

    while q:
        x, y = q.popleft()
        if (x, y) == (tx, ty):
            return dist[x][y]
        for dx, dy in dirs:
            nx, ny = x + dx, y + dy
            if 0 <= nx < n and 0 <= ny < m \
               and grid[nx][ny] == 0 and dist[nx][ny] == -1:
                dist[nx][ny] = dist[x][y] + 1
                q.append((nx, ny))
    return -1
```

口诀：**“BFS 一层一层走，第一次到就是最短”**

---

## 九、滑动窗口（双指针）

### 适用场景

- 连续子数组 / 子串
- 最短 / 最长区间
- 窗口内满足某种条件

### 通用模板

```python
l = 0
window = {}

for r in range(len(s)):
    window[s[r]] = window.get(s[r], 0) + 1
    while 窗口不合法:
        window[s[l]] -= 1
        if window[s[l]] == 0:
            del window[s[l]]
        l += 1
    # 此时窗口合法
```

口诀：**“右扩找解，左缩保法”**

---

### 典型例题：无重复字符的最长子串

```python
# 滑动窗口：最长无重复子串
def lengthOfLongestSubstring(s):
    cnt = {}
    l = 0
    ans = 0

    for r, ch in enumerate(s):
        cnt[ch] = cnt.get(ch, 0) + 1
        while cnt[ch] > 1:
            cnt[s[l]] -= 1
            l += 1
        ans = max(ans, r - l + 1)
    return ans
```

口诀：**“重复就缩，合法就算”**

---

## 九、Python 常用库速查（heapq / bisect / defaultdict / cmp\_to\_key / 拷贝）

### 0️⃣ cmp\_to\_key（自定义排序规则）

当内置 `key=` 不够用时，使用比较函数。

```python
from functools import cmp_to_key

# 返回负数：a < b，返回 0：相等，返回正数：a > b
def cmp(a, b):
    if a[0] != b[0]:
        return a[0] - b[0]        # 第一关键字升序
    return b[1] - a[1]            # 第二关键字降序

arr.sort(key=cmp_to_key(cmp))
```

⚠️ 注意：

- `cmp` **只返回大小关系**，不返回 True / False
- 能用 `key` 就别用 `cmp`（`cmp` 更慢）

典型场景：

- 区间题（左端点升序，右端点降序）
- 多关键字、非独立排序规则

口诀：**“key 不行，再上 cmp\_to\_key”**

---

### 1️⃣ heapq（优先队列 / 小根堆）

### 1️⃣ heapq（优先队列 / 小根堆）

**特点**：

- 只支持小根堆
- 维护“当前最小 / 最大的 k 个元素”

```python
import heapq

h = []
heapq.heappush(h, 3)
heapq.heappush(h, 1)
heapq.heappush(h, 5)

x = heapq.heappop(h)   # 弹出最小值 1
```

常见技巧：

```python
# 大根堆：取相反数
heapq.heappush(h, -x)

# 取最小的 k 个元素
heapq.nsmallest(k, nums)

# 取最大的 k 个元素
heapq.nlargest(k, nums)
```

典型应用：

- Top K 问题
- 合并 k 个有序数组
- Dijkstra（最短路）

口诀：**“heapq 默认小根堆，大根取反数”**

---

### 2️⃣ bisect（二分插入 / 有序数组）

```python
from bisect import bisect_left, bisect_right

a = [1, 3, 3, 5]

bisect_left(a, 3)   # 1（第一个 >= 3）
bisect_right(a, 3)  # 3（第一个 > 3）
```

典型用法：

```python
# 保持数组有序插入
from bisect import insort
insort(a, 4)
```

应用场景：

- LIS（最长上升子序列）
- 有序数组中查区间 / 计数
- 动态维护有序序列

口诀：**“left 找 >=，right 找 >”**

---

### 3️⃣ defaultdict（自动初始化字典）

```python
from collections import defaultdict

d = defaultdict(int)      # 默认 0
d2 = defaultdict(list)    # 默认空列表
```

示例：统计频率

```python
for x in nums:
    d[x] += 1
```

示例：建图

```python
g = defaultdict(list)
for u, v in edges:
    g[u].append(v)
```

对比普通 dict：

```python
# 普通 dict 需要判空
if x not in d:
    d[x] = 0
```

典型应用：

- 计数
- 邻接表
- 分组 / 哈希映射

口诀：**“defaultdict = 不用判空的 dict”**

---

## 十、Python 性能坑与技巧补充

### 1️⃣ 深拷贝 vs 浅拷贝（必踩坑）

```python
import copy

a = [[1, 2], [3, 4]]
b = a[:]              # 浅拷贝（只拷贝最外层）
c = copy.deepcopy(a)  # 深拷贝（完全独立）
```

⚠️ 区别：

- **浅拷贝**：内部元素仍然共享
- **深拷贝**：完全不共享

```python
a[0][0] = 99
print(b)  # 会变
print(c)  # 不变
```

常见坑位：

- `path[:]` 是浅拷贝（但对一维列表通常够用）
- DFS 中保存二维状态 → 必须深拷贝

口诀：**“一维切片够用，多维必须 deepcopy”**

---

### 1️⃣ 不定行输入（读到 EOF）

```python
import sys
for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    # 处理每一行
```

或一次性读取：

```python
import sys
data = sys.stdin.read().split()
```

适用：OJ 输入行数不固定 / 文件输入

---

### 2️⃣ 保留小数 / 浮点输出

```python
x = 3.1415926
print(f"{x:.2f}")   # 3.14
print("%.3f" % x)  # 3.142
```

⚠️ 比较浮点数时用误差：

```python
abs(a - b) < 1e-9
```

---

### 3️⃣ 格式化字符串（强烈推荐 f-string）

```python
name = "Alice"
age = 18
print(f"{name} is {age} years old")
```

常用格式：

```python
f"{x:>5}"    # 右对齐
f"{x:<5}"    # 左对齐
f"{x:^5}"    # 居中
f"{x:06d}"   # 补零
```

---

### 4️⃣ 快速输入（竞赛必备）

```python
import sys
input = sys.stdin.readline
```

---

### 5️⃣ 常用小技巧

```python
# 交换变量
a, b = b, a

# 枚举下标和值
for i, x in enumerate(arr):
    pass

# 解包
x, y = point
```

❌ 循环中字符串拼接

```python
s += 'a'  # 慢
```

✅ 正确写法

```python
''.join(chars)
```

---

## 十一、算法题型速查表

| 题目特征   | 首选方法    |
| ------ | ------- |
| 最短路径   | BFS     |
| 所有方案   | DFS     |
| 区间最优   | DP / 贪心 |
| 左右最近   | 单调栈     |
| 多次区间查询 | 前缀和     |
| 最小最大   | 二分答案    |

---

> 建议用法： **做题前翻目录 → 对号入座 → 直接套模板**

