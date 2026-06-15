---
title: LeetCode 173. 二叉搜索树迭代器
tags: [二叉搜索树, 栈, 迭代器, 中序遍历, 设计]
difficulty: Medium
category: 设计
date: 2026-06-15
---

# LeetCode 173. 二叉搜索树迭代器 —— 用栈拆解递归，让中序遍历"按需进行"

## 前言

如果给你一棵二叉搜索树（BST），让你从小到大依次输出所有节点值，你会怎么做？几乎所有人都会脱口而出：**中序遍历**。递归写起来也就 5 行代码，一次跑完。

但如果面试官追问：**"我不要一次性遍历完，我要一个迭代器。每次调用 `next()` 才返回下一个最小值，而且平均 O(1) 时间、O(h) 空间。"**

这时候你发现——递归版中序遍历是"一把梭"跑完的，根本停不下来。你需要控制它，让它每次只走一步。

LeetCode 173 **二叉搜索树迭代器**（Binary Search Tree Iterator）就是让你实现这样一个"受控的中序遍历"。本文将带你理解一个经典设计：**用栈模拟递归，把中序遍历拆成按需执行的原子步骤**。

> **前置阅读**：建议先掌握二叉树的中序遍历（递归版和迭代版），理解 BST 的性质：**中序遍历 BST 得到严格递增序列**。

---

## 问题描述

### LeetCode 173. Binary Search Tree Iterator（二叉搜索树迭代器）

> 实现一个二叉搜索树迭代器类 `BSTIterator`，表示一个按**中序遍历**二叉搜索树（BST）的迭代器：
>
> - `BSTIterator(TreeNode root)` 初始化 BSTIterator 类的一个对象。BST 的根节点 `root` 会作为构造函数的一部分给出。指针应初始化为一个不存在于 BST 中的数字，且该数字小于 BST 中的任何元素。
> - `boolean hasNext()` 如果我们向右遍历时，指针的右侧还有数字，则返回 `true`；否则返回 `false`。
> - `int next()` 将指针向右移动，然后返回指针处的数字。
>
> 注意，指针初始化为一个不存在于 BST 中的数字，所以对 `next()` 的首次调用将返回 BST 中的最小元素。
>
> 你可以假设 `next()` 调用总是有效的——即，当调用 `next()` 时，BST 的中序遍历中至少存在一个下一个数字。

**示例：**

```
输入：
["BSTIterator", "next", "next", "hasNext", "next", "hasNext", "next", "hasNext", "next", "hasNext"]
[[[7, 3, 15, null, null, 9, 20]], [], [], [], [], [], [], [], [], []]

输出：
[null, 3, 7, true, 9, true, 15, true, 20, false]

解释：
BSTIterator bSTIterator = new BSTIterator([7, 3, 15, null, null, 9, 20]);
bSTIterator.next();    // 返回 3
bSTIterator.next();    // 返回 7
bSTIterator.hasNext(); // 返回 true
bSTIterator.next();    // 返回 9
bSTIterator.hasNext(); // 返回 true
bSTIterator.next();    // 返回 15
bSTIterator.hasNext(); // 返回 true
bSTIterator.next();    // 返回 20
bSTIterator.hasNext(); // 返回 false
```

**图示：**

```
        7
       / \
      3   15
         /  \
        9    20

中序遍历顺序: 3 → 7 → 9 → 15 → 20
```

**提示：**
- 树中节点的数目在范围 `[1, 10^5]` 内
- `0 <= Node.val <= 10^6`
- 最多调用 `10^5` 次 `hasNext` 和 `next` 操作

**进阶：** 你可以设计一个满足下述条件的解决方案吗？`next()` 和 `hasNext()` 操作均摊时间复杂度为 O(1)，并使用 O(h) 内存。其中 `h` 是树的高度。

---

## 核心思想

### 问题本质：把递归拆成"可控步骤"

先回顾标准的迭代版中序遍历：

```
迭代版中序遍历（一次跑完）：
  stack = []
  curr = root
  while (curr || stack.length) {
      while (curr) {           // 一路向左走到底
          stack.push(curr)
          curr = curr.left
      }
      curr = stack.pop()       // 弹出栈顶，访问它
      console.log(curr.val)
      curr = curr.right        // 转向右子树
  }
```

你会发现，这套代码其实**天生适合拆成迭代器**：

- `next()` = 弹出栈顶节点，记录值，然后"向右一步，再一路向左"——这正是上述循环中"弹栈→访问→转向右子树"的部分
- `hasNext()` = 栈不为空，说明还有节点没被访问

关键洞察：**在构造函数中，我们先把"一路向左"的初始化做了，这样 `next()` 第一次调用就能直接返回最小值**。

