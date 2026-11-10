# STL 容器

## 复杂度一览

| 容器 | 头部 | 尾部 | 中间 | 查找 | 备注 |
|------|------|------|------|------|------|
| vector | O(n) | **O(1) 均摊** | O(n) | O(n) | 动态数组 |
| deque | **O(1)** | **O(1)** | O(n) | O(n) | 双端队列 |
| list | O(1)* | O(1)* | O(1)* | O(n) | 双向链表（少用） |
| stack | — | O(1) | — | — | 适配器 |
| queue | — | O(1) | — | — | 适配器 |
| priority_queue | — | — | — | O(log n) | 堆 |
| set/map | O(log n) | O(log n) | O(log n) | **O(log n)** | 红黑树 |
| unordered_set/map | O(1)* | O(1)* | O(1)* | **O(1)* 均摊** | 哈希表 |

> * 均摊复杂度；中间插入 O(1) 前提是已拿到迭代器。

## 1. vector 动态数组

### 创建

```cpp
vector<int> v;                              // 空
vector<int> v2{1, 2, 3, 4, 5};              // 初始化列表
vector<int> v3(5);                          // 5 个 0
vector<int> v4(5, 10);                      // 5 个 10
vector<int> v5(v4);                         // 拷贝
vector<int> v6(5, 0);                       // 二维的一维先开好
vector<vector<int>> g(n+1);                 // 邻接表常用写法
vector<vector<int>> a(n+1, vector<int>(m+1, 0));  // 二维矩阵
v.assign(n, x);                              // 重新赋值
v.reserve(1000);                             // 预分配容量（减少 vector 重新分配；rehash 是哈希容器的操作）
```

### 增删改查

```cpp
v.push_back(x);                              // 尾部加 O(1) 均摊
v.pop_back();                                // 尾部删 O(1)
v.front();                                   // 首元素
v.back();                                    // 尾元素
v[i];                                        // 随机访问 O(1)
v.insert(v.begin()+2, x);                    // 任意位置插入 O(n)
v.insert(v.begin()+2, 4, 10);               // 插入 4 个 10
v.erase(v.begin()+2);                        // 删除 O(n)
v.erase(v.begin(), v.begin()+4);             // 删除区间 [l, r)
v.clear();                                   // 清空
v.resize(n);                                 // 改大小
v.size();                                    // 元素个数
v.empty();                                   // 是否为空
```

### 遍历

```cpp
// 最常用
for(int x : v) cout << x;
// 带下标
for(int i = 0; i < v.size(); i++) cout << v[i];
// 迭代器
for(auto it = v.begin(); it != v.end(); it++) cout << *it;
// 倒序
for(auto it = v.rbegin(); it != v.rend(); it++) cout << *it;
```

### 排序

```cpp
sort(v.begin(), v.end());                  // 升序
sort(v.begin(), v.end(), greater<int>()); // 降序
sort(v.rbegin(), v.rend());                // 降序（反转迭代器）
```

## 2. stack 栈（后进先出 LIFO）

```cpp
stack<int> s;
stack<string> ss;
stack<pair<int,int>> sp;

s.push(x);    // 入栈
s.pop();      // 出栈（无返回值）
s.top();      // 栈顶
s.empty();
s.size();
```

**应用场景**：表达式求值、括号匹配、单调栈、DFS 辅助。

## 3. queue 队列（先进先出 FIFO）

```cpp
queue<int> q;
q.push(x);   // 队尾入队
q.pop();     // 队首出队
q.front();   // 队首
q.back();    // 队尾
```

**应用场景**：BFS、任务调度、滑动窗口。

## 4. deque 双端队列

支持两端 O(1) 插入删除，兼具 vector 和 queue 的功能。

```cpp
deque<int> dq;
dq.push_back(x);    dq.push_front(x);
dq.pop_back();      dq.pop_front();
dq.back();          dq.front();
dq.erase(it);       dq.erase(first, last);
dq[n];              // 随机访问
```

**应用场景**：滑动窗口、单调队列优化 DP。

## 5. priority_queue 优先队列（堆）

### 默认大根堆

```cpp
priority_queue<int> pq;              // 最大元素在顶
pq.push(x);
pq.pop();
pq.top();
```

### 小根堆

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

### 自定义比较

