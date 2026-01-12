# Hash

# **Hash Table = Array + Hash Function + 冲突处理**

- **Array**
    
    固定大小的 bucket 数组
    
- **Hash Function**
    
    key → index
    
- **Collision Resolution**
    
    冲突几乎是必然的，只能“解决”，难以“避免”
    

## Hash function  灵魂所在

本质：

> 将「大 key 空间」
> 
> 
> 映射到「小 index 空间」
> 

### 一个简单 Hash Function（整数）

```java
h(key) = key % m
```

- m 通常取 **质数 （m也通常是array的size）**

## 重点：避免冲突的各种方法（为何冲突？→可能hash函数把多个key映射到同一个index上）

# 1. Chaining（拉链法）

### 思想

> 每个 bucket 不是一个格子，而是一条“链/动态数组”
> 
> 
> 冲突就挂到同一个 bucket 里
> 

```java
table[i] -> (k1) -> (k2) -> (k3)
```

### 操作本质

- **insert**：算 index → 插到链表头/尾
- **search**：算 index → 在链表里线性查
- **delete**：算 index → 链表里删除

## 时间复杂度

令

- `m` = bucket 数
- `n` = 元素数
- `α = n/m` = load factor（平均每个 bucket 的元素个数）

### 平均情况（均匀散列假设）

- 一个 key 落入任一 bucket 的概率近似 `1/m`
- 所以**每个 bucket 的期望长度**是 `E[length] = n/m = α`

因此：

- **search / insert / delete 平均**：`O(1 + α)`
    - `O(1)`：算 hash + 定位 bucket
    - `O(α)`：在 bucket 内查找/处理的期望长度

### 最坏情况

- 全部 key 都落到同一 bucket → 链表长度 n
    
    ⇒ `O(n)`
    

## Chaining Hash具体代码

### 1.结构

```java
class ChainingHash {

    private LinkedList<Integer>[] table;
    private int capacity;

    public ChainingHash(int capacity) {
        this.capacity = capacity;
        table = new LinkedList[capacity];   // table 是 array
        for (int i = 0; i < capacity; i++)
            table[i] = new LinkedList<>();  // 每个格子是一个 bucket
    }

    private int hash(int key) {
        return key % capacity;
    }
}

```

### 2.插入

```java
public void insert(int key) {
    table[hash(key)].add(key);
}

```

### 3.查找

```java
public boolean contains(int key) {
    return table[hash(key)].contains(key);
}

```

### 4.删除（最自然）

```java
public void remove(int key) {
    table[hash(key)].remove((Integer) key);
}
```

# 2.Linear Probing（线性探测）

## 思想

> 不用链表，所有元素都放在表内
> 
> 
> 冲突时按固定步长往后找空位
> 

```
i = h(key)
i, i+1, i+2, ... (mod m)
```

## 操作本质

- **insert**：从 `h(k)` 起一路找第一个空槽
- **search**：从 `h(k)` 起一路扫，直到找到 key 或遇到空槽（空槽意味着“没插入过”）
- **delete**：不能直接清空（会断掉探测链），要用 **tombstone（墓碑标记）**

## 时间复杂度

- 若 α 控制在 **0.5~0.7**：平均接近 `O(1)`
- α 接近 1：性能急剧下降（因为 cluster 变长）

在均匀散列假设下：

- **成功查找**

$$
\Theta\!\left(\frac{1}{1-\alpha}\right)

$$

- **失败查找 / 插入**

$$
\Theta\left(\frac{1}{(1-\alpha)^2}\right)
$$

## Linear Probing Hash具体代码

### 1.结构

```java
class LinearProbingHash {

    private Integer[] table;     // table 本身是 array
    private boolean[] deleted;   // 墓碑
    private int capacity;

    public LinearProbingHash(int capacity) {
        this.capacity = capacity;
        table = new Integer[capacity];
        deleted = new boolean[capacity];
    }

    private int hash(int key) {
        return key % capacity;
    }
}

```

### 2.插入

```java
public void insert(int key) {
    int i = hash(key);

    while (table[i] != null && !deleted[i]) {
        i = (i + 1) % capacity;
    }

    table[i] = key;
    deleted[i] = false;
}

```

### 3.查找

```java
public boolean contains(int key) {
    int i = hash(key);

    while (table[i] != null || deleted[i]) { //只有table[i]==null 和 deleted[i]== false（没有删除过）才会停下来
        if (table[i] != null && table[i] == key)
            return true;
        i = (i + 1) % capacity;
    }
    return false;
}

```

### 4.删除

```java
table[i] = null;
deleted[i] = true;   // tombstone

```

# 3.Double Hashing(双重哈希)

## 思想

> 冲突时不是 +1，而是用第二个 hash 决定步长
> 
> 
> 让探测序列“像随机的一样”，减少聚集
> 

```
h(k, i) = (h1(k) + i * h2(k)) mod m

```

要求（很关键）：

- `h2(k) != 0`
- 并且 `h2(k)` 与 `m` **互质**（保证能遍历全表，不会只跳一部分格子）
    - 常见做法：m 取质数，h2(k)=1+(k mod (m-1))

## 操作本质

- **insert/search**：沿着 “由 key 决定的独特跳步” 去探测
- **delete**：同样需要墓碑

## 时间复杂度

**性能接近理想随机散列（alpha 可以比linear probing略高）**

查找 / 插入的期望复杂度是

$$
\Theta\!\left(\frac{1}{1-\alpha}\right)
$$