### 数据结构设计

迭代器只需要一个成员：**栈**。

```
栈 的 角色：保存"还没被访问的节点"

初始化：从根节点出发，一路向左走到底，把沿途所有节点压栈。
  这相当于把"递归调用栈"显式保存下来。

next()：弹出一个节点 → 这是当前最小值
        → 如果它有右子树，对右子树做"一路向左"压栈（为后续做准备）
        → 返回该节点的值

hasNext()：栈是否非空
```

```
初始化 BSTIterator(root=7)：

        7
       / \
      3   15
         /  \
        9    20

一路向左压栈: [7, 3]  ← 3 在栈顶
                               
此时栈中存的就是"从根到最左叶子"的整条路径。
栈顶 = 整棵树的最小值。
```

---

## 思路分析

### 解法：受控中序遍历（栈模拟递归）⭐ 唯一推荐

**核心思路**：用栈保存"还没有被访问的节点"。构造时做"一路向左"初始化，`next()` 时弹栈 → 处理右子树 → 返回。

```
算法框架：

1. 构造函数 BSTIterator(root):
   this.stack = []
   this._pushLeft(root)   ← 辅助方法：一路向左，全部压栈

2. _pushLeft(node):
   从 node 出发，一路向左走：
     while (node) {
         stack.push(node)
         node = node.left
     }
   效果：栈顶是当前子树的最小节点

3. next():
   node = stack.pop()                ← 栈顶 = 当前最小值
   if (node.right) {
       this._pushLeft(node.right)    ← 对右子树做"一路向左"（右子树的所有左子孙都小于右子树的根）
   }
   return node.val

4. hasNext():
   return stack.length > 0
```

### 为什么这个设计是 O(h) 空间？

一个常见疑问：栈里到底存了多少节点？会不会变成 O(n)？

答案是 **栈的大小不超过树的高度 h**。

```
证明：
  栈中存储的是"从根节点出发，到当前最小值行走路径上，
  所有已经走过但还没访问的节点"。
  
  具体来说，栈里的节点分布在树的不同层上——
  它们构成了一条从根出发、逐层向左下延伸的路径。
  
  在 BST 中，这样一条路径的长度不会超过树的高度 h。
  
  平衡树：O(log n)
  退化链表：O(n)（最坏情况）
```

```
栈中最多同时存在的节点示意：

        [7]       ← 已经在栈中（还没轮到访问）
       /   \
     [3]   15     ← 3 在栈中
           /  \
          9   20
          
  实际栈内容: [7, 3]（深度 = 2 = h）
  
  当 3 被弹出后，检查 3 没有右子树，
  栈变为: [7]，继续弹出 7...
  
  每条从根向左下的路径上，每层最多一个节点在栈中！
```

### 为什么不需要其他解法？

| 尝试 | 为什么不行 |
|------|-----------|
| 构造时一次性递归存数组 | 空间 O(n)，不符合进阶要求的 O(h) |
| 用生成器/yield | 本质也是栈模拟，且多数语言需要额外语法 |
| 在二叉树节点上加 `parent` 指针 | 改变了数据结构，题目不允许 |
| 维护中序前驱后继（Threaded BST / Morris） | 虽然空间 O(1)，但实现复杂，且题目要求均摊 O(1)，不是严格 O(1) |

**栈模拟递归**是这个场景的标准答案——简洁、符合空间要求、利用 BST 性质。

---

## 代码实现

### JavaScript 版本

```javascript
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 */
var BSTIterator = function(root) {
    this.stack = [];
    // 初始化：从根节点一路向左走到底
    this._pushLeft(root);
};

/**
 * 辅助方法：从 node 出发，一路向左，全部压栈
 * @param {TreeNode} node
 */
BSTIterator.prototype._pushLeft = function(node) {
    while (node) {
        this.stack.push(node);
        node = node.left;
    }
};

/**
 * @return {number}
 */
BSTIterator.prototype.next = function() {
    // 弹出栈顶——这正是当前中序序列的下一个节点
    const node = this.stack.pop();

    // 关键：如果这个节点有右子树，对右子树做"一路向左"
    // 原因：右子树的根比当前节点大，但右子树中最左的节点
    // 是"比当前节点大的所有节点中最小的一个"
    if (node.right) {
        this._pushLeft(node.right);
    }

    return node.val;
};

/**
 * @return {boolean}
 */
BSTIterator.prototype.hasNext = function() {
    return this.stack.length > 0;
};
```