```cpp
struct Node { int x, y; };

struct cmp
{
    bool operator()(const Node& a, const Node& b)
    {
        return a.x > b.x;
    }
};

priority_queue<Node, vector<Node>, cmp> pq;
```

或用 `operator<`：

```cpp
struct Node
{
    int x, y;
    bool operator<(const Node& o) const { return x > o.x; }  // 反号 = 小根堆
};
priority_queue<Node> pq;
```

**复杂度**：push/pop O(log n)，top O(1)。

## 6. set 集合（红黑树，自动去重+升序）

```cpp
set<int> s;
s.insert(x);        // O(log n)
s.erase(x);         // 按值删 O(log n)
s.erase(it);        // 按迭代器删 O(1)
s.find(x);          // 查找，返回迭代器或 end
s.count(x);         // 1 存在 / 0 不存在
s.lower_bound(x);   // 第一个 >= x
s.upper_bound(x);   // 第一个 > x
*(s.begin());       // 最小值
*(s.rbegin());      // 最大值
```

**遍历时删元素**：
```cpp
for(auto it = s.begin(); it != s.end(); )
{
    if(cond) it = s.erase(it);  // erase 返回下一个迭代器
    else it++;
}
```

**应用场景**：贪心调度（按值删）、窗口去重。

## 7. multiset（可重复）

```cpp
multiset<int> ms;
ms.insert(x);       // 可插多个
ms.erase(x);        // 删所有等于 x 的
auto it = ms.find(x);
if(it != ms.end()) ms.erase(it); // 存在时只删一个，不能 erase(end())
```

## 8. map / unordered_map

```cpp
map<int, string> mp;              // 红黑树，按键升序，O(log n)
unordered_map<int, string> um;    // 哈希，O(1) 均摊

mp[key] = value;                  // 插入或覆盖
mp.insert({key, value});          // 插入（不覆盖已存在的）
mp.find(key);                     // 返回迭代器
mp.count(key);                    // 1/0
mp.erase(key);                    // 删

// 遍历
for(auto& [k, v] : mp) cout << k << v;    // C++17 结构化绑定
for(auto& p : mp) cout << p.first << p.second;
```

**易错点**：`mp[key]` 在 key 不存在时会自动创建一个默认值（int 是 0，string 是空串）。如果只想查询是否存在，用 `find` 或 `count`。

## 9. bitset（位集合）

单个位访问为 O(1)；对 B 位整体进行位运算通常需要 O(⌈B / w⌉) 个机器字操作，w 为机器字长。

```cpp
bitset<100> b;                      // 固定大小，100 位
b.set(3);                            // 第 3 位设 1
b.reset(3);                          // 设 0
b[3] = 1;                            // 直接访问
b.test(3);                           // 查第 3 位
b.count();                           // 1 的个数
b.size();
b.reset(); b.set();                  // 全部 0 / 1
// 支持 & | ^ >> << ~ 位运算
bitset<100> c = b1 & b2;
```

**应用场景**：状态压缩（比 int 的 32/64 位宽）、bitset 优化 DP。

## 10. pair / tuple

```cpp
pair<int, int> p = {1, 2};
p.first;    p.second;
// 比较：先 first 再 second

tuple<int, int, int> t = {1, 2, 3};
get<0>(t);   // 取第 0 个
```

## 通用技巧

### 反向迭代器（降序遍历）

```cpp
for(auto it = v.rbegin(); it != v.rend(); it++) cout << *it;
sort(v.rbegin(), v.rend());  // 降序
```

### next_permutation（枚举全排列）

```cpp
int a[] = {1, 2, 3};
sort(a, a+3);                       // 必须先排序
do {
    for(int i = 0; i < 3; i++) cout << a[i];
    cout << endl;
} while(next_permutation(a, a+3));   // O(n·n!)
```

### min/max 的细节

```cpp
min(a, b);            // 返回值（C++11 起可接受初始化列表）
min({a, b, c, d});    // 多个值
min_element(a, a+n);  // 返回迭代器
```

### 不可复制元素的容器

```cpp
// 若使用 unique_ptr、mutex 等不可复制类型
vector<unique_ptr<int>> v;
v.push_back(make_unique<int>(5));       // 只能 move
v.emplace_back(new int(5));              // emplace 直接构造
```
