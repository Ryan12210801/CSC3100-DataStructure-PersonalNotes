# AVL Tree

## essence: BST+ 通过旋转保持高度平衡

每个节点需要另外一个height 参数来维持高度平衡


平衡因子

```jsx
BF=height（left）-height(right)  // BF >=+-2  , imbalance
```

AVL node结构 （java）

```java
class AvlNode{
	int key;
	int height;
	
	AvlNode left, right;
	
	AvlNode(int k) {
		key=k;
		height=1;  // new height as 1
		}
	}
```

 AVL Tree结构 基本功能 （java）

```java
public class AVLTree {

    // ================================
    // 工具函数
    // ================================

    // 返回节点高度（空节点高度为 0）
    int height(AVLNode n) {
        return n == null ? 0 : n.height;
    }

    // 更新当前节点高度 = 左右子树高度最大值 + 1
    void updateHeight(AVLNode n) {
        n.height = Math.max(height(n.left), height(n.right)) + 1;
    }

    // 平衡因子（Balance Factor = 左高 - 右高）
    int getBalance(AVLNode n) {
        return (n == null) ? 0 : height(n.left) - height(n.right);
    }

```

### key: four kinds of rotation  ()

Rotation（旋转）不是“数学上的旋转”。

它只是“重新连接指针”，让某个更合适的节点成为子树的根。

