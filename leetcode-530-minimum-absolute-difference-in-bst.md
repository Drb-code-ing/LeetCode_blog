---
title: LeetCode 530. 二叉搜索树的最小绝对差
tags: [二叉搜索树, 中序遍历, DFS, 递归, 迭代]
difficulty: Easy
category: 二叉搜索树
date: 2026-06-16
---

# LeetCode 530. 二叉搜索树的最小绝对差 —— 中序遍历即是答案

## 前言

给你一棵二叉搜索树（BST），让你找任意两个节点值之差的最小值。暴力思路是收集所有节点值，两两比较，但那样是 O(n²)。

实际上，这道题只要一句话就能破解：

> **BST 的中序遍历序列是严格递增的。在一个排好序的数组里，最小差值一定出现在相邻元素之间。**

所以问题转化为：**中序遍历 BST，记录前一个访问的节点，每次计算当前节点与前驱的差值，取最小值**。

LeetCode 530 **二叉搜索树的最小绝对差**（Minimum Absolute Difference in BST）就是考察你是否理解 BST 中序遍历的单调性。本文将带你用递归和迭代两种方式实现，并扩展到一类"利用 BST 中序单调性"的题目。

> **前置阅读**：建议掌握 BST 的基本性质（左 < 根 < 右），了解中序遍历的递归和迭代写法。如果还不熟悉迭代版中序遍历，可以先看 [LeetCode 173 二叉搜索树迭代器](./leetcode-173-binary-search-tree-iterator.md)。

---

## 问题描述

### LeetCode 530. Minimum Absolute Difference in BST（二叉搜索树的最小绝对差）

> 给你一个二叉搜索树的根节点 `root`，返回 **树中任意两不同节点值之间的最小差值**。
>
> 差值是一个正数，其数值等于两值之差的绝对值。

**示例 1：**

```
输入：root = [4,2,6,1,3]
输出：1

图示：

        4
       / \
      2   6
     / \
    1   3

解释：最小差值在 1 和 2 之间，或 2 和 3 之间，或 3 和 4 之间，
      最小值为 1。
```

**示例 2：**

```
输入：root = [1,0,48,null,null,12,49]
输出：1

图示：

        1
       / \
      0   48
          /  \
         12   49

解释：最小差值在 0 和 1 之间，值为 1。
      注意：差值也可以是 48 和 49 之间的 1，同样是最小值。
```

**提示：**
- 树中节点的数目在范围 `[2, 10^4]` 内
- `0 <= Node.val <= 10^5`

