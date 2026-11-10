# 树形 DP

## 核心思想

DP 在树上进行，状态挂在节点上。因为树没有环，从根 DFS 一遍，回溯时把儿子信息合并到父亲，天然满足后序依赖。

标准流程：

1. 任选一个点当根，DFS
2. 先递归所有儿子，再合并
3. 答案在根节点，或对所有节点取 max

适合的题型：最大独立集、树上选点、树的直径、树上背包。

## 模板：树的最大独立集

没有上司的舞会：每个点有权值，选了父亲就不能选儿子，求最大权值和。

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N = 100005;
vector<int> g[N];
int w[N], dp[N][2];

void dfs(int u, int fa)
{
    dp[u][1] = w[u];
    dp[u][0] = 0;
    for(int v : g[u])
    {
        if(v == fa) continue;
        dfs(v, u);
        dp[u][1] += dp[v][0];
        dp[u][0] += max(dp[v][0], dp[v][1]);
    }
}

int main()
{
    int n;
    cin >> n;
    for(int i = 1; i <= n; i++) cin >> w[i];
    for(int i = 1; i < n; i++)
    {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs(1, 0);
    cout << max(dp[1][0], dp[1][1]) << '\n';
    return 0;
}
```

`dp[u][1]` 表示选 u，`dp[u][0]` 表示不选 u。选 u 时儿子只能不选；不选 u 时儿子选不选都行。

## 树的直径（树形 DP 写法）

维护每个点向下的最长链和次长链，两者相加就是经过该点的最长路径。

```cpp
int down[N], ans;

void dfs(int u, int fa)
{
    int mx1 = 0, mx2 = 0;
    for(int v : g[u])
    {
        if(v == fa) continue;
        dfs(v, u);
        int d = down[v] + 1;
        if(d > mx1)
        {
            mx2 = mx1;
            mx1 = d;
        }
        else if(d > mx2) mx2 = d;
    }
    down[u] = mx1;
    ans = max(ans, mx1 + mx2);
}
```

复杂度 O(n)。两遍 DFS 的写法也可以求直径，但树形 DP 不用存距离数组，换根类题目也更容易改。

## 易错点

1. **父节点判重**：DFS 一定要带 `fa`，否则在无向树上会死循环
2. **叶子初始化**：选/不选两种状态的初值要在进入儿子之前设好
3. **合并顺序**：先 `dfs(v, u)` 再合并，不能先合并后递归
4. **递归深度**：链状树深度可达 1e5，必要时手写栈或开大栈空间
5. **根的选择**：普通树形 DP 任选根；换根 DP 要先求一次再二次扫描

---
