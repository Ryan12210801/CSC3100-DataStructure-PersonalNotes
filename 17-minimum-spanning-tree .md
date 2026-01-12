# MST（ Minimum  Spanning Tree）

# MST=无向图全连通+无环+边权和最小

## 一、基本概念

### 1. Spanning Tree

给定一个**连通无向图** G=(V,E)G=(V,E)G=(V,E)，生成树满足：

- 覆盖所有顶点
- 连通
- 无环
- 边数 = ∣V∣−1|V|-1∣V∣−1

---

### 2. Minimum Spanning Tree（ MST ）

在**带权无向连通图**中：

> 所有生成树里，边权和最小的一棵
> 

重要性质：

- MST **不一定唯一**
- MST **一定无环**
- 若所有边权互不相同 ⇒ MST 唯一

## 二、MST 的理论基础（Cut Property）

### 1. Cut（割）

    **(S, V-S)**

- 将顶点集划分为两个不相交部分

---

![image.png](assests/MST.png)

### 2. Light Edge（轻边）

- 在所有 **跨越该割** 的边中
- 权重最小的那条

---

### 3. Cut Property（核心定理）

> 设：
> 
> - A 是某个 MST 的子集
> - (S,V−S)是一个 关于**A** 的cut
> - ( u, v ）是该**cut**的轻边
> 
> 则：
> 
> ( u, v ) 对 A 是 **安全边 （safe）** 
> 

**Prim 和 Kruskal 算法正确性的唯一理论来源**

## 三、Prim 算法（点驱动，一棵树）

从任意起点开始，维护一个已经在生成树中的顶点集合。对每个未加入的顶点，记录它通过哪一条边可以**以最小代价**连到当前生成树。每一步都选择那个**接入代价最小的顶点**加入生成树，并用它的边去更新其他顶点的接入代价。重复直到所有顶点都被加入，得到的边集就是最小生成树。

### 思想

- 维护一个已在 MST 中的点集 V_A
- 每次选：
    - 连接 V_A 与外部顶点的 **最小权边**

---

### 关键变量设计（非常重要）

- `inMST[v]`：点 v 是否已加入 MST
- `key[v]`：v 到当前 MST 的最小边权
- `parent[v]`：MST 中 v 的父节点

### java代码模板

图结构

```java
static class Edge {
    int to;     // 这条边连向哪个点
    int w;      // 边权
    Edge(int to, int w) {
        this.to = to;
        this.w = w;
    }
}
```

prim算法

```java
static int prim(List<Edge>[] g, int r) {
    int n = g.length;

    // Q：最小优先队列，按 key 值排序
    PriorityQueue<int[]> Q =
        new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    // a[0] = key, a[1] = vertex

    int[] key = new int[n];        // key[u]
    int[] parent = new int[n];     // π[u]
    boolean[] inQ = new boolean[n]; // v ∈ Q ?

    /* ---------------------------------
     * for each u ∈ V
     *   key[u] ← ∞
     *   π[u] ← NIL
     *   INSERT(Q, u)
     * --------------------------------- */
    for (int u = 0; u < n; u++) {
        key[u] = Integer.MAX_VALUE;
        parent[u] = -1;
        inQ[u] = true;
    }

    /* DECREASE-KEY(Q, r, 0) */
    key[r] = 0;
    Q.add(new int[]{0, r});

    int totalWeight = 0;

    /* while Q ≠ ∅ */
    while (!Q.isEmpty()) {

        /* u ← EXTRACT-MIN(Q) */
        int[] cur = Q.poll();
        int u = cur[1];

        // 关键：跳过旧的 / 已处理过的条目
        if (!inQ[u]) continue;

        inQ[u] = false;       // u ∉ Q
        totalWeight += cur[0];

        /* for each v ∈ Adj[u] */
        for (Edge e : g[u]) {
            int v = e.to;
            int w = e.w;

            /* if v ∈ Q and w(u,v) < key[v] */
            if (inQ[v] && w < key[v]) {
                parent[v] = u;    // π[v] ← u
                key[v] = w;

                /* DECREASE-KEY(Q, v, w(u,v)) */
                Q.add(new int[]{key[v], v});
            }
        }
    }

    return totalWeight;
}

```

**时间复杂度（ElogV）**

## 四、Kruskal 算法（边驱动，多棵树）

### 思想

- 初始：每个顶点都是一个独立集合
- 按权重从小到大尝试加边
- **只要不成环就加（即之前不在同一个集合中）**
- Kruskal 的效率**取决于**如何实现这两件事：
1.  **Find**：判断两个点是否在同一集合
2. **Union**：把两个集合合并

### 实现方法一：Naive Set Union（显式集合，最原始）

**思想**

- `setArray[v]` 直接存 **v 所在的整个集合**
- 合并时：
    - 新建 `newSet = setU ∪ setV`
    - 对 `newSet` 中的每个顶点，重写 `setArray[w]`

**伪代码特征**

```
newSet = setU ∪ setV
for each w in newSet:
    setArray[w] = newSet
```

**特点**

- Find：O(1)（直接比较 set 引用）
- Union：O(|setU| + |setV|)（全量重写）

**时间复杂度**

- 最坏情况下，每次合并都要改 O(V) 个点
- 外层还要扫描 E 条边
- O（VE）

## 实现方式二：Label + Small-to-Large

### 思想

- `label[v]` 表示 v 当前所属集合编号
- `setArray[id]` 存该集合的所有元素
- 合并时：
    - **总是把小集合合并进大集合**
    - 只修改小集合中顶点的 label

### 关键优化

```
if size(setU) >= size(setV):
    把 setV 并入 setU
else:
    把 setU 并入 setV
```

### 核心结论（非常重要）

> 每个顶点的 label 被修改的次数 ≤ O(log V)
> 

因为：

- 每次被合并，所在集合至少翻倍
- 翻倍次数最多 log V 次

### 时间复杂度

- 排序边：O(Elog⁡E)
- 合并代价总和：O(Vlog⁡V)

$$
{O(E \log E + V \log V) \approx O(E \log E)}
$$

## 实现方式三：Union-Find（父指针 + 路径压缩）

### 思想

- 每个集合用一棵树表示
- `parent[x]` 指向父节点
- `find(x)` 找根节点
- `union(x,y)` 合并两棵树

### 关键优化

- **Path Compression**
- **Union by Rank / Size**

### 时间复杂度

- 单次操作：近似 O(1)
- 总体：O(Elog⁡E)  排序仍是瓶颈

### 具体代码

**Kruskal**

```java
    // Kruskal 用的边结构：保存 u-v-w（便于排序）
    static class E {
        int u, v, w;
        E(int u, int v, int w) {
            this.u = u;
            this.v = v;
            this.w = w;
        }
    }

    // Kruskal: 输入边集 + n，返回 MST 总权重
    static long kruskalMST(List<E> edges, int n) {
        // 1) 按权重从小到大排序（Kruskal 的第一核心）
        edges.sort(Comparator.comparingInt(e -> e.w));

        // 2) 初始化并查集：每个点各自为集合（森林的初始状态）
        DSU dsu = new DSU(n);

        long total = 0;     // MST 总权重
        int picked = 0;     // 已选边数（MST 最终应为 n-1 条）

        // 3) 依次尝试加入最小边
        for (E e : edges) {
            // 若 u 和 v 不在同一连通分量，加这条边不会形成环 => 可以加入 MST
            if (dsu.union(e.u, e.v)) {
                total += e.w;
                picked++;

                // MST 一旦选够 n-1 条边就结束
                if (picked == n - 1) break;
            }
            // 若 union 失败，说明 u v 已连通，再加会成环 => 丢弃
        }

        return total;
    }

```

**DSU**

```java
    // 并查集（DSU / Union-Find）
    // 支持：
    // - find(x): 找到 x 所在集合的代表（root）
    // - union(a,b): 合并两个集合，若本来不同则返回 true，否则 false
    static class DSU {
        int[] parent;
        int[] rank;  // rank 可理解为“近似高度”，用于按秩合并

        DSU(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }

        // 路径压缩：把 x 直接挂到 root 上，加速后续查询
        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }

        // 按秩合并：把“矮”的树挂到“高”的树上，避免退化
        boolean union(int a, int b) {
            int ra = find(a);
            int rb = find(b);
            if (ra == rb) return false; // 已同集合：再连会成环（对 Kruskal 来说就是丢弃边）

            if (rank[ra] < rank[rb]) {
                parent[ra] = rb;
            } else if (rank[ra] > rank[rb]) {
                parent[rb] = ra;
            } else {
                parent[rb] = ra;
                rank[ra]++; // 两棵同高树合并，高度+1
            }
            return true;
        }
    }

```
