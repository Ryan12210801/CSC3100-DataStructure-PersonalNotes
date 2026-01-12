# Heap

## **Heap = 完全二叉树 + 堆序性质**

- **完全二叉树**
    
    除最后一层外都满，最后一层从左到右填
    
- **堆序性质**
    - **最大堆（Max-Heap）**：父 ≥ 子
    - **最小堆（Min-Heap）**：父 ≤ 子

重要：

**Heap 不是 BST**，兄弟节点、左右子树之间**没有全局大小关系**

**Heap 不关心“顺序”，**

**只关心“谁最小”；**

**结构简单，是为了效率服务。**

这里以Min-Heap 为例展示结构和代码细节

### Min-Heap 结构（用array作为底层数据结构）

```jsx
public class MinHeap {

private int[] heap;   // array存储完全二叉树 Heap[0]=null 
private int size;     // 当前堆大小
private int capacity; // 容纳量

public MinHeap(int cap) {
    capacity = cap;
    heap = new int[cap+1];
    size = 0;
}
```

### Index 关系（Heap 的“骨架”）

```java
private int parent(int i) {
    return i  / 2;
}

private int left(int i) {
    return 2 * i ;
}

private int right(int i) {
    return 2 * i + 1;
}

```

### 核心工具函数 swap

```java
    private void swap(int i, int j) {
        int tmp = heap[i];
        heap[i] = heap[j];
        heap[j] = tmp;
    }

```

### 关键函数 insert

### 插入的本质逻辑

> 新元素插在最后
> 
> 
> ⇒ 只可能 **比父节点小**
> 
> ⇒ 沿着父链向上修复堆序
> 

![image.png](Heap/image.png)

```java
	void insert(int key) {

        // 1. 插入到末尾（保持完全二叉树）
        heap[++size] = key;
        int i = size;

        // 2. 上滤（sift-up）
        while (i > 1 && heap[i / 2] > heap[i]) {
            swap(i, i / 2);
            i = i / 2;
        }
```

## 删除最小值（Extract-Min —— 下滤）

### 删除的本质逻辑

> 删除根
> 
> 
> ⇒ 用最后一个元素顶替
> 
> ⇒ 新根可能过大
> 
> ⇒ 向下修复堆序
> 

### **Extract-Min 代码**

```java
    int extractMin() {

        if (size == 0)
            throw new RuntimeException("Heap is empty");

        int min = heap[1];

        // 1. 用最后一个节点替换根
        heap[1] = heap[size--];

        // 2. 下滤
        heapify(1);

        return min;
    }

```

### heapify（下滤）代码

```java
   void heapify(int i) {

        int smallest = i;
        int l = 2 * i;
        int r = 2 * i + 1;

        if (l <= size && heap[l] < heap[smallest])
            smallest = l;

        if (r <= size && heap[r] < heap[smallest])
            smallest = r;

        // 如果堆序被破坏，继续向下
        if (smallest != i) {
            swap(i, smallest);
            heapify(smallest);
        }
    }
```

### 下滤的本质

- 只与 **更小的子节点** 交换
- 方向唯一：**根 → 叶**

## Floyd建堆（O（n））（如果一个一个插入，时间复杂度为O（nlogn））

代码

```java
void buildHeap(int[] arr) {

    size = arr.length;
    heap = new int[size + 1];

    // 拷贝到 1-based heap
    for (int i = 0; i < arr.length; i++)
        heap[i + 1] = arr[i];

    // 从最后一个非叶子节点开始下滤
    for (int i = size / 2; i >= 1; i--) {
        heapify(i);
    }
}

```

| 操作 | 时间复杂度 | 说明 |
| --- | --- | --- |
| peek / getMin | O(1) | 堆顶 |
| insert | O(log n) | 上滤 |
| extractMin / extractMax | O(log n) | 下滤 |
| buildHeap | O(n) | Floyd 建堆 |
| heapify（单次） | O(log n) | 沿树高 |
| heapSort | O(n log n) | 排序 |
| 查找任意元素 | O(n) | 无序 |

*Heapsort time-complexity 均为nlogn 但不稳定

---

## 在java中，有一个ADT类型名为PriorityQueue， 其本质就为Min-Heap,以下将简单讲解其用法。

## 一、基本用法（核心函数）

### 1. 声明

```java
// 默认：小根堆（Min-Heap）
PriorityQueue<Integer> pq = new PriorityQueue<>();

// 大根堆（Max-Heap）
PriorityQueue<Integer> maxPQ =
        new PriorityQueue<>(Collections.reverseOrder());

```

---

### 2. 常用方法

```java
pq.offer(x);     // 插入元素
pq.poll();       // 删除并返回堆顶
pq.peek();       // 查看堆顶（不删除）
pq.size();       // 元素数量
pq.isEmpty();    // 是否为空

```

---

### 3. 简单示例

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();

pq.offer(3);
pq.offer(1);
pq.offer(5);

pq.peek(); // 1
pq.poll(); // 1
pq.poll(); // 3

```