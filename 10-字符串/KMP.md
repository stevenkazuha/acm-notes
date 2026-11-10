# KMP 字符串匹配

## 核心思想

匹配失败时，根据已匹配前缀的"失配函数"跳过不必要的比较。

## 模板

```cpp
char s[N], p[N];     // 文本串、模式串
int n, m;
int ne[N];           // next 数组

void get_next()
{
    ne[1] = 0;
    int j = 0;
    for(int i = 2; i <= m; i++)
    {
        while(j && p[i] != p[j+1]) j = ne[j];
        if(p[i] == p[j+1]) j++;
        ne[i] = j;
    }
}

vector<int> kmp()
{
    get_next();
    vector<int> res;
    int j = 0;
    for(int i = 1; i <= n; i++)
    {
        while(j && s[i] != p[j+1]) j = ne[j];
        if(s[i] == p[j+1]) j++;
        if(j == m)
        {
            res.push_back(i - m + 1);   // 匹配起点
            j = ne[j];
        }
    }
    return res;
}
```

## 例题

### 例：所有前缀的 border 长度和

**题意**：给定字符串 S，对每个前缀求最长相等真前缀、真后缀的长度，再求这些长度的总和。

**思路**：KMP 的 `ne[i]` 就是前缀 `S[1..i]` 的最长 border 长度，求出 next 数组后直接累加。

**复杂度**：O(n)

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

const int N = 200005;
char s[N];
int ne[N];

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);

    int n;
    cin >> n >> (s + 1);

    int j = 0;
    for(int i = 2; i <= n; i++)
    {
        while(j && s[i] != s[j+1]) j = ne[j];
        if(s[i] == s[j+1]) j++;
        ne[i] = j;
    }

    ll ans = 0;
    for(int i = 1; i <= n; i++) ans += ne[i];
    cout << ans << '\n';
    return 0;
}
```

**要点**：一个前缀的所有 border 可以通过 `ne[len], ne[ne[len]], ...` 依次得到，这是统计 border 出现次数等问题的基础。

---
