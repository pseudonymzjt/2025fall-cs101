
# f-string输出格式：
```python
pi = 3.1415926535

# 保留2位小数
print(f"{pi:.2f}")      # 3.14

# 保留5位小数
print(f"{pi:.5f}")      # 3.14159

# 保留0位小数（取整）
print(f"{pi:.0f}")      # 3
```

---
# LRU_cache演示使用方法 数的划分
```python
from functools import lru_cache

# 例1：计算整数n的划分方案数（无序划分）
@lru_cache(maxsize=None)  # maxsize=None表示缓存没有大小限制
def partition(n, m=None):
    """
    将整数n划分成若干个正整数之和的方案数
    m: 划分中最大的部分
    """
    if m is None:
        m = n
    
    # 基础情况
    if n == 0:
        return 1
    if n < 0 or m == 0:
        return 0
    
    # 递归关系：包含m的划分 + 不包含m的划分
    return partition(n - m, m) + partition(n, m - 1)

# 测试
print(partition(5))  # 输出: 7
print(partition(10))  # 输出: 42
print(partition.cache_info())  # 查看缓存信息
```

---
# 快速幂
```python
def pow_mod(a, b, m):
    """快速幂取模 - 一行实现"""
    r = 1
    while b:
        if b & 1: r = r * a % m
        a = a * a % m
        b >>= 1
    return r % m
```

## 矩阵快速幂：
```python
def mat_mul(A, B, m):
    """矩阵乘法取模"""
    n = len(A)
    C = [[0]*n for _ in range(n)]
    for i in range(n):
        for j in range(n):
            for k in range(n):
                C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % m
    return C

def mat_pow(A, b, m):
    """矩阵快速幂"""
    n = len(A)
    # 单位矩阵
    res = [[1 if i==j else 0 for j in range(n)] for i in range(n)]
    while b:
        if b & 1:
            res = mat_mul(res, A, m)
        A = mat_mul(A, A, m)
        b >>= 1
    return res
```

---

# 区间问题：

## 1. 区间合并（Merge Intervals）：左端点排序

```python
def merge_intervals(intervals):
    """合并重叠区间"""
    if not intervals:
        return []
    
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    
    return merged
```

## 2. 无重叠区间（Non-overlapping Intervals）：右端点排序

```python
def erase_overlap_intervals(intervals):
    """移除最少数量的区间使剩余区间互不重叠"""
    if not intervals:
        return 0
    
    # 按结束时间排序
    intervals.sort(key=lambda x: x[1])
    count = 0
    end = intervals[0][1]
    
    for i in range(1, len(intervals)):
        if intervals[i][0] < end:  # 有重叠
            count += 1
        else:
            end = intervals[i][1]
    
    return count
```

## 3. 区间覆盖（Minimum Interval Coverage）：左端点排序，贪心右端点

```python
def min_intervals_to_cover(target, intervals):
    """选择最少数量的区间覆盖目标区间"""
    intervals.sort(key=lambda x: x[0])
    result = []
    i = 0
    current_end = target[0]
    
    while i < len(intervals) and current_end < target[1]:
        max_end = current_end
        # 选择能覆盖当前起点的最长区间
        while i < len(intervals) and intervals[i][0] <= current_end:
            max_end = max(max_end, intervals[i][1])
            i += 1
        
        if max_end == current_end:  # 无法扩展覆盖
            return []
        
        result.append([current_end, max_end])
        current_end = max_end
    
    return result if current_end >= target[1] else []
```

## 4.区间点覆盖：右端点排序

```python
n = int(input())
a = [int(i) for i in input().split()]
ranges = []
for i,d in enumerate(a):
    ranges.append([i - d,i + d])
ranges = sorted(ranges, key = lambda x:x[0])
right = 0 # the first right point which can't be covered
index = 0
cameras = 0
while right < n:
    cameras += 1
    rightmax = right - 1
    #print(f'right = {right} index = {index} cameras = {cameras}')
    while index < n and ranges[index][0] <= right: # index<n prevent out of range
        rightmax = max(ranges[index][1],rightmax)
        index += 1
    right = rightmax + 1 #the first uncovered point is the next og 
print(cameras)
```
$\rm index<n$防止越界，但是不能$\rm index<n - 1$，这样就会漏掉最后一个区间了
## 5. 会议室问题（Meeting Rooms）（校门外的树问题）

```python
def can_attend_meetings(intervals):
    """判断能否参加所有会议（无重叠）"""
    intervals.sort(key=lambda x: x[0])
    for i in range(1, len(intervals)):
        if intervals[i][0] < intervals[i-1][1]:
            return False
    return True

def min_meeting_rooms(intervals):
    """需要的最少会议室数量"""
    starts = sorted(i[0] for i in intervals)
    ends = sorted(i[1] for i in intervals)
    
    rooms = 0
    end_ptr = 0
    
    for start in starts:
        if start < ends[end_ptr]:
            rooms += 1
        else:
            end_ptr += 1
    
    return rooms
```

## 6. 区间查询优化（前缀和）

```python
def interval_sum(nums, intervals):
    """快速计算多个区间和（前缀和优化）"""
    n = len(nums)
    prefix = [0] * (n + 1)
    
    # 构建前缀和数组
    for i in range(1, n + 1):
        prefix[i] = prefix[i - 1] + nums[i - 1]
    
    # 查询区间和
    result = []
    for left, right in intervals:
        result.append(prefix[right + 1] - prefix[left])
    
    return result
```

