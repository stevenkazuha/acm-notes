# 树链剖分与 LCA

## 最近公共祖先 LCA

### 倍增法

```cpp
vector<int> G[N];
int fa[N][20], depth[N];
const int LOG = 20;   // 与第二维一致；最大深度达到 2^20 时同步增大

void dfs(int u, int f)
{
    fa[u][0] = f;
    depth[u] = depth[f] + 1;
    for(int i = 1; i < LOG; i++)
        fa[u][i] = fa[fa[u][i-1]][i-1];
    for(int v : G[u])
        if(v != f) dfs(v, u);
}

int lca(int a, int b)
{
    if(depth[a] < depth[b]) swap(a, b);
    // 把 a 抬到 b 的深度
    for(int i = LOG-1; i >= 0; i--)
        if(depth[fa[a][i]] >= depth[b]) a = fa[a][i];
    if(a == b) return a;
    for(int i = LOG-1; i >= 0; i--)
    {
        if(fa[a][i] != fa[b][i])
        {
            a = fa[a][i];
            b = fa[b][i];
        }
    }
    return fa[a][0];
}
```

## 树链剖分（重链剖分）

```cpp
vector<int> G[N];
int fa[N], depth[N], heavy[N], size[N];   // 重儿子
int top[N], id[N], w[N], new_w[N], dfs_cnt;

int dfs1(int u, int f)
{
    fa[u] = f;
    depth[u] = depth[f] + 1;
    size[u] = 1;
    heavy[u] = 0;   // 多测重建树时清除上一棵树的重儿子
    int mx = 0;
    for(int v : G[u])
    {
        if(v == f) continue;
        size[u] += dfs1(v, u);
        if(size[v] > mx) { mx = size[v]; heavy[u] = v; }
    }
    return size[u];
}

void dfs2(int u, int t)
{
    top[u] = t;
    id[u] = ++dfs_cnt;
    new_w[dfs_cnt] = w[u];
    if(!heavy[u]) return;
    dfs2(heavy[u], t);       // 重链延伸
    for(int v : G[u])
        if(v != fa[u] && v != heavy[u]) dfs2(v, v);   // 轻链另起
}

void build_hld()
{
    dfs_cnt = 0;
    dfs1(1, 0);
    dfs2(1, 1);
}

// 路径查询 u-v：拆成重链段
int query_path(int u, int v)
{
    int ans = 0;
    while(top[u] != top[v])
    {
        if(depth[top[u]] < depth[top[v]]) swap(u, v);
        ans += seg_query(id[top[u]], id[u]);
        u = fa[top[u]];
    }
    if(depth[u] > depth[v]) swap(u, v);
    ans += seg_query(id[u], id[v]);
    return ans;
}
```
