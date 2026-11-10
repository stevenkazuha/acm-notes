# 区间 DP

## 核心思想

`dp[i][j]` 表示区间 `[i, j]` 的最优解。枚举分割点 `k`，将区间一分为二。

## 石子合并

```cpp
int a[N], pre[N];   // pre 前缀和
int dp[N][N];

void solve(int n)
{
    for(int len = 2; len <= n; len++)          // 区间长度
    {
        for(int i = 1; i + len - 1 <= n; i++)  // 左端点
        {
            int j = i + len - 1;
            dp[i][j] = 0x3f3f3f3f;
            for(int k = i; k < j; k++)         // 分割点
            {
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + pre[j] - pre[i-1]);
            }
        }
    }
    cout << dp[1][n] << endl;
}
```

## 矩阵链乘

```cpp
int p[N];          // 维度 p[0] x p[1], p[1] x p[2], ...
int dp[N][N];

void solve(int n)
{
    for(int len = 2; len <= n; len++)
    {
        for(int i = 1; i + len - 1 <= n; i++)
        {
            int j = i + len - 1;
            dp[i][j] = 0x3f3f3f3f;
            for(int k = i; k < j; k++)
            {
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + p[i-1]*p[k]*p[j]);
            }
        }
    }
}
```

## 真题演练

### 【2024 CCPC 网络赛 D - 编码器-解码器】

**原题**：[Codeforces Gym 105336 D](https://codeforces.com/gym/105336/problem/D)

**简述**：S 逐次编码，第 i 次编码后串 S'_i 由 S'_{i-1} + S[i] + S'_{i-1} 拼接。给定 T，求 S'_n 中 T 作为子序列出现的次数，mod 998244353。

**思路**：区间 DP。dp[i][l][r] 表示 S'_i 中匹配 T[l..r] 的子序列数。转移：空串匹配空串 = 1；S'_i = S'_{i-1} + S[i] + S'_{i-1}，分三段贡献。

**复杂度**：O(n m³)

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 105;
const int mod = 998244353;

ll dp[N][N][N];   // dp[i][l][r] = S'_i 中匹配 T[l..r] 的子序列数

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);

    string S, T;
    cin >> S >> T;
    int n = S.size(), m = T.size();
    S = ' ' + S;
    T = ' ' + T;

    // 空串边界：T[l..t-1] 为空时 = 1
    for(int i = 0; i <= n; i++)
        for(int j = 1; j <= m + 1; j++)
            for(int t = 0; t < j; t++)
                dp[i][j][t] = 1;

    for(int i = 1; i <= n; i++)
    {
        for(int l = 1; l <= m; l++)
        {
            for(int r = l; r <= m; r++)
            {
                // 主体：S'_{i-1} + S'_{i-1} 两段合并
                for(int k = l - 1; k <= r; k++)
                    dp[i][l][r] = (dp[i][l][r] + dp[i-1][l][k] * dp[i-1][k+1][r]) % mod;
                // 中间 S[i] 刚好匹配 T[k+1] 时多一份贡献
                for(int k = l - 1; k < r; k++)
                    if(S[i] == T[k+1])
                        dp[i][l][r] = (dp[i][l][r] + dp[i-1][l][k] * dp[i-1][k+2][r]) % mod;
            }
        }
    }
    cout << dp[n][1][m] << '\n';
    return 0;
}
```

**区间 DP 为什么能解**：T 的子序列匹配只关心 T 的一个区间 [l, r]。S'_i 由 S'_{i-1} 拼接而成，所以可以从两侧的 [l,k] 和 [k+1,r] 合并得到 [l,r]，中间插入 S[i] 时额外检查是否等于 T 的某一位。

---