### Python 版本

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class BSTIterator:
    def __init__(self, root: TreeNode):
        self.stack = []
        # 初始化：从根节点一路向左走到底
        self._push_left(root)

    def _push_left(self, node: TreeNode) -> None:
        """辅助方法：从 node 出发，一路向左，全部压栈"""
        while node:
            self.stack.append(node)
            node = node.left

    def next(self) -> int:
        # 弹出栈顶——这正是当前中序序列的下一个节点
        node = self.stack.pop()

        # 关键：如果这个节点有右子树，对右子树做"一路向左"
        if node.right:
            self._push_left(node.right)

        return node.val

    def hasNext(self) -> bool:
        return len(self.stack) > 0
```

---

## 逐步推演

以 `root = [7, 3, 15, null, null, 9, 20]` 为例，展示每次调用的内部状态：

```
树结构：

        7
       / \
      3   15
         /  \
        9    20
```

### 构造阶段：`new BSTIterator(root)`

```
_pushLeft(7):
  node = 7 → stack.push(7), node = 7.left = 3
  node = 3 → stack.push(3), node = 3.left = null
  node = null → 停止

初始栈状态: [7, 3]  ← 栈顶是 3（最小值）
```

### 第一次 `next()` — 返回 3

```
弹栈: node = 3, stack = [7]
       node.right = null → 不需要 pushLeft

返回: 3
```

### 第二次 `next()` — 返回 7

```
弹栈: node = 7, stack = []
       node.right = 15 ≠ null → _pushLeft(15)

_pushLeft(15):
  node = 15 → stack.push(15), node = 15.left = 9
  node = 9  → stack.push(9),  node = 9.left = null
  node = null → 停止

栈状态: [15, 9]  ← 栈顶是 9

返回: 7
```

### 第三次 `next()` — 返回 9

```
弹栈: node = 9, stack = [15]
       node.right = null

返回: 9
```

### 第四次 `next()` — 返回 15

```
弹栈: node = 15, stack = []
       node.right = 20 ≠ null → _pushLeft(20)

_pushLeft(20):
  node = 20 → stack.push(20), node = 20.left = null

栈状态: [20]

返回: 15
```

### 第五次 `next()` — 返回 20

```
弹栈: node = 20, stack = []
       node.right = null

返回: 20
```

### `hasNext()` — 返回 `false`

```
栈为空 → 返回 false，遍历结束
```

### 完整执行序列

```
调用顺序:    next()  next()  next()  next()  next()  hasNext()
返回值:       3       7       9       15      20      false
───────────────────────────────────────────────────────────
弹栈前栈:    [7,3]   [7]     [15,9]  [15]    [20]    []
弹栈后栈:    [7]     []      [15]    []      []      []
pushLeft:    -       [15,9]  -       [20]    -       -
```

> 💡 **核心观察**：注意第二次 `next()`——弹出了 7 之后，栈为空。但还有右子树 `15` 没被处理，所以立即对 15 做 `_pushLeft`，栈变成 `[15, 9]`。这个"弹栈后主动向右探索"的动作，正是中序遍历"左→根→右"中转换到"右"的那一步。

---

## 复杂度分析

| 项目 | 分析 |
|------|------|
| **时间复杂度 — next()** | 均摊 O(1)。单次 `next()` 可能触发 `_pushLeft`（O(h)），但每个节点恰好被压栈和弹栈各一次，N 次调用总操作数 = 2N，均摊 O(1) |
| **时间复杂度 — hasNext()** | O(1)，只检查栈长度 |
| **空间复杂度** | O(h)。栈中同时存在的节点数不超过树的高度。平衡树为 O(log n)，退化链表 O(n) |

### 均摊分析的直观理解

```
"next() 有时候快、有时候慢"——均摊 O(1) 怎么理解？

考虑整棵树的每个节点：
  • 每个节点恰好被 push 一次（在 _pushLeft 中）
  • 每个节点恰好被 pop 一次（在 next 中）

N 个节点 → 总共 N 次 push + N 次 pop = 2N 次操作

调用 N 次 next() → 平均每次 = 2N / N = 2 次操作 → O(1)

虽然某次 next() 可能做多次 push（当节点有很深的左子树时），
但"总工作量 / 总调用次数"是常数级别的。
```

> 这和动态数组（如 C++ 的 `vector`）的均摊分析是一样的道理——偶尔扩容很慢，但平均插入是 O(1)。

---

## 举一反三

### 本题在 BST + 栈 体系中的位置

```
LeetCode 94 二叉树的中序遍历（Easy）
  → 递归版 和 迭代版（栈模拟）
  ↓
LeetCode 173 二叉搜索树迭代器（Medium）⭐ 本题
  → 把迭代版中序遍历拆成 next() / hasNext()，栈做成员变量
  ↓
