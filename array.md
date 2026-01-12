# Array

---

## 本质：连续内存 + index 映射

Array 的本质不是“能存很多元素”，而是：

> 元素在内存中连续存放，index 可直接映射到地址
> 

因此：

- 访问第 i 个元素是 **O(1)**（random access）
- 不需要遍历、不需要指针跳转

---

## 基本性质

- 元素类型相同（same type）
- 长度在创建时固定（fixed length）
- 下标从 **0** 开始（C / C++ / Java）

---

## ADT 视角（和 AVL 的“工具函数”类比）

Array 的核心操作非常少：

```
createArray(n)
retrieve(arr, i)
store(arr, i, x)

```

📌 重要思想（考试爱考）：

> 可以在不知道底层实现的情况下设计算法
> 
> 
> ——这就是 ADT 的意义
> 

---

## 时间复杂度总览（非常重要）

| 操作 | 时间复杂度 | 原因 |
| --- | --- | --- |
| 访问 arr[i] | O(1) | index → address |
| 修改 arr[i] | O(1) | 同上 |
| 查找（无序） | O(n) | 必须遍历 |
| 查找（有序） | O(log n) | binary search |
| 尾部插入 | O(1) | 不搬移 |
| 中间插入 | O(n) | 元素整体右移 |
| 删除 | O(n) | 元素整体左移 |

📌 核心判断标准：

> 是否需要 shift（搬移元素）
> 

---

## 插入与删除（算法本质）

### 插入（Insert）

假设在 index = k 插入：

1. 从末尾开始，所有元素向右移动一格
2. 在 arr[k] 放新元素

成本来自 **shift**

最坏：O(n)

---

### 删除（Delete）

删除 index = k：

1. arr[k+1 ... n-1] 全部左移
2. 覆盖被删元素

 本质仍是 shift

最坏：O(n)

---

## Array vs Linked List

|  | Array | Linked List |
| --- | --- | --- |
| 内存 | 连续 | 不连续 |
| 访问 | O(1) | O(n) |
| 插入/删除 | O(n) | O(1)（已定位） |
| cache-friendly | 是 | 否 |

结论式答法：

> Array 适合 读多写少、频繁随机访问的场景。
> 

---

## Iteration（遍历的算法意义）

遍历数组的本质：

> 对每个 index 做一次 retrieve
> 

```
for i = 0 .. n-1

```

时间复杂度固定：

```
O(n)

```

for-each 只是语法糖，本质不变。

---

## 典型算法：Frequency Table

问题：

给定一组大写字母，统计每个字母出现次数。

### 核心思想

- 用 **index 直接映射含义**
- `'A' → 0`, `'B' → 1`, …

```
count[ch - 'A']++

```

📌 为什么高效？

- 无比较
- 无查找
- 全程 O(n)

---

## 固定长度的“突破”思想

Array **本身不能扩容**。

但可以：

1. 创建一个更大的新数组
2. 把旧数组 copy 进去
3. 用新数组替换旧数组重要认知：

> Java 中的“动态数组”（如 ArrayList）
> 
> 
> **不是数组会变大，而是“换数组”**
> 

---

## Pass-by-Value

Java 中：

- **一切都是 pass-by-value**
- 但数组 / 对象传的是“引用的值”

结果：

- 能修改数组内容
- 不能让外部变量指向新数组

考试一句话：

> Java does not support pass-by-reference; object references are passed by value.
>