## Code

### 1.结构

```java
class DoubleHashing {

    private Integer[] table;
    private boolean[] deleted;
    private int capacity;

    public DoubleHashing(int capacity) {
        this.capacity = capacity;
        table = new Integer[capacity];
        deleted = new boolean[capacity];
    }

    private int h1(int key) {
        return key % capacity;
    }

    private int h2(int key) {
        return 1 + (key % (capacity - 1));
    }
}
```

### 2.insert

```java
public void insert(int key) {
int i = 0;

while (true) {
    int idx = (h1(key) + i * h2(key)) % capacity;

    if (table[idx] == null || deleted[idx]) {
        table[idx] = key;
        deleted[idx] = false;
        return;
    }
    i++;
}

```

### 3.查找

```java
public boolean contains(int key) {
    int i = 0;

    while (i < capacity) {
        int idx = (h1(key) + i * h2(key)) % capacity;

        if (table[idx] == null && !deleted[idx])
            return false;

        if (table[idx] != null && table[idx] == key)
            return true;

        i++;
    }
    return false;
}
```

### 4.删除

```java
table[idx] = null;
deleted[idx] = true;
```

# Universal Hashing(一种随机选择hash function的策略)

### 核心思想

不要试图设计一个“永远不冲突”的哈希函数，

而是从一组哈希函数中随机选一个。

## 定义

设：

- `U`：key 的全集
- `m`：hash table 的大小（bucket 数）
- `H`：一组哈希函数h:U→{0,1,…,m−1}
    
$$
h:U \rightarrow \{0, 1, \dots, m-1\}
$$
    

### **Universal Hash Family 的定义**

`H` 是一个 universal hash family，当且仅当：

> 对任意两个不同的 key x≠y
> 
> 
> 从 `H` 中**均匀随机选择一个**哈希函数 `h`，有：
> 
> $$
> \Pr {[h(x) = h(y)]} \le \frac{1}{m}
> $$
> 

## 经典构造

$$
h_{a,b}(x) = ((a \cdot x + b) \bmod p) \bmod m
$$

### 参数选择

- 选一个 **大质数** `p`，满足：
    
    ```
    p > max(key)
    
    ```
    
- table 大小：`m`
- 随机选择：
    - `a ∈ {1, 2, ..., p-1}`
    - `b ∈ {0, 1, ..., p-1}`

关系理解

| 维度 | Universal Hashing | Chaining / Probing |
| --- | --- | --- |
| 解决什么 | 冲突概率 | 冲突发生后怎么办 |
| 类型 | hash 函数选择策略 | 数据结构机制 |
| 是否随机 | 是（选函数） | 否 |
| 是否互斥 | ❌ | ❌ |

# JAVA中自带的Hash基本用法

## 一、最常用的两个类

```java
HashMap<K, V>
HashSet<E>
```

一句话区分：

- **HashMap**：key → value
- **HashSet**：只关心元素是否存在（本质是 HashMap 的 key）

---

## 二、HashMap 的核心函数（必须会）

### 1️⃣ 创建

```java
HashMap<Integer, String> map = new HashMap<>();

```

或指定参数：

```java
HashMap<Integer, String> map = new HashMap<>(16, 0.75f);

```

- `16`：初始桶数量
- `0.75`：负载因子（超过就扩容）

---

### 2️⃣ put —— 插入 / 更新

```java
map.put(1, "A");
map.put(2, "B");
map.put(1, "C"); // 覆盖旧值

```

特点：

- key **不存在** → 插入
- key **已存在** → 覆盖 value

---

### 3️⃣ get —— 查找并返回 value

```java
String v = map.get(1);   // "C"
String x = map.get(3);   // null（不存在）

```

注意：

- **找不到返回 `null`**
- 不会抛异常

---

### 4️⃣ containsKey —— 是否存在

```java
map.containsKey(1);  // true
map.containsKey(3);  // false

```

 只判断“在不在”，不关心 value

---

### 5️⃣ remove —— 删除

```java
map.remove(2);

```

- 删除 key–value 对
- key 不存在也不会报错

---

### 6️⃣ size / isEmpty

```java
map.size();
map.isEmpty();

```

---

## 三、HashSet 的核心函数（和 HashMap 非常像）

### 1️⃣ 创建

```java
HashSet<Integer> set = new HashSet<>();

```

---

### 2️⃣ add —— 插入

```java
set.add(1);
set.add(2);
set.add(1); // 不会重复插入

```

---

### 3️⃣ contains —— 查找

```java
set.contains(1); // true
set.contains(3); // false

```

---

### 4️⃣ remove

```java
set.remove(2);

```

---

## 四、HashSet 和 HashMap 的关系（一句话）

> HashSet 底层就是 HashMap，只是 value 被隐藏了
> 

```java
// 概念等价
HashSet<E> ≈ HashMap<E, Object>

```

---

## 五、hash 在 Java 里“真正起作用”的地方

### 1️⃣ `hashCode()`

```java
int h = key.hashCode();

```

- 决定 key 落在哪个 bucket
- 所有对象都有这个方法

---

### 2️⃣ `equals()`

```java
key.equals(otherKey)

```

- 决定两个 key 是否“逻辑相等”

---

### **必须遵守的一条规则（非常重要）**

> 如果 equals() 相等那么 hashCode() 必须相等
> 

否则：

- HashMap / HashSet 行为错误
- 查不到、删不掉、重复存

---
