# 动态规划 DP 基础

## 什么是 DP

DP = 分治 + 记忆化。把原问题**拆分**成同构子问题 + **重叠**子问题（能记住避免重算）。

### 适用条件（两个必须同时满足）

1. **最优子结构**：大问题最优解包含子问题最优解（贪心也需要这个）
2. **重叠子问题**：子问题被重复计算很多次（这是 DP 存在的理由）

### DP vs 贪心 vs 分治

| | 贪心 | DP | 分治 |
|--|------|-----|------|
| 重叠子问题 | 不管 | ✓ 记忆化 | 一般没有 |
| 最优子结构 | ✓ | ✓ | ✓ |
| 选择 | 当前最优就好 | 试所有选择 | 直接递归 |
| 例 | 区间调度 | 背包 | 归并排序 |

## DP 五步法（标准流程）

```
① 设计状态 dp[i] 或 dp[i][j]
   ↓
② 写出状态转移方程
   ↓
③ 确定边界（初始值）
   ↓
④ 确定计算顺序（保证用到的子问题已算好）
   ↓
⑤ 确定答案位置
```

## DP 分类总览

| 类型 | 代表 | 维度 | 复杂度 |
|------|------|------|--------|
| 线性 DP | LIS、LCS、数字三角形、打家劫舍 | 1~2 维 | O(n) ~ O(n²) |
| 背包 DP | 01、完全、分组、多重 | 1~2 维 | O(nW) |
| 区间 DP | 石子合并、矩阵链乘、最长回文子串 | 2 维 | O(n³) |
| 状态压缩 DP | TSP、棋盘放置、枚举子集 | 2 维（含 mask） | O(n·2^n) |
| 数位 DP | 统计区间内满足条件的数 | 多维含前缀 | O(len·state) |
| 树形 DP | 树的最大独立集、树的直径、重心 | 1~2 维（含 fa） | O(n) ~ O(n²) |
| 计数 DP | 跳台阶、不同路径、排列数 | 1~2 维 | O(n) ~ O(n²) |
| 概率 DP | 抛硬币、比赛胜负 | 1~2 维 | O(n) |

## 线性 DP 经典模型

### 1. 最长上升子序列 LIS

**状态**：`dp[i] = 以 a[i] 结尾的 LIS 长度`

**转移**：`dp[i] = max(dp[j]+1) 对所有 j<i 且 a[j]<a[i]`

**复杂度**：O(n²)

```cpp
int a[N], dp[N];

int lis_n2(int n)
{
    int ans = 0;
    for(int i = 1; i <= n; i++)
    {
        dp[i] = 1;
        for(int j = 1; j < i; j++)
        {
            if(a[j] < a[i]) dp[i] = max(dp[i], dp[j] + 1);
        }
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

**优化：O(n log n)** —— 贪心 + 二分

```cpp
int lis_fast(int n)
{
    vector<int> tail;     // tail[i] = 长度 i+1 的 LIS 的最小结尾
    for(int i = 1; i <= n; i++)
    {
        auto it = lower_bound(tail.begin(), tail.end(), a[i]);
        if(it == tail.end()) tail.push_back(a[i]);
        else *it = a[i];
    }
    return tail.size();
}
```

**变体**：
- LDS（下降）：把 `<` 改成 `>`，或 `greater`
- 非递减：`lower_bound` 改 `upper_bound` / `<=`
- 同时求 LIS 长度和个数：再加 `cnt[i]` 计数

### 2. 最长公共子序列 LCS

**状态**：`dp[i][j] = a[1..i] 和 b[1..j] 的 LCS 长度`

**转移**：
- 若 `a[i] == b[j]`：`dp[i][j] = dp[i-1][j-1] + 1`
- 否则：`dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

```cpp
int a[N], b[N];
int dp[N][N];

int lcs(int n, int m)
{
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= m; j++)
        {
            if(a[i] == b[j]) dp[i][j] = dp[i-1][j-1] + 1;
            else dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
        }
    return dp[n][m];
}
```

**空间优化**：LCS 只用到上一行，开两行 `dp[2][N]` 即可。

### 3. 数字三角形 / 最小路径和