## 7. 贪心区间选择（Select Intervals）：右端点排序，贪心右端点

```python
def select_intervals(intervals):
    """选择最多不重叠区间（贪心）"""
    if not intervals:
        return 0
    
    # 按结束时间排序
    intervals.sort(key=lambda x: x[1])
    count = 1
    end = intervals[0][1]
    
    for i in range(1, len(intervals)):
        if intervals[i][0] >= end:
            count += 1
            end = intervals[i][1]
    
    return count
```

## 关键技巧总结：

1. **排序是关键**：大多数区间问题都需要先按起点或终点排序
2. **贪心策略**：按结束时间排序通常能得到最优解
3. **扫描线算法**：将区间拆分为起点和终点事件
4. **差分数组**：用于区间批量更新
5. **前缀和**：用于快速区间查询
6. **双指针**：处理两个区间列表的交集

---

# 购物：dp+贪心
```python
value, ncoin = map(int,input().split())
coins = [int(_) for _ in input().split()]
coins = sorted(coins,reverse = True)
available = 0
cnt = 0
while available < value:
    # studying available + 1
    for coin in coins:
        if available + 1 - coin >= 0:
            cnt += 1
            available += coin
            break
print(cnt)
```

---

# candy贪心：

每一种方案都唯一对应一些$(x_i,y_i)$和一些互不相同的$x_i$

根据贪心的原理，我们应该全部选用最小的$(x_i+y_i)$，互不相同的$x_i$应该从小往大选

代码

```python
n,m = map(int,input().split())
x = []
summ = []
for i in range(n):
    xi,yi = map(int,input().split())
    x.append(xi)
    summ.append(xi+yi)
x = sorted(x)
couple = min(summ)

single = 0
epochs = m // couple * 2
for i in range(n):
    single += x[i]
    epochs = max(epochs,i + 1 + (m-single) // couple * 2)
print(epochs)
```

---
# Book my spacecraft内容

随着做题量的增加，你对栈和队列这两种数据结构的理解会越来越多元。

- 从数据处理的角度，栈和队列为数据处理提供了一种逻辑思路。

