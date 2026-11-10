# 状态压缩 DP

## 核心思想

用一个整数的二进制位表示"选/不选"的状态。

## 旅行商问题 TSP

```cpp
int n;
int dist[N][N];
int dp[1<<N][N];   // dp[mask][i] 表示走过 mask 状态，最后在 i 的最短距离

void solve()
{
    memset(dp, 0x3f, sizeof(dp));
    dp[1][0] = 0;                  // 从 0 出发
    for(int mask = 1; mask < (1<<n); mask++)
    {
        for(int i = 0; i < n; i++)
        {
            if(!(mask & (1<<i))) continue;
            for(int j = 0; j < n; j++)
            {
                if(mask & (1<<j)) continue;
                int nmask = mask | (1<<j);
                dp[nmask][j] = min(dp[nmask][j], dp[mask][i] + dist[i][j]);
            }
        }
    }
    int ans = n == 1 ? 0 : 0x3f3f3f3f;
    for(int i = 1; i < n; i++)
    {
        ans = min(ans, dp[(1<<n)-1][i] + dist[i][0]);
    }
}
```

## 棋盘放置（互不攻击）

```cpp
// 每行放一个，不能同行同列同对角线冲突
int n, ans;

void dfs(int r, int col, int dg, int udg)
{
    if(r == n) { ans++; return; }
    int full = (1 << n) - 1;
    int avail = full & ~(col | dg | udg);
    while(avail)
    {
        int p = avail & -avail;
        avail ^= p;
        dfs(r + 1, col | p, ((dg | p) << 1) & full, (udg | p) >> 1);
    }
}

void solve()
{
    ans = 0;
    dfs(0, 0, 0, 0);
}
```
