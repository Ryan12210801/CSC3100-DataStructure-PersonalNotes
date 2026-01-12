# Graph Shortest Path (2) (All-Pair)

## 1. 问题定义（Problem Definition）

目标：

计算图中**任意两点之间的最短路径距离**（All-Pairs Shortest Paths）。

输入表示：权重矩阵 W

- W(i,i)=0
- W(i,j)=weight若存在边 (i,j)
- W(i,j)=∞，若不存在边

---

### 朴素方法对比

方法：

对每个顶点运行一次 Dijkstra。

时间复杂度：

$$
O(|V|\cdot|E|\log|V|)
$$

局限：

- 在稠密图中效率不佳
    
    $$
    |E|\approx |V|^2
    $$
    
- Floyd 更适合矩阵表示的稠密图

## 2. Floyd 算法核心思想（Core Idea）

Floyd 的本质是 **动态规划（Dynamic Programming）**。

核心思想：通过逐步扩大“允许作为中间点的顶点集合”，来更新最短路径。

---

### 动态规划状态定义

$$
D^{(k)}[i,j]
$$

含义：

从顶点vi到 vj的最短路径，其中**中间节点只能来自集合**

$$
\{v_1, v_2, \dots, v_k\}
$$

---

### 状态转移方程

$$
D^{(k)}[i,j]
=
\min\Big(
D^{(k-1)}[i,j],
D^{(k-1)}[i,k] + D^{(k-1)}[k,j]
\Big)
$$

解释：

- 不经过 vk：距离保持不变
- 经过 vk：路径拆成 i→k 和 k→j

## 3. 实现方式与空间优化

### 实现版本演进

1. 保存所有 D0,D1,…,Dn：空间 O(n^3)
2. 只保存前后两层：空间 O(n^2)
3. 原地更新（In-place）：O(n^2)

---

### 原地更新为何可行

在第 k 轮更新中：

- 主对角线：D[k][k]=0
- 第 k 行：

$$
D[k][j]^{(k)}=min(D^{(k-1)}[k][j],D^{(k-1)}[k][k]+D^{(k-1)}[k][j])=D^{(k-1)}[k][j]
$$

- 第 k 列同理

结论：

本轮计算所需的 D[i][k]和 D[k][j] 不会被破坏，因此可以安全原地更新。

## 4. Floyd 核心代码（Java）

### 距离矩阵版本

```java
static final long INF = (long) 1e18;

static long[][] floyd(long[][] W) {
    int n = W.length;
    long[][] D = new long[n][n];

    // 初始化
    for (int i = 0; i < n; i++)
        System.arraycopy(W[i], 0, D[i], 0, n);

    // Floyd 主循环
    for (int k = 0; k < n; k++) {      // 中间点必须在最外层
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (D[i][k] + D[k][j] < D[i][j]) {
                    D[i][j] = D[i][k] + D[k][j];
                }
            }
        }
    }
    return D;
}
```

## 5. 路径重构（Path Reconstruction）

### P 矩阵含义

- P[i][j]=k
    
    从 i到 j的最短路径**经过中间点 k**
    

---

### 递归打印路径

```java
static void printPath(int i, int j) {
    if (P[i][j] == -1) return;
    int k = P[i][j];
    printPath(i, k);
    System.out.print(" -> " + k);
    printPath(k, j);
}

```

## 6. 复杂度分析

- 时间复杂度：
    
    O(|V|^3)
    
- 空间复杂度：
    
    O(|V|^2)
    

## 7. 易错点

1. 循环顺序
    
    中间点 k **必须在最外层循环**，否则动态规划语义错误。
    
2. 初始化
    - 对角线必须为 0
    - 不连通边必须设为 ∞
3. 适用范围
    - 适用于稠密图
    - 允许负权边
    - 不能存在负权环