如[OpenJudge - T29947:校门外的树又来了](http://cs101.openjudge.cn/pctbook/T29947/)，我们可以利用`stack`
对种树的区间进行合并。按左端点排序遍历区间，如果当前区间与栈尾的区间有重叠，则更新栈尾的区间，否则将当前区间压入栈中。

```python
L, M = map(int, input().split())
interval = []
for _ in range(M):
    interval.append(list(map(int, input().split())))
interval.sort()
stack = []
for t in interval:
    if stack and t[0] <= stack[-1][1]:
        stack[-1][1] = max(stack[-1][1], t[1])
    else:
        stack.append(t)
print(L + 1 - sum(map(lambda x: x[1] - x[0] + 1, stack)))
```

如[OpenJudge - 27371:Playfair密码](http://cs101.openjudge.cn/practice/27371/)，其中有一步是将明文转变为字母对，我们可以利用
`stack`进行处理。

```python
word = input()
stack = []
for a in word:
    if a == 'j':
        a = 'i'
    if not stack:
        stack.append(a)
        continue
    if len(stack[-1]) == 2:
        stack.append(a)
        continue
    if stack[-1] != a:
        b = stack.pop()
        stack.append(b + a)
    else:
        if stack[-1] == 'x':
            stack.pop()
            stack.append('xq')
            stack.append(a)
        else:
            b = stack.pop()
            stack.append(b + 'x')
            stack.append(a)
if len(stack[-1]) == 1:
    if stack[-1] == 'x':
        stack.pop()
        stack.append('xq')
    else:
        b = stack.pop()
        stack.append(b + 'x')
```


---
# 贪心其它排序：

##  最大与最小整数：循环小数排序
```python
n = int(input())
a = [i for i in input().split()]
a = sorted(a,key = lambda x: int(x) / (10**(len(x))-1))
print(''.join(a[::-1]),''.join(a))
```



## 排队：拓扑排序
```python
n,d = map(int,input().split())
h = []
for i in range(n):
    h.append(int(input()))
result = []
used = [False] * n
while len(result)<n:
    seq = [] # Those who can swap to the front
    cur_max = -1
    cur_min = 1e10
    for i in range(0,n):
        if used[i]:#skip used men
            continue
        cur_max = max(cur_max,h[i])
        cur_min = min(cur_min,h[i])
        if cur_max - h[i] <= d and h[i] - cur_min <= d:
            used[i] = True
            seq.append(h[i])
    result = result + sorted(seq)# Fronters float
for j in result:
    print(j)
```

## 国王游戏：乘积排序

```python
n = int(input())
a,b = map(int,input().split())
money = []
prod = [1]*n
minister = [1]*n
for i in range(n):
    money.append(list(map(int,input().split())))
money = sorted(money,key = lambda x:x[0]*x[1])
prod[0] = 1
minister[0] = 1 // money[0][1]
for i in range(1,n):
    prod[i] = prod[i-1]*money[i-1][0]
    minister[i] = prod[i] // money[i][1]
print(max(minister))
```

## 总结：相邻元素试试对换，然后归纳总结

---
# 动态规划：

## Kadane

```python
def max_subarray_sum(arr):
    """基础Kadane算法：返回最大子数组和"""
    max_sum = curr_sum = arr[0]
    for num in arr[1:]:
        curr_sum = max(num, curr_sum + num)
        max_sum = max(max_sum, curr_sum)
    return max_sum

def max_subarray(arr):
    """Kadane算法：返回最大和及子数组的起止索引"""
    max_sum = curr_sum = arr[0]
    start = end = 0
    curr_start = 0
    
    for i in range(1, len(arr)):
        if arr[i] > curr_sum + arr[i]:
            curr_sum = arr[i]
            curr_start = i
        else:
            curr_sum = curr_sum + arr[i]
        
        if curr_sum > max_sum:
            max_sum = curr_sum
            start = curr_start
            end = i
    
    return max_sum, start, end
```
## Manacher

```python
def longest_palindrome(s):
    """Manacher算法：返回最长回文子串"""
    if not s:
        return ""
    
    # 预处理字符串，插入特殊字符
    t = '#' + '#'.join(s) + '#'
    n = len(t)
    p = [0] * n  # 回文半径数组
    center = right = max_len = start = 0
    
    for i in range(n):
        # 利用对称性初始化回文半径
        if i < right:
            p[i] = min(right - i, p[2 * center - i])
        
        # 尝试扩展回文
        while i - p[i] - 1 >= 0 and i + p[i] + 1 < n and t[i - p[i] - 1] == t[i + p[i] + 1]:
            p[i] += 1
        
        # 更新中心和右边界
        if i + p[i] > right:
            center = i
            right = i + p[i]
        
        # 更新最长回文信息
        if p[i] > max_len:
            max_len = p[i]
            start = (i - max_len) // 2
    
    return s[start:start + max_len]
```

## TSP 问题：
```python
from functools import lru_cache

n = int(input())
cost = [list(map(int, input().split())) for _ in range(n)]

@lru_cache(maxsize=None)
def dfs(mask, i):
    if mask == (1 << n) - 1:
        return cost[i][0] or float('inf')
    
    res = float('inf')
    for j in range(n):
        if not (mask >> j & 1) and cost[i][j]:
            res = min(res, cost[i][j] + dfs(mask | (1 << j), j))
    return res

print(dfs(1, 0))  # 从城市0开始，mask=1表示城市0已访问
```

## 动态规划经典序列

### 一、基础模板结构

```python
def dp_sequence_template(seq):
    """基础一维DP模板"""
    n = len(seq)
    # 1. 状态定义
    dp = [0] * (n + 1)  # 根据问题调整维度
    
    # 2. 初始化
    dp[0] = 初始值  # 根据问题调整
    
    # 3. 状态转移
    for i in range(1, n + 1):
        # 根据问题逻辑填充
        dp[i] = max/min(dp[i-1], 其他状态) + 相关计算
    
    # 4. 返回结果
    return dp[n]

def dp_2d_template(seq1, seq2):
    """二维DP模板"""
    m, n = len(seq1), len(seq2)
    # 1. 状态定义
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # 2. 初始化（第一行和第一列）
    for i in range(m + 1):
        dp[i][0] = 初始值
    for j in range(n + 1):
        dp[0][j] = 初始值
    
    # 3. 状态转移
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if 条件:
                dp[i][j] = 转移方程1
            else:
                dp[i][j] = 转移方程2
    
    return dp[m][n]  # 或根据问题返回其他值
```

### 二、具体问题模板

#### 1. 最长递增子序列 (LIS)
```python
def length_of_LIS(nums):
    """最长递增子序列 O(n²)"""
    if not nums:
        return 0
    
    n = len(nums)
    dp = [1] * n  # dp[i]表示以nums[i]结尾的最长递增子序列长度
    
    for i in range(n):
        for j in range(i):
            if nums[i] > nums[j]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

def length_of_LIS_binary(nums):
    """最长递增子序列 O(nlogn) 贪心+二分"""
    tails = []  # tails[k]表示长度为k+1的子序列的尾元素最小值
    
    for num in nums:
        # 二分查找第一个≥num的位置
        left, right = 0, len(tails)
        while left < right:
            mid = (left + right) // 2
            if tails[mid] < num:
                left = mid + 1
            else:
                right = mid
        
        if left == len(tails):
            tails.append(num)
        else:
            tails[left] = num
    
    return len(tails)

def print_LIS(nums):
    """输出最长递增子序列（而不仅仅是长度）"""
    if not nums:
        return []
    
    n = len(nums)
    dp = [1] * n
    prev = [-1] * n  # 记录前驱索引
    
    for i in range(n):
        for j in range(i):
            if nums[i] > nums[j] and dp[j] + 1 > dp[i]:
                dp[i] = dp[j] + 1
                prev[i] = j
    
    # 找到最长序列的结尾位置
    max_len = max(dp)
    end_idx = dp.index(max_len)
    
    # 回溯重建序列
    result = []
    while end_idx != -1:
        result.append(nums[end_idx])
        end_idx = prev[end_idx]
    
    return result[::-1]
```

#### 2. 最长公共子序列 (LCS)
```python
def longest_common_subsequence(text1, text2):
    """最长公共子序列"""
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    # 如果需要重建LCS
    def build_lcs():
        i, j = m, n
        lcs = []
        while i > 0 and j > 0:
            if text1[i-1] == text2[j-1]:
                lcs.append(text1[i-1])
                i -= 1
                j -= 1
            elif dp[i-1][j] > dp[i][j-1]:
                i -= 1
            else:
                j -= 1
        return ''.join(reversed(lcs))
    
    return dp[m][n], build_lcs()

def LCS_optimized(text1, text2):
    """空间优化的LCS（只保留两行）"""
    if len(text1) < len(text2):
        text1, text2 = text2, text1
    
    m, n = len(text1), len(text2)
    prev = [0] * (n + 1)
    curr = [0] * (n + 1)
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        prev, curr = curr, [0] * (n + 1)
    
    return prev[n]
```

#### 3. 最长回文子序列/子串
```python
def longest_palindromic_subsequence(s):
    """最长回文子序列"""
    n = len(s)
    dp = [[0] * n for _ in range(n)]
    
    # 单个字符都是回文
    for i in range(n):
        dp[i][i] = 1
    
    # 从长度2开始
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1] + 2
            else:
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
    
    return dp[0][n-1]

def longest_palindromic_substring(s):
    """最长回文子串"""
    n = len(s)
    if n < 2:
        return s
    
    # dp[i][j]表示s[i:j+1]是否是回文
    dp = [[False] * n for _ in range(n)]
    start, max_len = 0, 1
    
    # 单个字符都是回文
    for i in range(n):
        dp[i][i] = True
    
    # 两个字符的情况
    for i in range(n - 1):
        if s[i] == s[i+1]:
            dp[i][i+1] = True
            start = i
            max_len = 2
    
    # 长度≥3的情况
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i+1][j-1]:
                dp[i][j] = True
                start = i
                max_len = length
    
    return s[start:start+max_len]

def longest_palindromic_substring_center(s):
    """中心扩展法求最长回文子串 O(n²) 但空间O(1)"""
    def expand_around_center(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return left + 1, right - 1
    
    start, end = 0, 0
    for i in range(len(s)):
        # 奇数长度回文
        l1, r1 = expand_around_center(i, i)
        # 偶数长度回文
        l2, r2 = expand_around_center(i, i + 1)
        
        if r1 - l1 > end - start:
            start, end = l1, r1
        if r2 - l2 > end - start:
            start, end = l2, r2
    
    return s[start:end+1]
```

#### 4. 编辑距离
```python
def edit_distance(word1, word2):
    """最小编辑距离（Levenshtein距离）"""
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # 初始化
    for i in range(m + 1):
        dp[i][0] = i  # 删除i次
    for j in range(n + 1):
        dp[0][j] = j  # 插入j次
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]  # 字符相同，无需操作
            else:
                # 分别对应：替换、删除、插入
                dp[i][j] = 1 + min(
                    dp[i-1][j-1],  # 替换
                    dp[i-1][j],    # 删除word1[i-1]
                    dp[i][j-1]     # 插入word2[j-1]
                )
    
    return dp[m][n]

def edit_distance_with_weights(word1, word2, cost_ins=1, cost_del=1, cost_rep=1):
    """带权重的编辑距离"""
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(m + 1):
        dp[i][0] = i * cost_del
    for j in range(n + 1):
        dp[0][j] = j * cost_ins
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = min(
                    dp[i-1][j-1] + cost_rep,  # 替换
                    dp[i-1][j] + cost_del,    # 删除
                    dp[i][j-1] + cost_ins     # 插入
                )
    
    return dp[m][n]
```

#### 5. 最大子数组和
```python
def max_subarray_sum(nums):
    """最大子数组和（Kadane算法）"""
    if not nums:
        return 0
    
    curr_sum = max_sum = nums[0]
    
    for num in nums[1:]:
        curr_sum = max(num, curr_sum + num)
        max_sum = max(max_sum, curr_sum)
    
    return max_sum

def max_subarray_sum_with_indices(nums):
    """返回最大子数组和及其起始结束位置"""
    if not nums:
        return 0, -1, -1
    
    curr_sum = max_sum = nums[0]
    start = end = 0
    temp_start = 0
    
    for i in range(1, len(nums)):
        if nums[i] > curr_sum + nums[i]:
            curr_sum = nums[i]
            temp_start = i
        else:
            curr_sum += nums[i]
        
        if curr_sum > max_sum:
            max_sum = curr_sum
            start = temp_start
            end = i
    
    return max_sum, start, end

def max_product_subarray(nums):
    """最大乘积子数组"""
    if not nums:
        return 0
    
    max_prod = min_prod = result = nums[0]
    
    for i in range(1, len(nums)):
        if nums[i] < 0:
            max_prod, min_prod = min_prod, max_prod
        
        max_prod = max(nums[i], max_prod * nums[i])
        min_prod = min(nums[i], min_prod * nums[i])
        
        result = max(result, max_prod)
    
    return result
```

#### 6. 正则表达式匹配
```python
def is_match_regex(s, p):
    """正则表达式匹配：.匹配任意字符，*匹配前一个字符0次或多次"""
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    
    # 空字符串匹配空模式
    dp[0][0] = True
    
    # 处理模式中的*可以匹配空字符串的情况
    for j in range(1, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-2]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == s[i-1] or p[j-1] == '.':
                dp[i][j] = dp[i-1][j-1]
            elif p[j-1] == '*':
                # *匹配0次或多次
                dp[i][j] = dp[i][j-2]  # 匹配0次
                if p[j-2] == s[i-1] or p[j-2] == '.':
                    dp[i][j] = dp[i][j] or dp[i-1][j]  # 匹配1次或多次
    
    return dp[m][n]

def wildcard_match(s, p):
    """通配符匹配：?匹配任意单个字符，*匹配任意序列（包括空）"""
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    
    dp[0][0] = True
    
    # 处理开头的*匹配空字符串的情况
    for j in range(1, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-1]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == s[i-1] or p[j-1] == '?':
                dp[i][j] = dp[i-1][j-1]
            elif p[j-1] == '*':
                dp[i][j] = dp[i-1][j] or dp[i][j-1]  # 匹配多个字符 or 匹配空
    
    return dp[m][n]
```

#### 7. 股票买卖系列
```python
def max_profit_one_transaction(prices):
    """只能买卖一次"""
    if not prices:
        return 0
    
    min_price = float('inf')
    max_profit = 0
    
    for price in prices:
        min_price = min(min_price, price)
        max_profit = max(max_profit, price - min_price)
    
    return max_profit

def max_profit_unlimited_transactions(prices):
    """可以无限次买卖"""
    profit = 0
    for i in range(1, len(prices)):
        if prices[i] > prices[i-1]:
            profit += prices[i] - prices[i-1]
    return profit

def max_profit_k_transactions(k, prices):
    """最多k次买卖"""
    if not prices:
        return 0
    
    n = len(prices)
    if k >= n // 2:  # 相当于无限次交易
        return max_profit_unlimited_transactions(prices)
    
    # dp[i][j]表示第i天进行j次交易的最大利润
    dp = [[0] * (k + 1) for _ in range(n)]
    
    for j in range(1, k + 1):
        max_diff = -prices[0]
        for i in range(1, n):
            dp[i][j] = max(dp[i-1][j], prices[i] + max_diff)
            max_diff = max(max_diff, dp[i][j-1] - prices[i])
    
    return dp[n-1][k]

def max_profit_with_cooldown(prices):
    """含冷冻期（卖出后需等待一天）"""
    if not prices:
        return 0
    
    n = len(prices)
    # 状态：0-持有股票，1-不持有股票且在冷冻期，2-不持有股票且不在冷冻期
    dp = [[0] * 3 for _ in range(n)]
    dp[0][0] = -prices[0]  # 第一天买入
    
    for i in range(1, n):
        # 第i天持有股票：前一天已持有 or 第i天买入（前一天不持有且不在冷冻期）
        dp[i][0] = max(dp[i-1][0], dp[i-1][2] - prices[i])
        # 第i天不持有且在冷冻期：前一天卖出股票
        dp[i][1] = dp[i-1][0] + prices[i]
        # 第i天不持有且不在冷冻期：前一天就不持有（可能在冷冻期或不在）
        dp[i][2] = max(dp[i-1][1], dp[i-1][2])
    
    return max(dp[n-1][1], dp[n-1][2])
```

#### 8. 打家劫舍系列
```python
def rob_linear(nums):
    """线性排列"""
    if not nums:
        return 0
    
    n = len(nums)
    if n == 1:
        return nums[0]
    
    dp = [0] * n
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    
    for i in range(2, n):
        dp[i] = max(dp[i-1], dp[i-2] + nums[i])
    
    return dp[n-1]

def rob_linear_optimized(nums):
    """空间优化的线性排列"""
    prev, curr = 0, 0
    for num in nums:
        prev, curr = curr, max(curr, prev + num)
    return curr

def rob_circular(nums):
    """环形排列（首尾相连）"""
    if not nums:
        return 0
    
    n = len(nums)
    if n == 1:
        return nums[0]
    
    # 两种情况：不偷第一家 or 不偷最后一家
    return max(rob_linear(nums[:n-1]), rob_linear(nums[1:]))

def rob_tree(root):
    """树形排列（二叉树）"""
    def dfs(node):
        if not node:
            return (0, 0)  # (偷当前节点的最大值, 不偷当前节点的最大值)
        
        left = dfs(node.left)
        right = dfs(node.right)
        
        # 偷当前节点
        rob_curr = node.val + left[1] + right[1]
        # 不偷当前节点
        not_rob_curr = max(left) + max(right)
        
        return (rob_curr, not_rob_curr)
    
    return max(dfs(root))
```

#### 9. 跳跃游戏
```python
def can_jump(nums):
    """能否到达最后一个位置"""
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach:  # 当前位置无法到达
            return False
        max_reach = max(max_reach, i + jump)
        if max_reach >= len(nums) - 1:
            return True
    return max_reach >= len(nums) - 1

def min_jumps(nums):
    """到达最后一个位置的最少跳跃次数"""
    if len(nums) <= 1:
        return 0
    
    jumps = 0
    curr_end = 0
    farthest = 0
    
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])
        
        if i == curr_end:
            jumps += 1
            curr_end = farthest
            
            if curr_end >= len(nums) - 1:
                break
    
    return jumps
```

#### 10. 矩阵链乘法
```python
def matrix_chain_multiplication(dims):
    """最小化矩阵相乘的计算代价"""
    n = len(dims) - 1  # 矩阵个数
    # dp[i][j]表示计算矩阵A[i]...A[j]的最小代价
    dp = [[0] * n for _ in range(n)]
    
    # l是链的长度
    for l in range(2, n + 1):
        for i in range(n - l + 1):
            j = i + l - 1
            dp[i][j] = float('inf')
            
            # 尝试所有可能的划分点k
            for k in range(i, j):
                cost = dp[i][k] + dp[k+1][j] + dims[i] * dims[k+1] * dims[j+1]
                dp[i][j] = min(dp[i][j], cost)
    
    return dp[0][n-1]

def matrix_chain_with_parenthesis(dims):
    """返回最优括号化方案"""
    n = len(dims) - 1
    dp = [[0] * n for _ in range(n)]
    split = [[0] * n for _ in range(n)]  # 记录分割点
    
    for l in range(2, n + 1):
        for i in range(n - l + 1):
            j = i + l - 1
            dp[i][j] = float('inf')
            
            for k in range(i, j):
                cost = dp[i][k] + dp[k+1][j] + dims[i] * dims[k+1] * dims[j+1]
                if cost < dp[i][j]:
                    dp[i][j] = cost
                    split[i][j] = k
    
    # 重建括号化方案
    def build(i, j):
        if i == j:
            return f"A{i+1}"
        else:
            k = split[i][j]
            left = build(i, k)
            right = build(k+1, j)
            return f"({left}×{right})"
    
    return dp[0][n-1], build(0, n-1)
```

#### 11. 单词拆分
```python
def word_break(s, word_dict):
    """判断能否拆分"""
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True  # 空字符串
    
    word_set = set(word_dict)
    
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[n]

def word_break_all(s, word_dict):
    """返回所有可能的拆分"""
    word_set = set(word_dict)
    memo = {}
    
    def backtrack(start):
        if start in memo:
            return memo[start]
        
        if start == len(s):
            return [""]
        
        result = []
        for end in range(start + 1, len(s) + 1):
            word = s[start:end]
            if word in word_set:
                for subsentence in backtrack(end):
                    if subsentence:
                        result.append(word + " " + subsentence)
                    else:
                        result.append(word)
        
        memo[start] = result
        return result
    
    return backtrack(0)
```

---

# 接雨水：单调栈
1. 双指针
每个点的积水高度为其左侧的$\rm max\_height$和右侧$\rm max\_height$的minimum
将从左往右和从右往左的$\rm max\_height$记为$\rm leftmax$和$\rm rightmax$
因此如果短板在左侧：$\rm leftmax(pos - 1) < \rm rightmax(pos + k)$，则该格水位高度为左侧的max
此时可以把pos的水量算出来，并向右前进一格）（或者更新max）
如果不然，则右侧为短板，显然右侧可以求出水量并累计，右指针向左进发

用双指针法简单得多：
```python
t = int(input())
for _ in range(t):
    n = int(input())
    height = [int(i) for i in input().split()]
    leftmax = 0
    rightmax = 0
    left = 0
    right = n - 1
    water = 0
    while left <= right:
        if leftmax <= rightmax:# move left
            water += max(leftmax - height[left],0)
            leftmax = max(leftmax, height[left])
            left += 1
        else:
            water += max(rightmax - height[right],0)
            rightmax = max(rightmax, height[right])
            right -= 1
    print(water)
```
2. dp
维护一个点的leftmax和rightmax数组（$\rm O(n)$）然后直接求和
```python
t = int(input())
for _ in range(t):
    n = int(input())
    height = [int(i) for i in input().split()]
    leftmax = [-1] * n
    rightmax = [-1] * n
    leftmax[0] = height[0]
    rightmax[n - 1] = height[n - 1]
    water = 0
    for i in range(1,n):
        leftmax[i] = max(leftmax[i - 1], height[i])
    for i in range(n - 2,-1,-1):
        rightmax[i] = max(rightmax[i + 1], height[i])
    for i in range(n):
        water += min(leftmax[i], rightmax[i]) - height[i]
    print(water)
```
3. 单调栈
```python
n = int(input())
h = list(map(int, input().split()))

stack = []  # 存储索引的单调递减栈
water = 0

for i, height in enumerate(h):
    # 当当前高度大于栈顶高度时，可以形成凹槽
    while stack and h[stack[-1]] < height:
        bottom = stack.pop()  # 弹出底部
        if not stack:
            break  # 没有左边界，无法接水
        left = stack[-1]  # 左边界索引
        width = i - left - 1  # 凹槽宽度
        depth = min(h[left], height) - h[bottom]  # 可接水深度
        water += width * depth
    
    stack.append(i)

print(water)
```


---
# 护林员盖房子：单调栈
这个单调栈搞起来真麻烦，一开始调试的时候死活调不对。这个和接雨水是类似的，都需要找到下凹或者上突序列并统计。**单调栈可以理解为：每次弹出都相当于填平沟壑（或者削平山峰）**

<mark>还没有代码</mark>

---
# 删除数字问题：单调栈

```python
s = input().strip()
k = int(input())

stack = []
to_remove = k

for ch in s:
    while stack and stack[-1] > ch and to_remove > 0:
        stack.pop()
        to_remove -= 1
    stack.append(ch)

# 如果还有剩余删除次数，从末尾删除（因为此时栈是递增的）
result = ''.join(stack[:len(stack) - to_remove])

# 去除前导零（但保留最后一个0）
print(int(result) if result else 0)
```
---
# Dijkstra算法：
```python
import heapq

def dijkstra(n, start, graph):
    """邻接矩阵版 Dijkstra"""
    dist = [float('inf')] * n
    dist[start] = 0
    heap = [(0, start)]  # (距离, 节点)
    
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]:
            continue
        for v in range(n):
            if graph[u][v] > 0 and dist[v] > d + graph[u][v]:
                dist[v] = d + graph[u][v]
                heapq.heappush(heap, (dist[v], v))
    return dist
```

---
# 食物链：边上带有信息的并查集
```python
class UFS:
    def __init__(self, items = 0):
        """initialization"""
        self.point = [(i,0) for i in range(items)] # phase: relative to me (x)!
        self.depth = [1 for i in range(items)]

    def union(self, x, y, phase):# phase: y relative to x
        x, phase_x = self.find(x)
        y, phase_y = self.find(y)
        phase = phase - phase_x + phase_y
        phase %= 3
        if x == y:
            if phase != 0:
                return False
            else:
                return True
        if self.depth[x] > self.depth[y]:
            self.point[y] = (x,(-phase) % 3)
            self.depth[x] = max(self.depth[x], self.depth[y] + 1)
        else:
            self.point[x] = (y,phase)
            self.depth[y] = max(self.depth[y], self.depth[x] + 1)
        return True

    def find(self, x):
        #print('x= ',x)
        if self.point[x][0] == x:
            return (x, 0)
        root, phase = self.find(self.point[x][0])
        phase += self.point[x][1]
        phase %= 3
        self.point[x] = (root, phase)
        return (root, phase)

    def __str__(self):
        return str(self.point)

n,k = map(int,input().split())
ufs = UFS(n + 1)
fake = 0
for i in range(k):
    d,x,y = map(int,input().split())
    if (x > n) or (y > n) or (not ufs.union(x,y,d-1)):
        fake += 1
print(fake)

```

AI version:
```python
class UFS:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rel = [0] * n  # rel[i] 表示 i 到 parent[i] 的关系 (0:同类, 1: i 吃 parent[i], 2: parent[i] 吃 i)
        self.depth = [1] * n  # 按秩合并的深度

    def find(self, x):
        if self.parent[x] == x:
            return (x, 0)
        root, r = self.find(self.parent[x])
        # 更新 x 到根节点的关系：x->parent[x] + parent[x]->root
        new_rel = (self.rel[x] + r) % 3
        self.parent[x] = root
        self.rel[x] = new_rel
        return (root, new_rel)

    def union(self, x, y, phase):
        x_root, x_rel = self.find(x)
        y_root, y_rel = self.find(y)
        if x_root == y_root:
            # 检查 x 和 y 的关系是否符合要求
            if (x_rel - y_rel) % 3 != phase:
                return False
            return True
        else:
            # 按深度合并：将深度小的树挂到深度大的树上
            if self.depth[x_root] >= self.depth[y_root]:
                # 将 y_root 挂到 x_root 上
                r = (x_rel - y_rel - phase + 3) % 3
                self.parent[y_root] = x_root
                self.rel[y_root] = r
                if self.depth[x_root] == self.depth[y_root]:
                    self.depth[x_root] += 1
                return True
            else:
                # 将 x_root 挂到 y_root 上
                r = (phase - x_rel + y_rel + 3) % 3
                self.parent[x_root] = y_root
                self.rel[x_root] = r
                if self.depth[x_root] == self.depth[y_root]:
                    self.depth[y_root] += 1
                return True

def main():
    import sys
    data = sys.stdin.read().split()
    if not data:
        return
    n = int(data[0])
    k = int(data[1])
    ufs = UFS(n)
    fake = 0
    index = 2
    for _ in range(k):
        d = int(data[index])
        x = int(data[index + 1])
        y = int(data[index + 2])
        index += 3
        # 检查 2: X 或 Y 超出范围
        if x > n or y > n:
            fake += 1
            continue
        # 检查 3: X 吃 X
        if d == 2 and x == y:
            fake += 1
            continue
        # 转换为 0-indexed
        x0 = x - 1
        y0 = y - 1
        if not ufs.union(x0, y0, d - 1):
            fake += 1
    print(fake)

if __name__ == "__main__":
    main()
```

---
# next_permutation:

## 算法1：Narayana Pandita 算法（直接模拟k次）

```python
import bisect

def succ(arr, n):
    for _ in range(k):
        i = n - 2
        while i >= 0 and arr[i] >= arr[i + 1]:
            i -= 1
        if i < 0:
            arr = list(range(1, n + 1))
            continue
        j = n - 1
        while arr[j] <= arr[i]:
            j -= 1
        arr[i], arr[j] = arr[j], arr[i]
        arr[i + 1:] = reversed(arr[i + 1:])
    return arr

t = int(input())
for _ in range(t):
    n, k = map(int, input().split())
    arr = list(map(int, input().split()))
    print(*succ(arr, n))
```

## 算法2：康托展开（数学方法）

```python
def succ(arr, n, k):
    # 康托展开
    fact = 1
    for i in range(2, n):
        fact *= i
    
    code = [0] * n
    for i in range(n):
        cnt = 0
        for j in range(i + 1, n):
            if arr[j] < arr[i]:
                cnt += 1
        code[i] = cnt
    
    # 将康托编码视为特殊进制数并加k
    carry = n - 1
    code[-1] += k
    for i in range(n - 1, 0, -1):
        if code[i] > carry:
            code[i - 1] += code[i] // (carry + 1)
            code[i] %= (carry + 1)
        carry -= 1
    
    # 逆康托展开
    used = [False] * (n + 1)
    result = []
    for i in range(n):
        cnt = code[i]
        num = 1
        while cnt or used[num]:
            if not used[num]:
                cnt -= 1
            num += 1
        used[num] = True
        result.append(num)
    
    return result

t = int(input())
for _ in range(t):
    n, k = map(int, input().split())
    arr = list(map(int, input().split()))
    print(*succ(arr, n, k))
```

## 极简版（使用Python内置函数）

如果允许使用Python的itertools，可以更简单：

```python
import itertools

t = int(input())
for _ in range(t):
    n, k = map(int, input().split())
    arr = tuple(map(int, input().split()))
    perms = list(itertools.permutations(range(1, n + 1)))
    idx = perms.index(arr)
    result = perms[(idx + k) % len(perms)]
    print(*result)
```

## 主要精简点：

1. **移除多余函数定义**：将函数内联到主逻辑中
2. **简化变量名**：使用更短的变量名
3. **优化算法逻辑**：
   - 算法1：使用`reversed()`简化序列反转
   - 算法2：简化康托编码的计算
4. **减少代码行数**：通过合并操作减少代码行数
5. **使用Python特性**：如列表切片、内置函数等

**注意**：算法2中，carry的变化是因为第i位的最大值为`n-i-1`，所以进制从`n-1`递减到`0`。

---

# 跳房子

```python
from collections import deque

while True:
    n,m = map(int,input().split())
    if n == 0 and m == 0:
        break
    # Create a deque object
    queue = deque()
    
    queue.append([n,'']) # position, sequence
    visited = {n}
    if m == n:
        print('1\n\n')
    while True:
        pos,seq = queue.popleft()
        for i in range(2):
            if i==0:
                nowpos, nowseq = pos * 3, seq + 'H'
            else:
                nowpos, nowseq = pos // 2, seq + 'O'
            if nowpos in visited:
                continue
            queue.append([nowpos,nowseq])
            visited.add(nowpos)
            if nowpos == m:
                print(len(nowseq))
                print(nowseq)
                break
        if nowpos == m:
            break
        if len(nowseq) > 25:
            print('no solution')
            break

```

---
# 熄灯问题：
深搜
```python
def solve_lights_out():
    # 读入初始状态
    grid = [list(map(int, input().split())) for _ in range(5)]
    
    # 枚举第一行的所有操作状态
    for first in range(1 << 6):
        cur = [row[:] for row in grid]  # 当前状态
        press = [[0] * 6 for _ in range(5)]  # 操作记录
        
        # 设置第一行的操作
        for j in range(6):
            if first >> j & 1:
                press[0][j] = 1
                # 按下(0,j)的影响
                for dx, dy in [(0,0), (-1,0), (1,0), (0,-1), (0,1)]:
                    x, y = dx, dy + j
                    if 0 <= x < 5 and 0 <= y < 6:
                        cur[x][y] ^= 1
        
        # 递推确定其他行的操作
        for i in range(1, 5):
            for j in range(6):
                if cur[i-1][j] == 1:  # 上一行灯亮着，必须按当前位置
                    press[i][j] = 1
                    for dx, dy in [(0,0), (-1,0), (1,0), (0,-1), (0,1)]:
                        x, y = i + dx, j + dy
                        if 0 <= x < 5 and 0 <= y < 6:
                            cur[x][y] ^= 1
        
        # 检查最后一行是否全灭
        if all(cur[4][j] == 0 for j in range(6)):
            for row in press:
                print(' '.join(map(str, row)))
            return

# 调用函数
if __name__ == "__main__":
    solve_lights_out()
```

---
# 基于堆的优先队列卷积，并取出前几个元素：

```python
import heapq

T = int(input())

def convolve(seqs, n):
    """使用堆合并多个有序序列的前n个最小和"""
    if not seqs:
        return []
    if len(seqs) == 1:
        return seqs[0][:n]
    
    # 递归分治合并
    mid = len(seqs) // 2
    left = convolve(seqs[:mid], n)
    right = convolve(seqs[mid:], n)
    
    # 使用堆合并两个有序序列的前n个最小和
    heap = [(left[0] + right[0], 0, 0)]
    visited = {(0, 0)}
    result = []
    
    while len(result) < n:
        num, i, j = heapq.heappop(heap)
        result.append(num)
        
        # 扩展下一个可能的组合
        if i + 1 < len(left) and (i + 1, j) not in visited:
            heapq.heappush(heap, (left[i + 1] + right[j], i + 1, j))
            visited.add((i + 1, j))
        if j + 1 < len(right) and (i, j + 1) not in visited:
            heapq.heappush(heap, (left[i] + right[j + 1], i, j + 1))
            visited.add((i, j + 1))
    
    return result

for _ in range(T):
    m, n = map(int, input().split())
    seqs = []
    
    # 读取m个有序序列
    for _ in range(m):
        seq = sorted(map(int, input().split()))
        seqs.append(seq)
    
    # 计算合并后的前n个最小和
    result = convolve(seqs, n)
    print(*result)
```
---
# merge sort + count inv
```python
n = int(input())
num_list = [int(i) for i in input().split()]
inv = 0

def merge_sort(L,R):
    global num_list, inv
    if L+1 >= R:
        return
    mid = (L + R) // 2
    merge_sort(L,mid)
    merge_sort(mid,R)
    index_left = L
    index_right = mid
    combined_list = []
    while True:
        if index_left >= mid or index_right >= R:
            break
        if num_list[index_left] <= num_list[index_right]:
            combined_list.append(num_list[index_left])
            index_left += 1
        else:
            combined_list.append(num_list[index_right])
            inv += mid - index_left
            index_right += 1
    while index_left < mid:
        combined_list.append(num_list[index_left])
        index_left += 1
    while index_right < R:
        combined_list.append(num_list[index_right])
        index_right += 1
    for i in range(L,R):
        num_list[i] = combined_list[i - L]

merge_sort(0,n)
print(inv)

```

---