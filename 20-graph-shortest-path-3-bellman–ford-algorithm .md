# Graph Shortest Path (3)(Bellman–Ford Algorithm)

## Bellman–Ford = 全边松弛 × (|V|−1) + 再检查一轮

## 1. 问题背景与挑战（Why Bellman–Ford）

### 1.1 负权边带来的问题

- **Dijkstra 的前提**：边权非负
- **失败原因**：
    
    Dijkstra 使用贪心思想，一旦“选定”某个点，其最短距离就不再改变
    
    → **负权边可能让“已确定”的点被后续路径再次缩短**
    

### 1.2 负权环（Negative-weight Cycle）

- 定义：一个环，其边权和 < 0
- 后果：
    - 若源点 `s` 能到达该环
    - 可无限次绕环 → 路径长度 → `∞`
- 结论：
    - **最短路径“不存在”**
    - 算法必须能**检测负环**
    
    ## 2. Bellman–Ford 核心思想（Core Idea）
    
    ### 2.1 路径长度上界
    
    - 在**无负环的最短路径**中：
        - 任意简单路径最多包含 `|V| - 1` 条边
    - 因为：
        - 超过 `|V| - 1` 条边 ⇒ 必然有重复点 ⇒ 构成环
        - 若是最短路径，不会绕“正环 / 零环”
    
    ---
    
    ### 2.2 松弛（Relaxation）的含义
    
    对一条边 `(u, v, w)`：
    
    ```java
    if d[v] > d[u] + w {
       d[v] = d[u] + w //update
    }
    
    ```
    
    **解释**：
    
    > “我是否找到了一条经过 u 再到 v的更短路径？
    > 
    
    ### 2.3 算法整体流程
    
    1. 初始化
        
        ```
        d[s] = 0
        其他点 = +∞
        ```
        
    2. 重复 `|V| - 1` 轮
        - 每一轮：**遍历所有边并尝试松弛**
    3. 再额外做 1 轮
        - 若仍可松弛 ⇒ **存在负权环**
    
    ---
    
    ## 3. 负环检测机制（Why It Works）
    
    - 在第 `|V| - 1` 轮后：
        - 所有**合法最短路径**（最多 `|V|-1` 条边）已被考虑
    - 若第 `|V|` 轮仍能更新：
        - 意味着某条路径可以**无限缩短**
        - 只能来自 **负权环**
    
    ---
    
    ## 4. 正确性检验（偏直觉）
    
    ### 4.1 上界性质（Upper-bound Property）
    
    - 任意时刻：
        
        ```
        d[v] ≥ δ(s, v)
        ```
        
    - 原因：
        - 初始为 +∞
        - 每次更新来自某条真实路径
        - 从不“低估”最短距离
    
    ---
    
    ### 4.2 收敛性质（Convergence）
    
    - 若最短路径为：
        
        ```
        s → ... → u → v
        ```
        
    - 当 `d[u]` 正确后：
        - 松弛 `(u, v)` ⇒ `d[v]` 正确
    
    ---
    
    ### 4.3 为什么要 |V|−1 轮？
    
    - 第 1 轮：正确处理 1 条边的最短路径
    - 第 2 轮：2 条边
    - ...
    - 第 `k` 轮：k 条边
    - 最长合法最短路径 ≤ `|V|-1`
    
    ---
    
    ### 4.4 有负环时的反证逻辑
    
    - 若存在负环却算法仍“稳定”
    - 对环上所有松弛不等式求和：
        
        ```
        d[v] ≤ d[v] + (负数)
        ```
        
        ## 5. Java 参考实现（清晰、可直接用）
        
        ### 5.1 边结构
        
        ```java
        static class Edge {
            int from, to;
            int w;
            Edge(int f, int t, int w) {
                this.from = f;
                this.to = t;
                this.w = w;
            }
        }
        ```
        
        ---
        
        ### 5.2 Bellman–Ford 主体
        
        ```java
        static long[] bellmanFord(int n, List<Edge> edges, int s) {
            long INF = Long.MAX_VALUE / 4;
            long[] dist = new long[n];
            Arrays.fill(dist, INF);
            dist[s] = 0;
        
            // |V| - 1 rounds of relaxation
            for (int i = 0; i < n - 1; i++) {
                boolean updated = false;
                for (Edge e : edges) {
                    if (dist[e.from] != INF &&
                        dist[e.to] > dist[e.from] + e.w) {
                        dist[e.to] = dist[e.from] + e.w;
                        updated = true;
                    }
                }
                // 提前收敛优化
                if (!updated) break;
            }
        
            // negative cycle detection
            for (Edge e : edges) {
                if (dist[e.from] != INF &&
                    dist[e.to] > dist[e.from] + e.w) {
                    return null; // 存在负权环
                }
            }
        
            return dist; // 正确的最短路径
        }
        ```
        
        ---
        
        ### 5.3 返回值设计说明
        
        - 返回 `null` → 存在负权环（无解）
        - 返回 `dist[]` → 合法最短路径
        
        ---
        
        ## 6. 复杂度分析
        
        - **时间复杂度**：
            
            ```
            O(|V| × |E|)
            
            ```
            
        - **空间复杂度**：
            
            ```
            O(|V|)
            
            ```
            
        
        ### 对比总结
        
        | 算法 | 是否允许负权 | 是否检测负环 | 复杂度 |
        | --- | --- | --- | --- |
        | Dijkstra | ❌ | ❌ | O(E log V) |
        | Bellman–Ford | ✅ | ✅ | O(VE) |
        | Floyd | ✅ | ❌（需额外） | O(V³) |
    - 推出矛盾 ⇒ 不可能
