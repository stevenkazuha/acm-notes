# DFS 深度优先搜索

## 核心思想

一条路走到黑，走不通就**回溯**（退一步换方向）。用**栈**实现（或调用栈）。

**与 BFS 的区别**：DFS 找**所有可能解**或**一个可行解**，BFS 找**最短步数**。

## 时间复杂度

- 图的 DFS：O(n + m)，n 点数 m 边数
- 全排列：O(n · n!)
- 子集和：O(2^n)
- n 皇后：O(n!)

## 递归 DFS 模板

### 有约束的枚举

```cpp
void dfs(int state)
{
    if(到达终点/满足条件)
    {
        更新全局答案;
        return;
    }
    for(每个可能的选择 next)
    {
        if(合法条件 check(state, next))
        {
            做选择（如 vis[next] = 1, path.push_back(next)）;
            dfs(next);
            撤销选择（如 vis[next] = 0, path.pop_back()）;   // 回溯
        }
    }
}
```

### 图的 DFS（不需要回溯）

```cpp
vector<int> G[N];
bool vis[N];

void dfs(int u)
{
    vis[u] = 1;
    处理 u;
    for(int v : G[u])
        if(!vis[v]) dfs(v);
}
```

## 常见 DFS 应用

### 1. 全排列

```cpp
int n, path[10];
bool vis[10];

void dfs(int k)
{
    if(k == n)
    {
        for(int i = 0; i < n; i++) cout << path[i] << ' ';
        cout << endl;
        return;
    }
    for(int i = 1; i <= n; i++)
    {
        if(!vis[i])
        {
            vis[i] = 1;
            path[k] = i;
            dfs(k + 1);
            vis[i] = 0;
        }
    }
}

// 调用：dfs(0)
```

**去重版**（有重复元素时）：先排序，跳过相同元素且前一个没选的情况
```cpp
int a[10], n, path[10];
bool vis[10];

void dfs(int k)
{
    if(k == n) { /* 输出 path */ return; }
    for(int i = 0; i < n; i++)
    {
        if(i && a[i] == a[i-1] && !vis[i-1]) continue;   // 去重关键
        if(!vis[i])
        {
            vis[i] = 1;
            path[k] = a[i];
            dfs(k + 1);
            vis[i] = 0;
        }
    }
}
sort(a, a+n);
dfs(0);
```

### 2. n 皇后

```cpp
int n, ans;
bool col[15], dg[30], udg[30];   // 列、主对角线、副对角线

void dfs(int r)
{
    if(r == n) { ans++; return; }
    for(int c = 0; c < n; c++)
    {
        if(!col[c] && !dg[r+c] && !udg[n-1+r-c])   // 三个方向都没被占
        {
            col[c] = dg[r+c] = udg[n-1+r-c] = 1;
            dfs(r + 1);
            col[c] = dg[r+c] = udg[n-1+r-c] = 0;
        }
    }
}

// 调用：dfs(0); cout << ans;
```

**对角线编号**：
- 主对角线（左上→右下）：`r + c` 相同
- 副对角线（右上→左下）：`(n-1) + r - c` 相同（保证非负）

### 3. 子集和（选一些数凑成目标）

```cpp
int a[N], n, target;
bool found;

void dfs(int k, int sum)
{
    if(found) return;
    if(sum == target) { found = true; return; }
    if(k == n) return;
    // 选
    dfs(k + 1, sum + a[k]);
    // 不选
    dfs(k + 1, sum);
}
```

### 4. 树的 DFS（求深度、子树大小、直径）

```cpp
vector<int> G[N];
int fa[N], depth[N], sz[N];

void dfs(int u, int f)
{
    fa[u] = f;
    depth[u] = depth[f] + 1;
    sz[u] = 1;
    for(int v : G[u])
    {
        if(v == f) continue;
        dfs(v, u);
        sz[u] += sz[v];
    }
}

// 求直径（两次 DFS）
int far, max_d;
void dfs_diam(int u, int f, int d)
{
    if(d > max_d) { max_d = d; far = u; }
    for(int v : G[u])
        if(v != f) dfs_diam(v, u, d + 1);
}
// 第一次：从任意点出发找最远点 far
// 第二次：从 far 出发找最远点，距离就是直径
```

### 5.  Flood Fill（连通块染色）

```cpp
int n, m;
char g[N][N];
bool vis[N][N];
int dx[4] = {1, -1, 0, 0}, dy[4] = {0, 0, 1, -1};

int dfs(int x, int y)
{
    vis[x][y] = 1;
    int cnt = 1;
    for(int i = 0; i < 4; i++)
    {
        int nx = x + dx[i], ny = y + dy[i];
        if(nx >= 0 && nx < n && ny >= 0 && ny < m && g[nx][ny] == g[x][y] && !vis[nx][ny])
            cnt += dfs(nx, ny);
    }
    return cnt;
}

// 统计所有连通块
int block_cnt = 0, max_size = 0;
for(int i = 0; i < n; i++)
    for(int j = 0; j < m; j++)
        if(!vis[i][j])
        {
            max_size = max(max_size, dfs(i, j));
            block_cnt++;
        }
```

## 剪枝技巧

### 1. 可行性剪枝

