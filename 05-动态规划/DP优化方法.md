# 动态规划优化

## 常见优化方法

| 方法 | 作用 | 典型场景 |
|------|------|----------|
| 滚动数组 | 空间 O(n) → O(k) | 背包、只依赖前一两层的 DP |
| 单调队列 | 时间 O(n²) → O(n) | 转移区间是滑动窗口 |
| 前缀和 | 时间 O(n²) → O(n) | 转移是一段区间的求和 |
| 斜率优化 | 特定形式的 1D/1D | 了解即可，ICPC 低频 |
| 四边形不等式 | Knuth 优化 | 了解即可，区间 DP 少数题 |

拿奖优先掌握前三种，后两种看到题能认出来就够。

## 滚动数组

DP 只依赖前一层时，可以把第一维压掉。空间从 O(n·W) 降到 O(W) 甚至 O(1)。

```cpp
// 背包只保留两行
int dp[2][W+1];
int cur = 0;
for(int i = 1; i <= n; i++)
{
    cur ^= 1;
    for(int j = 0; j <= W; j++)
    {
        dp[cur][j] = dp[cur^1][j];
        if(j >= w[i]) dp[cur][j] = max(dp[cur][j], dp[cur^1][j-w[i]] + v[i]);
    }
}
```

01 背包还可以直接压成一维，但**容量必须倒序**：

```cpp
for(int i = 1; i <= n; i++)
    for(int j = W; j >= w[i]; j--)
        dp[j] = max(dp[j], dp[j-w[i]] + v[i]);
```

**要点**：倒序保证每件物品只用一次；完全背包改成正序就是“可以用多次”。

## 单调队列优化 DP

转移形如 `dp[i] = max(dp[j]) + a[i]`，其中 j 只能取 [i-k, i-1]。用单调队列维护这个窗口的最大值，O(n²) 降到 O(n)。

```cpp
const int N = 100005;
int dp[N], a[N];
deque<int> dq;   // 存下标，dp 值递减

int solve(int n, int k)
{
    dq.clear();
    dp[0] = 0;
    dq.push_back(0);
    for(int i = 1; i <= n; i++)
    {
        while(!dq.empty() && dq.front() < i - k) dq.pop_front();
        dp[i] = dp[dq.front()] + a[i];
        while(!dq.empty() && dp[dq.back()] <= dp[i]) dq.pop_back();
        dq.push_back(i);
    }
    return dp[n];
}
```

**顺序不能反**：先弹掉过期下标，再取队头转移，最后把当前 i 入队。入队太早会让 i 用到自己。

## 前缀和优化 DP

转移是一段区间的和时，用前缀和把枚举 j 的循环省掉。

```cpp
// dp[i] = sum(dp[j]), j ∈ [i-k, i-1]
int dp[N], sum[N];

void solve(int n, int k)
{
    dp[0] = 1;
    sum[0] = 1;
    for(int i = 1; i <= n; i++)
    {
        int l = max(0, i - k);
        dp[i] = sum[i-1] - (l > 0 ? sum[l-1] : 0);
        sum[i] = sum[i-1] + dp[i];
    }
}
```

## 易错点

1. **滚动数组忘记清空**：换层时旧值可能残留，尤其是多组数据
2. **01 背包正序**：写成正序就变成完全背包，每件物品会被重复选
3. **单调队列存值不存下标**：需要下标判断窗口是否过期，只存值无法弹出
4. **前缀和取模**：减法后要 `(x % mod + mod) % mod`，否则出现负数
5. **优化过头**：先写出 O(n²) 的正确转移，再判断能不能优化，不要一开始就套模板

---
