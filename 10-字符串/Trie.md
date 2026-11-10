# Trie 字典树

## 核心思想

每个节点代表一个前缀，边代表字符。查询一个串是否存在，复杂度只和串长有关，和集合大小无关。

常见用途：前缀查询、单词检索、01-Trie 求最大异或。

## 复杂度

- 插入 / 查询：O(L)，L 为串长
- 空间：O(总字符数 × 字符集大小)，26 个字母时按节点数 × 26 开

## 模板（小写字母）

```cpp
int cnt;
int ch[300001][26];   // 总字符数 + 根节点
bool is_end[300001];

void insert(string s)
{
    int p = 0;
    for(char c : s)
    {
        int idx = c - 'a';
        if(!ch[p][idx]) ch[p][idx] = ++cnt;
        p = ch[p][idx];
    }
    is_end[p] = 1;
}

bool query(string s)
{
    int p = 0;
    for(char c : s)
    {
        int idx = c - 'a';
        if(!ch[p][idx]) return false;
        p = ch[p][idx];
    }
    return is_end[p];
}

bool starts_with(string prefix)
{
    int p = 0;
    for(char c : prefix)
    {
        int idx = c - 'a';
        if(!ch[p][idx]) return false;
        p = ch[p][idx];
    }
    return true;
}
```

## 01-Trie（求最大异或）

把数字按二进制从高位到低位插入。查询时尽量走相反位，就能得到最大异或值。

```cpp
const int N = 100005;
int ch[N * 31][2], cnt;

void insert(int x)
{
    int p = 0;
    for(int i = 30; i >= 0; i--)
    {
        int b = (x >> i) & 1;
        if(!ch[p][b]) ch[p][b] = ++cnt;
        p = ch[p][b];
    }
}

// 返回与 x 异或的最大值
int query(int x)
{
    int p = 0, res = 0;
    for(int i = 30; i >= 0; i--)
    {
        int b = (x >> i) & 1;
        if(ch[p][b ^ 1])
        {
            res |= (1 << i);
            p = ch[p][b ^ 1];
        }
        else p = ch[p][b];
    }
    return res;
}
```

**经典模型**：求数组中两个数的最大异或。把所有数插入 01-Trie，再对每个数查询一次，取最大值。

## 易错点

1. **根节点编号**：根固定为 0，新节点从 ++cnt 开始，不要从 1 开始再冲突
2. **is_end 与前缀**：只查前缀不需要 `is_end`，查完整单词必须判断
3. **多组数据清空**：`ch`、`cnt`、`is_end` 都要重置，否则上一组数据串进来
4. **空间开够**：节点上限是总字符数，01-Trie 是 n × 31
5. **字符集映射**：大小写、数字混出现时，映射函数要统一

---