当前状态明显不合法时提前终止：
```cpp
if(当前已超过目标值) return;
if(剩余步数不可能达到最好结果) return;
```

### 2. 最优性剪枝（可行性剪枝的进阶）

维护当前最优解 `best`，如果当前解不可能比它好就剪掉：
```cpp
if(cur_cost >= best) return;
// 或估一个下界 lower_bound，如果 cur_cost + lower_bound >= best 则剪
```

### 3. 顺序剪枝

对选择按影响排序，让好的选择先搜（更快找到可行解 + 触发剪枝）。

### 4. 迭代加深（IDA*/IDDFS）

当深度过大无法直接 DFS 时，限制深度逐层加深：
```cpp
bool dfs(int depth, int limit)
{
    if(已到达) return true;
    if(depth == limit) return false;
    for(...) if(dfs(depth+1, limit)) return true;
    return false;
}
// 调用：for(int lim = 0; lim <= 最大深度; lim++)
//       if(dfs(0, lim)) { cout << lim; break; }
```

## 非递归 DFS（手动栈）

适用于递归深度过大导致栈溢出的情况：

```cpp
stack<int> st;
bool vis[N];
st.push(start);
while(!st.empty())
{
    int u = st.top(); st.pop();
    if(vis[u]) continue;
    vis[u] = 1;
    for(int v : G[u])
        if(!vis[v]) st.push(v);
}
```

**注意**：非递归 DFS 的访问顺序和递归版不完全相同（取决于入栈顺序）。

## 例题精讲

### 例 1 求树的最大深度

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

vector<int> p;
int dfs(int u, int len)
{
    if(u == -1) return len;
    return dfs(p[u], len + 1);
}

int main()
{
    int n;
    cin >> n;
    p.assign(n, 0);
    for(int i = 0; i < n; i++)
    {
        cin >> p[i];
        if(p[i] != -1) p[i]--;
    }
    int ans = 0;
    for(int i = 0; i < n; i++)
        ans = max(ans, dfs(i, 1));
    cout << ans - 1 << endl;
    return 0;
}
```

**思路**：每个节点向上递归到根，返回深度。根节点的父是 -1 时返回当前深度。

### eg.2 棋盘放国王（互不攻击）

> n×n 棋盘放 k 个国王，每个国王周围 8 格不能有国王，求方案数。

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

int n, k;
ll ans;

void dfs(int r, int cnt, int prev)
{
    if(cnt == k) { ans++; return; }
    if(r == n) return;
    for(int mask = 0; mask < (1 << n); mask++)
    {
        if(mask & (mask << 1)) continue;   // 同一行不能相邻
        if((mask & prev) || ((mask << 1) & prev) || ((mask >> 1) & prev)) continue;
        int add = __builtin_popcount(mask);
        if(cnt + add > k) continue;
        dfs(r + 1, cnt + add, mask);
    }
}

int main()
{
    cin >> n >> k;
    dfs(0, 0, 0);
    cout << ans << endl;
    return 0;
}
```

**剪枝优化**：若 `cnt + __builtin_popcount(mask) > k`，直接跳过该行状态。

### eg.3 八数码（状态压缩 + DFS）

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

string start = "", target = "12345678x";
int dx[4] = {0, 0, 1, -1}, dy[4] = {1, -1, 0, 0};
int bound;
string path;

int h(const string& s)   // 曼哈顿距离和（启发式）
{
    int cnt = 0;
    for(int i = 0; i < 9; i++)
    {
        if(s[i] == 'x') continue;
        int j = s[i] - '1';
        cnt += abs(i/3 - j/3) + abs(i%3 - j%3);
    }
    return cnt;
}

bool dfs(int depth, int f, int x, int y, string s)
{
    if(s == target) return true;
    if(depth + f > bound) return false;   // 迭代加深边界
    for(int i = 0; i < 4; i++)
    {
        int nx = x + dx[i], ny = y + dy[i];
        if(nx < 0 || nx >= 3 || ny < 0 || ny >= 3) continue;
        string t = s;
        swap(t[x*3+y], t[nx*3+ny]);
        int nh = h(t);
        path.push_back("rldu"[i]);
        if(dfs(depth+1, nh, nx, ny, t)) return true;
        path.pop_back();
    }
    return false;
}

int main()
{
    for(int i = 0; i < 9; i++)
    {
        string c; cin >> c;
        start += c;
    }
    int sx, sy;
    for(int i = 0; i < 9; i++)
        if(start[i] == 'x') { sx = i/3; sy = i%3; }
    bound = h(start);
    while(bound <= 31)
    {
        path.clear();
        if(dfs(0, bound, sx, sy, start)) { cout << path.size() << endl; return 0; }
        bound++;
    }
    cout << -1 << endl;
    return 0;
}
```

## 易错点

1. **递归深度**：默认栈深约 1~4MB，约能支撑 10^4~10^5 层递归。n 太大要改用非递归或迭代加深
2. **回溯漏撤销**：`vis[u] = 0` 不能忘！否则后面的组合会乱
3. **剪枝方向**：有时从终点往起点搜（逆向 DFS）剪枝效率更高
4. **状态表示**：n 皇后用 `dg[r+c]` 和 `udg[n-1+r-c]` 编号，确保不越界