LeetCode 98 验证二叉搜索树（Medium）
  → 中序遍历检查是否严格递增，同样可以用栈或递归
  ↓
LeetCode 230 二叉搜索树中第K小的元素（Medium）
  → 和本题几乎一模一样——用栈做 k 次 next()
  ↓
LeetCode 99 恢复二叉搜索树（Medium）
  → Morris 遍历（O(1) 空间），中序找出两个被交换的节点
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [94. 二叉树的中序遍历](https://leetcode.com/problems/binary-tree-inorder-traversal/) | 迭代版中序：栈 + 一路向左 | 本题的前置知识——理解"栈模拟中序"就能秒懂迭代器 |
| [230. 二叉搜索树中第K小的元素](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | 用栈做 k 次 next() | 直接复用本题代码，调用 k-1 次 next() 后返回 next() |
| [98. 验证二叉搜索树](https://leetcode.com/problems/validate-binary-search-tree/) | 中序遍历检查递增 | 解法之一：边遍历边比较前驱，同样用栈或递归 |
| [99. 恢复二叉搜索树](https://leetcode.com/problems/recover-binary-search-tree/) | 中序遍历找到异常节点 | 进阶版——用 Morris 遍历实现 O(1) 空间 |
| [653. 两数之和 IV - 输入二叉搜索树](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/) | 双指针在 BST 上 | 可以用两个迭代器（一个正序、一个逆序）做双指针 |
| [285. 二叉搜索树中的中序后继](https://leetcode.com/problems/inorder-successor-in-bst/) | 找中序后继 | 本题 next() 实质就是在找中序后继 |

### 迭代器模式在算法题中的扩展

本题揭示了一个通用设计模式：**把遍历算法拆成迭代器**。

```
通用模板：
  1. 构造函数 → 初始化遍历状态（压入第一批候选节点）
  2. next()   → 消费当前结果，推进到下一个状态
  3. hasNext() → 判断是否还有结果可消费

这个模式不仅适用于 BST 中序遍历，还适用于：
  • 图的 BFS → 队列存储下一层
  • 图的 DFS → 栈存储待访问节点
  • 堆的依次弹出 → 堆本身作为状态
  • N 叉树的扁平化迭代器
```

### 进阶变体：反向迭代器（逆序中序）

如果面试官追问："再实现一个从大到小遍历的迭代器呢？"

答案很简单——把"一路向左"改成"一路向右"即可：

```javascript
// 逆序迭代器：每次返回最大值（递减序列）
var BSTReverseIterator = function(root) {
    this.stack = [];
    this._pushRight(root);  // 一路向右走到底
};

BSTReverseIterator.prototype._pushRight = function(node) {
    while (node) {
        this.stack.push(node);
        node = node.right;  // 向右走
    }
};

BSTReverseIterator.prototype.next = function() {
    const node = this.stack.pop();
    if (node.left) {          // 注意：弹栈后处理左子树
        this._pushRight(node.left);
    }
    return node.val;
};
```

> 这个变体可以用于 LeetCode 653（两数之和 IV）——正序迭代器 + 逆序迭代器，就得到了"BST 上的双指针"。

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **173 二叉搜索树迭代器** | 🟡 Medium | 栈模拟递归，受控中序遍历 | 栈 + 一路向左 |

这道题的核心就三句话：

| 要点 | 说明 |
|------|------|
| **构造时** | 从根一路向左压栈——准备好最小值 |
| **next()** | 弹栈顶 → 如果它有右子树，对右子树一路向左压栈 |
| **hasNext()** | 栈非空就有下一个 |

**三步记忆法：**

1. **构造函数** = 把"迭代版中序遍历"中第一段 `while (curr) { push; curr = curr.left }` 执行掉——栈里是从根到最小值的整条路径
2. **next()** = 弹栈 + 处理右子树——弹出的节点正是当前中序位置，然后为它的右子树做准备（如果有）
3. **空间 O(h)** = 栈里最多存一条从根向左下的路径——每层最多一个节点

> **记住一句话：二叉搜索树迭代器 = 把"迭代版中序遍历"的 while 循环拆开，用栈保存上下文，每次 next() 只执行一次弹栈 + 一次向右探索。stack 的栈顶永远是"当前剩余节点中最小的那个"。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 94. 二叉树的中序遍历](https://leetcode.com/problems/binary-tree-inorder-traversal/)（前置知识） | [LeetCode 230. 二叉搜索树中第K小的元素](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)（直接复用迭代器） | [LeetCode 98. 验证二叉搜索树](https://leetcode.com/problems/validate-binary-search-tree/)（中序递增检查） | 设计/迭代器系列持续更新中，关注不迷路。
