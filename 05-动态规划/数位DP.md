# 数位 DP

## 核心思想

按位枚举数字，用"前缀 + 记忆化"统计区间 [l, r] 内满足条件的数。

## 模板

```cpp
using ll = long long;
string num;
long long dp[20][20];     // dp[pos][state]，状态范围须按题目调整
int a[20];

long long dfs(int pos, int state, bool limit)
{
    if(pos < 0) return 1;          // 找到一个合法数
    if(!limit && dp[pos][state] != -1) return dp[pos][state];
    int up = limit ? a[pos] : 9;
    long long res = 0;
    for(int i = 0; i <= up; i++)
    {
        if(check(state, i))        // 根据题目条件过滤
        {
            res += dfs(pos-1, new_state, limit && (i == up));
        }
    }
    if(!limit) dp[pos][state] = res;
    return res;
}

long long solve(ll x)
{
    if(x < 0) return 0;
    int n = 0;
    while(x) { a[n++] = x % 10; x /= 10; }
    memset(dp, -1, sizeof(dp));
    return dfs(n-1, 0, true);
}

int main()
{
    ll l, r;
    cin >> l >> r;
    cout << solve(r) - solve(l-1) << endl;
    return 0;
}
```

## 经典问题：统计 1 的个数

```cpp
long long dfs(int pos, int cnt, bool limit)
{
    if(pos < 0) return cnt;
    if(!limit && dp[pos][cnt] != -1) return dp[pos][cnt];
    int up = limit ? a[pos] : 9;
    long long res = 0;
    for(int i = 0; i <= up; i++)
    {
        res += dfs(pos-1, cnt + (i==1), limit && (i==up));
    }
    if(!limit) dp[pos][cnt] = res;
    return res;
}
```