```cpp
// 从顶到底，每步可到左下或右下，求最大和
int a[N][N];
int dp[N][N];

int triangle(int n)
{
    for(int j = 1; j <= n; j++) dp[n][j] = a[n][j];   // 最后一行
    for(int i = n-1; i >= 1; i--)
        for(int j = 1; j <= i; j++)
            dp[i][j] = a[i][j] + max(dp[i+1][j], dp[i+1][j+1]);
    return dp[1][1];
}
```

### 4. 打家劫舍（经典线性 DP）

```cpp
// 不能偷相邻房屋
int rob(vector<int>& a)
{
    int n = a.size();
    if(n == 1) return a[0];
    int prev2 = a[0];
    int prev1 = max(a[0], a[1]);
    for(int i = 2; i < n; i++)
    {
        int cur = max(prev1, prev2 + a[i]);
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**环形版本**：分成两个序列 `[0, n-2]` 和 `[1, n-1]` 分别求 max。

### 5. 爬楼梯（计数 DP）

```cpp
// 每次 1 或 2 阶，n 阶楼梯，多少种爬法？
int climb(int n)
{
    if(n <= 2) return n;
    int a = 1, b = 2;
    for(int i = 3; i <= n; i++)
    {
        int c = a + b;
        a = b; b = c;
    }
    return b;
}
```

### 6. 编辑距离（字符串 DP）

```cpp
// 最少操作把 a 变成 b（插入、删除、替换）
int edit_dist(string& a, string& b)
{
    int n = a.size(), m = b.size();
    vector<vector<int>> dp(n+1, vector<int>(m+1));
    for(int i = 0; i <= n; i++) dp[i][0] = i;    // 删光 a
    for(int j = 0; j <= m; j++) dp[0][j] = j;    // 插光 b
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= m; j++)
        {
            if(a[i-1] == b[j-1]) dp[i][j] = dp[i-1][j-1];
            else dp[i][j] = 1 + min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]});
            // 删 a[i] / 插 b[j] / 替换
        }
    return dp[n][m];
}
```

### 7. 最长递增子序列的个数（进阶）

```cpp
int a[N], dp[N], cnt[N];

void lis_cnt(int n)
{
    for(int i = 1; i <= n; i++) { dp[i] = 1; cnt[i] = 1; }
    for(int i = 1; i <= n; i++)
    {
        for(int j = 1; j < i; j++)
        {
            if(a[j] < a[i])
            {
                if(dp[j] + 1 > dp[i]) { dp[i] = dp[j]+1; cnt[i] = cnt[j]; }
                else if(dp[j] + 1 == dp[i]) { cnt[i] += cnt[j]; }
            }
        }
    }
}
```

## 空间优化的原则

**看转移需要用到哪一行/列**：
- 如果 `dp[i]` 只依赖 `dp[i-1]` → O(n)
- 如果 `dp[i][j]` 只依赖 `dp[i-1][j-1] / dp[i-1][j] / dp[i][j-1]`（如 LCS）→ 两行 O(m)
- 如果 `dp[i][j]` 只依赖 `dp[i-1][*]` → 背包优化（见背包章节）
- 如果只依赖 `dp[*][j]`（最长回文）→ O(n)

## 转移方程的对称感

很多方程对称：
```
dp[i][j] = dp[i-1][j-1] + 1    // LCS 两字符相同
dp[i][j] = dp[i-1][j] + dp[i][j-1] - dp[i-1][j-1]  // 二维前缀和
dp[i][j] = max(dp[i-1][j], dp[i][j-1])  // LCS 两字符不同
```

## 易错点

1. **循环顺序**：区间 DP 先枚举长度，再枚举左端点；树形 DP 先 DFS 再合并
2. **边界**：`dp[0]` 一般初始化为 0 或 1（计数），`dp[i][0]` 和 `dp[0][j]` 也要初始化
3. **下标**：1-indexed 还是 0-indexed？要统一
4. **取模**：答案要取模时每步都取，不要最后取
5. **枚举 k 的位置**：区间 DP 分割点 k 枚举 `[l, r)` 或 `(l, r]`？影响转移形式
6. **状态设计**：状态太多会 OOM，要精简维度