**注意：** 本题和 [LeetCode 783. 二叉搜索树节点最小距离](https://leetcode.com/problems/minimum-distance-between-bst-nodes/) 完全相同。

---

## 核心思想

### 问题的本质：BST 中序序列天然有序

BST 的定义保证了：

```
对于任意节点 node：
  · node.left  子树中所有节点的值 < node.val
  · node.right 子树中所有节点的值 > node.val
```

因此，**中序遍历（左 → 根 → 右）访问的节点值严格递增**。在递增序列中，最小差值只会出现在相邻元素之间。证明如下：

```
假设有三个值 a < b < c：
  c - a = (c - b) + (b - a) ≥ b - a

推广到任意两个不相邻的元素：
  非相邻的差值 = 中间多个相邻差值之和 ≥ 其中最小的相邻差值

所以：最小差值一定出现在相邻元素之间。
```

于是，答案 = **中序遍历过程中，相邻节点差值的最小值**。

### 数据结构设计

只需要两个变量：

```
minDiff  → 全局最小差值，初始化为 Infinity
prev     → 前驱节点（中序遍历中上一个被访问的节点），初始为 null

遍历过程中：
  如果 prev 存在：
    diff = curr.val - prev.val
    minDiff = min(minDiff, diff)
  更新 prev = curr
```

```
遍历顺序：

        4
       / \
      2   6
     / \
    1   3

中序访问顺序: 1 → 2 → 3 → 4 → 6

相邻差值:
  2-1=1, 3-2=1, 4-3=1, 6-4=2
  最小值: 1 ✅
```

> 💡 **关键洞察**：你不需要把所有值存到数组里比较——在中序遍历中维护一个 `prev`，边走边比较就够了。空间 O(h) 而非 O(n)。

---

## 思路分析

### 解法一：递归 DFS（中序遍历）⭐ 推荐

**核心思路**：递归做中序遍历，维护 `prev` 和 `minDiff` 两个变量。

```
算法框架（递归版）：

1. 初始化 minDiff = Infinity, prev = null

2. 定义递归函数 inorder(node):
   a. 递归基：if (node === null) return

   b. 递归左子树：inorder(node.left)

   c. 处理当前节点（这是中序中"访问根"的位置）：
      if (prev !== null) {
          minDiff = min(minDiff, node.val - prev.val)
      }
      prev = node

   d. 递归右子树：inorder(node.right)

3. 调用 inorder(root)
4. 返回 minDiff
```

### 解法二：迭代栈（中序遍历）

**核心思路**：用栈模拟递归（和 LeetCode 173 一样的"一路向左"套路），在弹出节点时计算差值。

```
算法框架（迭代版）：

1. stack = [], curr = root, prev = null, minDiff = Infinity

2. while (curr || stack.length) {
     // 一路向左走到底
     while (curr) {
         stack.push(curr)
         curr = curr.left
     }

     // 弹出栈顶——这是当前中序位置
     curr = stack.pop()

     // 处理当前节点
     if (prev !== null) {
         minDiff = min(minDiff, curr.val - prev.val)
     }
     prev = curr

     // 转向右子树
     curr = curr.right
   }

3. 返回 minDiff
```

### 为什么不需要其他解法？

| 尝试 | 为什么不行 |
|------|-----------|
| 暴力枚举所有节点对 | O(n²) 时间，n 可达 10⁴，不可行 |
| 存到数组后排序再比较 | 如果输入不是 BST，需要 O(n log n) 排序；但题目给了 BST，中序就是天然排序，不需要额外排序 |
| 存到数组后遍历比较 | 可以，空间 O(n)。但维护 `prev` 变量可以在 O(h) 空间内完成，更优 |
| 只比较父子节点 | 错误！BST 的最小差值不一定来自父子节点。例如 `[10,5,15,1,8]`，1 和 8 不直接相邻但在中序中相邻 |

> 💡 **唯一推荐解法**：递归中序遍历 + `prev` 指针。这是最简洁、最符合直觉的写法。

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

// ============ 解法一：递归中序遍历 ============

/**
 * @param {TreeNode} root
 * @return {number}
 */
var getMinimumDifference = function(root) {
    let minDiff = Infinity;
    let prev = null;

    function inorder(node) {
        if (node === null) return;

        // 左
        inorder(node.left);

        // 根：计算与前驱的差值
        if (prev !== null) {
            minDiff = Math.min(minDiff, node.val - prev.val);
        }
        prev = node;

        // 右
        inorder(node.right);
    }

    inorder(root);
    return minDiff;
};

// ============ 解法二：迭代栈中序遍历 ============

/**
 * @param {TreeNode} root
 * @return {number}
 */
var getMinimumDifferenceIterative = function(root) {
    let minDiff = Infinity;
    let prev = null;
    const stack = [];
    let curr = root;

    while (curr !== null || stack.length > 0) {
        // 一路向左走到底
        while (curr !== null) {
            stack.push(curr);
            curr = curr.left;
        }

        // 弹出栈顶
        curr = stack.pop();

        // 计算与前驱的差值
        if (prev !== null) {
            minDiff = Math.min(minDiff, curr.val - prev.val);
        }
        prev = curr;

        // 转向右子树
        curr = curr.right;
    }

    return minDiff;
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

# ============ 解法一：递归中序遍历 ============

class Solution:
    def getMinimumDifference(self, root: TreeNode) -> int:
        self.min_diff = float('inf')
        self.prev = None

        def inorder(node: TreeNode) -> None:
            if node is None:
                return

            # 左
            inorder(node.left)

            # 根：计算与前驱的差值
            if self.prev is not None:
                self.min_diff = min(self.min_diff, node.val - self.prev.val)
            self.prev = node

            # 右
            inorder(node.right)

        inorder(root)
        return self.min_diff


# ============ 解法二：迭代栈中序遍历 ============

class Solution:
    def getMinimumDifference(self, root: TreeNode) -> int:
        min_diff = float('inf')
        prev = None
        stack = []
        curr = root

        while curr is not None or len(stack) > 0:
            # 一路向左走到底
            while curr is not None:
                stack.append(curr)
                curr = curr.left

            # 弹出栈顶
            curr = stack.pop()

            # 计算与前驱的差值
            if prev is not None:
                min_diff = min(min_diff, curr.val - prev.val)
            prev = curr

            # 转向右子树
            curr = curr.right

        return min_diff
```

---

## 逐步推演

### 完整推演（示例 1）

以 `root = [4, 2, 6, 1, 3]` 为例，展示中序遍历的完整过程：

```
树结构：

        4
       / \
      2   6
     / \
    1   3

中序访问顺序: 1 → 2 → 3 → 4 → 6
```

---

**第一步：递归到 1（最深左叶子）**

```
inorder(1):
  左: inorder(null) → 直接返回
  根: prev = null → 不计算差值
      prev = node(1)
  右: inorder(null) → 直接返回

状态: minDiff = ∞, prev = 1
```

**第二步：回到 2，处理节点 2**

```
inorder(2):
  根: prev = 1 → diff = 2 - 1 = 1
      minDiff = min(∞, 1) = 1 ⭐
      prev = node(2)
  右: inorder(3)

状态: minDiff = 1, prev = 2
```

**第三步：递归到 3，处理节点 3**

```
inorder(3):
  左: inorder(null) → 返回
  根: prev = 2 → diff = 3 - 2 = 1
      minDiff = min(1, 1) = 1（不变）
      prev = node(3)
  右: inorder(null) → 返回

状态: minDiff = 1, prev = 3
```

**第四步：回到 4，处理节点 4**

```
inorder(4):
  根: prev = 3 → diff = 4 - 3 = 1
      minDiff = min(1, 1) = 1（不变）
      prev = node(4)
  右: inorder(6)

状态: minDiff = 1, prev = 4
```

**第五步：递归到 6，处理节点 6**

```
inorder(6):
  左: inorder(null) → 返回
  根: prev = 4 → diff = 6 - 4 = 2
      minDiff = min(1, 2) = 1（不变）
      prev = node(6)
  右: inorder(null) → 返回

状态: minDiff = 1, prev = 6
```

**最终结果：minDiff = 1**

```
所有相邻差值: [1, 1, 1, 2]
最小值: 1 ✅

        [4]
        / \
      [2]   [6]
      / \
    [1]   [3]

   中序：1 → 2 → 3 → 4 → 6
   差值：  1    1    1    2
```

### 如果你把"任意两节点"理解成必须连着走

```
❌ 错误理解： 只比较父子节点 → 差值集合 {2, 4, 1, 1} → min = 1（碰巧也是 1）

但如果树是:  [10, 5, 15, 1, 8]

         10
        /  \
       5    15
      / \
     1   8

父子差值: |10-5|=5, |10-15|=5, |5-1|=4, |5-8|=3 → min = 3
中序差值: 5-1=4, 8-5=3, 10-8=2, 15-10=5 → min = 2 ⭐正确

正确答案是 2（8 和 10 的差值），而只看父子会得出 3，
因为 8 和 10 在树中不是父子关系，但在中序序列中是相邻的！
```

> 💡 **关键教训**：在 BST 中，"中序相邻"不等于"树中相邻"。必须用中序遍历来找差值，不能只比父子节点。

---

## 复杂度分析

| 项目 | 递归版 | 迭代版 |
|------|--------|--------|
| **时间复杂度** | O(n) | O(n) |
| **空间复杂度** | O(h)（递归栈） | O(h)（显式栈） |
| **平衡树** | O(log n) | O(log n) |
| **退化链表** | O(n) | O(n) |

> - **时间复杂度**：每个节点被访问恰好一次，在节点内部做 O(1) 的比较和赋值操作，总复杂度严格 O(n)
> - **空间复杂度**：递归栈/显式栈的深度 = 树的高度 h。平衡树 O(log n)，退化链表 O(n)。注意：如果写"先存数组再遍历"，空间会变成 O(n)，不符合最优解的要求
> - **递归 vs 迭代**：递归版更简洁，迭代版更安全（不会栈溢出）。对于 n ≤ 10⁴ 的规模，递归版完全够用

---

## 举一反三

### 本题在 BST 中序遍历体系中的位置

```
LeetCode 94 二叉树的中序遍历（Easy）
  → 递归版 和 迭代版（栈模拟）
  ↓
LeetCode 530 二叉搜索树的最小绝对差（Easy）⭐ 本题
  → 中序遍历过程中维护 prev，计算相邻差值的最小值
  ↓
LeetCode 98 验证二叉搜索树（Medium）
  → 中序遍历过程中维护 prev，检查是否严格递增（prev < curr）
  ↓
LeetCode 501 二叉搜索树中的众数（Easy）
  → 中序过程中维护计数，找出现次数最多的值
  ↓
LeetCode 99 恢复二叉搜索树（Medium）
  → 中序遍历找到两个被交换的节点，用 prev/curr 比较定位异常点
```

**共同模式**：所有涉及"BST 中节点的顺序关系"的问题，都可以通过**中序遍历 + 维护前驱**来解决。核心代码结构几乎不变，变的只是"在访问当前节点时做什么"。

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [783. 二叉搜索树节点最小距离](https://leetcode.com/problems/minimum-distance-between-bst-nodes/) | 与本题完全相同 | 孪生题，连示例都一样 |
| [98. 验证二叉搜索树](https://leetcode.com/problems/validate-binary-search-tree/) | 中序遍历 + prev 检查递增 | 本题是求 min(diff)，98 是检查所有 diff > 0 |
| [501. 二叉搜索树中的众数](https://leetcode.com/problems/find-mode-in-binary-search-tree/) | 中序遍历 + 计数统计 | 同样维护 prev，但变成统计 prev === curr 的次数 |
| [173. 二叉搜索树迭代器](https://leetcode.com/problems/binary-search-tree-iterator/) | 栈模拟中序，均摊 O(1) | 同一条"一路向左"逻辑，本题的迭代版就是 173 的微缩版 |
| [99. 恢复二叉搜索树](https://leetcode.com/problems/recover-binary-search-tree/) | 中序找异常相邻对 | 进阶——用中序找到"非递增"的位置定位被交换的节点 |
| [230. 二叉搜索树中第 K 小的元素](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | 中序计数 | 同样中序遍历，但计数到第 k 个就停止 |

### 通用模板：BST 中序遍历 + 前驱处理

下面这个模板可以套用到本系列绝大多数题目中：

```javascript
// BST 中序遍历 + 前驱处理 通用模板
function bstInorderTemplate(root) {
    let prev = null;   // 前驱节点
    // 各种需要的变量，如 minDiff, count, result 等

    function inorder(node) {
        if (node === null) return;

        inorder(node.left);     // 递归左子树

        // ===== 处理当前节点 =====
        if (prev !== null) {
            // 在这里写你需要的逻辑
            // 例如：比较 node.val 和 prev.val
        }
        prev = node;
        // =========================

        inorder(node.right);    // 递归右子树
    }

    inorder(root);
    return /* 你要的结果 */;
}
```

> 只要把中间的"处理逻辑"换成题目需要的，就能覆盖 530、98、501、99 等一大类题目。

### 如果树不是 BST 呢？

如果题目把"二叉搜索树"改为"普通二叉树"，中序遍历就不再得到有序序列。两种处理方式：

1. **收集所有值 → 排序 → 遍历相邻差值**：O(n log n) 时间，O(n) 空间
2. **任意两节点最小差值**：这是另一个完全不同的问题，没有比 O(n²) 暴力更优的通用解法

> 这就是为什么题目强调"BST"——**利用 BST 结构省掉排序的 O(n log n)**，直接 O(n) 解决。

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **530 二叉搜索树的最小绝对差** | 🟢 Easy | 中序序列递增，最小差值在相邻元素间 | 递归中序遍历 + prev 前驱 |

这道题的"简单"在于思路极其直接，但包含了 BST 类题目的核心范式：

| 要点 | 说明 |
|------|------|
| **BST 的中序序列** | 严格递增——这是 BST 最核心的性质之一 |
| **最小差值** | 在递增序列中，最小差值必定出现在相邻元素之间 |
| **prev 指针** | 不需要存整个数组，一个 prev 变量就能记录前驱，空间从 O(n) 降到 O(h) |
| **代码结构** | 标准的中序遍历框架，在"访问根"的位置加两行逻辑即可 |

**三步记忆法：**

1. **中序遍历 BST** = 得到一个递增序列
2. **相邻差值取 min** = 维护 `prev`，每次计算 `curr.val - prev.val`，更新最小值
3. **空间 O(h)** = 不存数组，中序遍历过程中边遍历边计算

> **记住一句话：BST 的最小绝对差 = 中序遍历过程中，每个节点和它的前驱的差值的最小值。中序遍历让 BST 的值"排好队"，最小差值只可能在相邻的两个人之间出现。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 98. 验证二叉搜索树](https://leetcode.com/problems/validate-binary-search-tree/)（同模板：prev 检查递增） | [LeetCode 501. 二叉搜索树中的众数](https://leetcode.com/problems/find-mode-in-binary-search-tree/)（同模板：prev 计数） | [LeetCode 173. 二叉搜索树迭代器](./leetcode-173-binary-search-tree-iterator.md)（迭代版中序详解） | 二叉搜索树系列持续更新中，关注不迷路。