(visualization see [https://www.cs.usfca.edu/~galles/visualization/AVLtree.html](https://www.cs.usfca.edu/~galles/visualization/AVLtree.html))

single rotation

- LL  （right rotation）
- RR  （left rotation）

double rotation

- LR   （left rotation+ right rotation)
- RL      (right rotation + left rotation)

```java
//right rotation 
    // ---------- Right Rotation (LL case) ----------
    //         y                             x
    //        / \                           / \
    //       x   T3       →                T1  y
    //      / \                               / \
    //     T1  T2                             T2 T3
    // T1 is longer than T2
    
    private AVLNode rotateRight(AVLNode y) {
        AVLNode x = y.left;
        AVLNode T2 = x.right;

        // 右旋
        x.right = y;
        y.left = T2;

        // 必须先更新下层节点高度，再更新上层
        updateHeight(y);
        updateHeight(x);

        return x; // 新根
    }
    
    // ---------- Left Rotation (RR case) ----------
    //      y                                  x
    //     / \                                / \
    //    T1  x           →                  y   T3
    //       / \                            / \
    //      T2 T3                          T1 T2
    //
    private AVLNode rotateLeft(AVLNode y) {
        AVLNode x = y.right;
        AVLNode T2 = x.left;

        x.left = y;
        y.right = T2;

        updateHeight(y);
        updateHeight(x);

        return x; // 新根
    }
```

### 插入操作（AVL Tree 核心之一）

```java
 public AVLNode insert(AVLNode node, int key) {

        // 1. 正常 BST 插入
        if (node == null)
            return new AVLNode(key);

        if (key < node.key)
            node.left = insert(node.left, key);
        else if (key > node.key)
            node.right = insert(node.right, key);
        else
            return node; // 不允许重复键

        // 2. 插入后更新高度
        updateHeight(node);

        // 3. 检查平衡因子
        int balance = getBalance(node);  //通过递归更新了插入路径中所有的高度和平衡因子

        // 4 种不平衡情况：

        // ---------- LL Case ----------
        if (balance > 1 && key < node.left.key)
            return rotateRight(node);

        // ---------- RR Case ----------
        if (balance < -1 && key > node.right.key)
            return rotateLeft(node);

        // ---------- LR Case ----------
        /*     y                    y                     z
					   /                     /                    /   \
					  x      ------>        z           --->     x     y
						  \                  /  
						   z                x
        */

        // 插入在左子树的右侧
        if (balance > 1 && key > node.left.key) {
            node.left = rotateLeft(node.left); // 先左旋子树
            return rotateRight(node);          // 再右旋
        }

        // ---------- RL Case ----------
        if (balance < -1 && key < node.right.key) {
            node.right = rotateRight(node.right);
            return rotateLeft(node);
        }

        // 平衡，不需要旋转
        return node;
    }

```

### 删除操作 （更加复杂）

具体代码在下方，但是逻辑更重要。

逻辑

1. 正常BST删除逻辑（已经较复杂，需考虑删除节点的子节点关系（0/1/2子节点））
2. 更新删除路径中的节点高度
3. 通过旋转 rebalance

```java
    public AVLNode deleteNode(AVLNode root, int key) {

        // 1. BST 正常删除
        if (root == null)
            return root;

        if (key < root.key)
            root.left = deleteNode(root.left, key);

        else if (key > root.key)
            root.right = deleteNode(root.right, key);

        else { 
            // 找到需要删除的节点

            // 情况 1: 单子树或无子树
            if (root.left == null || root.right == null) {
                AVLNode temp = (root.left != null) ? root.left : root.right;

                // 无子树
                if (temp == null) {
                    root = null;
                } else { // 有一个子树
                    root = temp;
                }
            }
            else {
                // 情况 2: 有两个子树 → 找 inorder successor
                AVLNode successor = minValueNode(root.right);
                root.key = successor.key;
                root.right = deleteNode(root.right, successor.key);
            }
        }

        // 如果树只剩一个节点
        if (root == null)
            return root;

        // 2. 更新高度
        updateHeight(root);

        // 3. rebalance
        int balance = getBalance(root);

        // ---------- LL Case ----------
        if (balance > 1 && getBalance(root.left) >= 0)
            return rotateRight(root);

        // ---------- LR Case ----------
        if (balance > 1 && getBalance(root.left) < 0) {
            root.left = rotateLeft(root.left);
            return rotateRight(root);
        }

        // ---------- RR Case ----------
        if (balance < -1 && getBalance(root.right) <= 0)
            return rotateLeft(root);

        // ---------- RL Case ----------
        if (balance < -1 && getBalance(root.right) > 0) {
            root.right = rotateRight(root.right);
            return rotateLeft(root);
        }

        return root; // 平衡完成
    }

    // 找右子树中的最小值（用于删除）
    AVLNode minValueNode(AVLNode node) {
        AVLNode current = node;
        while (current.left != null)
            current = current.left;
        return current;
    }

```

Question

1. 为什么操作旋转不会影响BST的有序性？
2. 插入时判断 LR / RL，用的是 **插入的 key**；

       而删除时判断 LR / RL，用的是 **子树的 balance factor**。

为什么这里不能再用 key？

3. 在一次插入中，
    
    **最多会发生几次旋转？**
    
    在一次删除中呢？
    
    为什么两者不同？
    

4.

假设某节点 balance = -2，且它的right child balance = 0 。

这是哪一类情况？

应该进行哪种旋转？为什么

5.

如果你把 AVL 的所有旋转代码都删掉，只保留 height 和 balance 的计算，

这棵树会退化成什么数据结构？

它在时间复杂度上会发生什么变化？

参考回答：

1. 旋转只改变节点之间的父子关系，但不改变中序遍历序列；
因此所有节点的相对大小关系保持不变，BST 有序性自然不被破坏。
2. 插入时，插入的node的在tree的最底端，如果imbalance的话，说明imbalance就发生在插入的no的上，所以根据判断key的大小和node.left/right.key的大小就可以明确其LR/RL类型
    
     删除时，这个key已经被我们删除，不能起到比较作用，所以可以必须得根据left/right的balance因子来判断类型 （实际上，插入也能使用平衡因子来判断类型）
    
3. 插入时，最多发生两次旋转
    
    删除时，最多发生多于两次的旋转，但是我们无法判断具体最多多少次，因为删除过后可能会破坏结构，然后进行rebalance,但是height更高的节点可能还没平衡，所以仍然需要rebalance
    
    - 插入：**最多 1 次 rebalance（单旋或双旋）**
    - 删除：**最坏 O(log n) 次 rebalance（沿路径向上）**
4. 类型为RR，应该left rotation （这种情况只会在删除时出现，其实并非严格意义上的rr，但是我们通常会把它当作rr来处理，实际上还需要根据情况旋转。）
5. 这棵树会退化成普通的BST。时间复杂度和BST一样，不过会多一些更新height和balance的常数。
    
    (由于没有结构修复，树可能退化为链表，
    
    搜索 / 插入 / 删除的最坏复杂度退化为 O(n)，
    
    而 height / balance 只剩下“记录信息”，不再提供性能保障。)
