---
title: LeetCode 124. 二叉树中的最大路径和
tags: [二叉树, DFS, 递归, 后序遍历, 最大增益]
difficulty: Hard
category: 二叉树
date: 2026-06-14
---

# LeetCode 124. 二叉树中的最大路径和 —— 从局部贡献到全局最优的后序遍历艺术

## 前言

在二叉树问题中，大多数题目都是"自顶向下"思考——从根节点出发，逐层传递信息。但 LeetCode 124 **二叉树中的最大路径和**（Binary Tree Maximum Path Sum）打破了这个惯性：**路径可以经过任意节点，甚至可以完全局限在左子树或右子树中**。

这意味着我们不能只盯着根节点，而是要站在每个节点上，思考一个关键问题：

> **"如果这条最大路径经过我，那么左子树能给我贡献多少？右子树能给我贡献多少？"**

一旦把视角切换到"以每个节点为拐点"，这道 Hard 题就变成了一个简洁的后序遍历。本文将用**递归 DFS + 全局变量**的思路一层层拆解，附 JavaScript/Python 双版本代码 + 完整递归推演。

> **前置阅读**：建议先掌握二叉树的后序遍历，理解"自底向上"的信息传递方式。本题是后序遍历思想的经典应用。

---

## 问题描述

### LeetCode 124. Binary Tree Maximum Path Sum（二叉树中的最大路径和）

> **路径** 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。同一个节点在一条路径序列中 **至多出现一次**。该路径 **至少包含一个** 节点，且不一定经过根节点。
>
> **路径和** 是路径中各节点值的总和。
>
> 给你一个二叉树的根节点 `root`，返回其 **最大路径和**。

**示例 1：**

```
输入：root = [1,2,3]
输出：6
解释：最优路径是 2 → 1 → 3，路径和为 2 + 1 + 3 = 6
```

**图示：**

```
        1
       / \
      2   3

最优路径: 2 → 1 → 3，值为 6
```

**示例 2：**

```
输入：root = [-10,9,20,null,null,15,7]
输出：42
解释：最优路径是 15 → 20 → 7，路径和为 15 + 20 + 7 = 42
```

**图示：**

```
       -10
       /  \
      9   20
         /  \
        15   7

最优路径: 15 → 20 → 7，值为 42
注意：这条路径完全不经过根节点 -10！
```

**约束条件：**
- 树中节点数目范围是 `[1, 3 * 10^4]`
- `-1000 <= Node.val <= 1000`

---

## 核心思想

### 问题的本质：每个节点都有两个角色

这道题的难点在于**路径可以是任意形状**。但仔细分析，任意一条合法路径都有一个"最高点"（离根最近的节点）。对于每个节点，我们考虑两种角色：

```
角色一：作为"经过点"（路径的拐点/最高点）
  → 路径 = 左子树贡献 + 当前节点值 + 右子树贡献
  → 这是我们用来更新全局最大值的候选

角色二：作为"路径的一部分"（向上传递贡献）
  → 当前节点只能选择左子树或右子树中的一个方向
  → 传给父节点的值 = 当前节点值 + max(左贡献, 右贡献, 0)
  → 如果左、右贡献都是负数，就只传当前节点值自己
```

```
以节点为"拐点"的路径示意：

        /···\
       /     \
      /       \
     ← 左子树 →  ← 右子树 →
              \
               \── 当前节点为最高点：
                    路径 = 左子树往下走到底 + 当前节点 + 右子树往下走到底
                          
              注意：以当前节点为拐点时，左右子树的路径都必须
              向下延伸（不能有分叉），因为路径在二叉树中是一条链
```

### 数据结构设计

每个递归调用返回一个值：**以当前节点为起点的最大单侧路径和**。

```
maxGain(node) 的定义：
  从 node 出发，沿着树向下走（只能走左或走右），
  能获得的最大路径和。
  
  数学表达：
    maxGain(node) = node.val + max(
      0,                    // 左右子树贡献都是负数，只取当前节点
      maxGain(node.left),   // 走左边
      maxGain(node.right)   // 走右边
    )
  
  注意：如果子树的贡献是负数，我们宁可不要（取 0），
  因为"不包含"比"包含一个负数"更优。

全局变量 maxSum：
  在计算每个节点的 maxGain 时，同时计算：
    candidate = maxGain(node.left) + node.val + maxGain(node.right)
    maxSum = max(maxSum, candidate)
  即以当前节点为拐点的完整路径和。
```

**整体流程示意：**

```
后序遍历，自底向上：

         root
        /    \
       L      R        ← 先递归拿到左右子树的 maxGain
      / \    / \
     ...  ...  ...

对于节点 L：
  leftGain  = max(0, maxGain(L.left))    ← 左子树的贡献
  rightGain = max(0, maxGain(L.right))   ← 右子树的贡献
  candidate = leftGain + L.val + rightGain  ← 以 L 为拐点的完整路径
  更新 maxSum = max(maxSum, candidate)
  返回 L.val + max(leftGain, rightGain)    ← L 能为父节点提供的最大贡献
```

> 💡 **关键洞察**：`maxGain` 返回的是"单侧"最大贡献（只能选一条路走下去），而全局 `maxSum` 更新时用的是"双侧"（左+自己+右）。这个不对称性正是本题的核心——**子节点只能给父节点提供一条路，但子节点自己可以同时拥有左右两条路**。

---

## 思路分析

### 解法：递归 DFS（后序遍历）⭐ 唯一推荐

**核心思路**：自底向上计算每个节点的"最大单侧贡献"，并在每个节点处检查"以该节点为拐点的完整路径"是否能刷新全局最大值。

```
算法框架：

1. 初始化全局变量 maxSum = -Infinity（因为节点值可能都是负数）

2. 定义递归函数 maxGain(node):
   a. 递归基：if (node === null) return 0
   
   b. 递归计算左右子树的贡献：
      leftGain  = max(0, maxGain(node.left))   ← 负数就舍弃，取 0
      rightGain = max(0, maxGain(node.right))  ← 同上
   
   c. 以当前节点为拐点计算完整路径和：
      currentPathSum = leftGain + node.val + rightGain
      maxSum = max(maxSum, currentPathSum)
   
   d. 返回当前节点能向上提供的最大贡献：
      return node.val + max(leftGain, rightGain)

3. 调用 maxGain(root)
4. 返回 maxSum
```

> 💡 **为什么 `max(0, ...)` 是关键？** 因为路径可以不包含某侧子树。如果左子树带来的最大贡献是 -500，那"不走左边"就是更好的选择（相当于贡献为 0）。这保证了每个节点的选择都是最优的。

### 为什么不需要其他解法？

这道题和大多数 Hard 题不同，它没有多种可行思路——**后序遍历 + 全局最大值**几乎是唯一的正解框架：

| 尝试 | 为什么不行 |
|------|-----------|
| 暴力枚举所有路径 | 路径数量是指数级，O(2ⁿ) 不可行 |
| 自顶向下 DP | 无法预知子树的最优贡献，必须先算子树 |
| 两次 DFS | 不需要，一次后序遍历就能同时完成"计算贡献"和"更新答案" |
| BFS / 层序遍历 | 路径不一定在同一层，BFS 无法处理 |

这道题的难点在于**想到**这个框架，而不是在多种解法中做选择。一旦理解了"以每个节点为拐点"的视角，代码不到 20 行。

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
 * @return {number}
 */
var maxPathSum = function(root) {
    // 全局最大值：初始化为 -Infinity，处理全负数节点的情况
    let maxSum = -Infinity;

    /**
     * 后序遍历：返回以 node 为起点的最大单侧路径和
     * @param {TreeNode} node
     * @return {number}
     */
    function maxGain(node) {
        // 递归基：空节点贡献为 0
        if (node === null) return 0;

        // 递归计算左右子树的贡献（负数直接舍弃，取 0）
        const leftGain = Math.max(0, maxGain(node.left));
        const rightGain = Math.max(0, maxGain(node.right));

        // 以当前节点为拐点的完整路径和
        const currentPathSum = leftGain + node.val + rightGain;

        // 更新全局最大值
        maxSum = Math.max(maxSum, currentPathSum);

        // 返回当前节点能向上提供的最大贡献（只能选一条路往下走）
        return node.val + Math.max(leftGain, rightGain);
    }

    maxGain(root);
    return maxSum;
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

class Solution:
    def maxPathSum(self, root: TreeNode) -> int:
        # 全局最大值：初始化为负无穷
        self.max_sum = float('-inf')

        def max_gain(node: TreeNode) -> int:
            """
            后序遍历：返回以 node 为起点的最大单侧路径和
            """
            # 递归基
            if node is None:
                return 0

            # 递归计算左右子树的贡献（负数舍弃）
            left_gain = max(0, max_gain(node.left))
            right_gain = max(0, max_gain(node.right))

            # 以当前节点为拐点的完整路径和
            current_path_sum = left_gain + node.val + right_gain

            # 更新全局最大值
            self.max_sum = max(self.max_sum, current_path_sum)

            # 返回当前节点向上提供的最大贡献
            return node.val + max(left_gain, right_gain)

        max_gain(root)
        return self.max_sum
```

---

## 逐步推演

### 完整推演（示例 2）

以 `root = [-10, 9, 20, null, null, 15, 7]` 为例，展示递归过程：

```
树结构：

        -10
       /   \
      9    20
          /  \
         15   7

所有节点值: -10, 9, 20, 15, 7
```


**第一步：递归到叶子节点 9**

```
max_gain(9):
  9.left  = null → leftGain  = max(0, 0) = 0
  9.right = null → rightGain = max(0, 0) = 0

  currentPathSum = 0 + 9 + 0 = 9
  更新 maxSum = max(-∞, 9) = 9

  返回: 9 + max(0, 0) = 9
  （9 能为父节点提供的最大贡献是 9）
```


**第二步：递归到叶子节点 15**

```
max_gain(15):
  leftGain = 0, rightGain = 0

  currentPathSum = 0 + 15 + 0 = 15
  更新 maxSum = max(9, 15) = 15

  返回: 15 + 0 = 15
```


**第三步：递归到叶子节点 7**

```
max_gain(7):
  leftGain = 0, rightGain = 0

  currentPathSum = 0 + 7 + 0 = 7
  更新 maxSum = max(15, 7) = 15（不变）

  返回: 7 + 0 = 7
```


**第四步：递归到节点 20**

```
max_gain(20):
  20.left  → max_gain(15) 返回 15 → leftGain  = max(0, 15) = 15
  20.right → max_gain(7)  返回 7  → rightGain = max(0, 7)  = 7

  currentPathSum = 15 + 20 + 7 = 42
  更新 maxSum = max(15, 42) = 42 ⭐ 最大路径和！

  返回: 20 + max(15, 7) = 20 + 15 = 35
  （20 能为父节点提供的最大贡献：走 20 → 15，不去 7）
```


**第五步：递归到根节点 -10**

```
max_gain(-10):
  -10.left  → max_gain(9)  返回 9  → leftGain  = max(0, 9)  = 9
  -10.right → max_gain(20) 返回 35 → rightGain = max(0, 35) = 35

  currentPathSum = 9 + (-10) + 35 = 34
  更新 maxSum = max(42, 34) = 42（不变）

  返回: -10 + max(9, 35) = -10 + 35 = 25
  （不需要用到这个返回值，因为已经到了根节点）
```


**最终结果：maxSum = 42**

```
最优路径: 15 → 20 → 7

        -10
       /   \
      9    [20]     ← 拐点
          /    \
        [15]   [7]  ← 路径两端

这条路径以 20 为拐点，完全不经过根节点 -10
```

> 💡 **核心观察**：注意节点 20 的递归返回值是 `20 + max(15, 7) = 35`，而全局最大值用的是 `15 + 20 + 7 = 42`。**返回值是"单侧最好"，全局更新是"双侧之和"**——这正是本题最关键的设计。

### 全负数节点的情况（边界条件验证）

以 `root = [-3]` 为例：

```
max_gain(-3):
  leftGain = 0, rightGain = 0

  currentPathSum = 0 + (-3) + 0 = -3
  更新 maxSum = max(-∞, -3) = -3

  返回: -3 + 0 = -3

结果: -3 ✅（路径只有一个节点，值就是 -3）
```

以 `root = [-2, -1]`（即 -2 的左子节点为 -1）为例：

```
        -2
       /
     -1

max_gain(-1):
  leftGain = 0, rightGain = 0
  currentPathSum = -1
  更新 maxSum = max(-∞, -1) = -1
  返回: -1

max_gain(-2):
  leftGain  = max(0, -1) = 0  ← 负数贡献被舍弃！
  rightGain = max(0, 0) = 0

  currentPathSum = 0 + (-2) + 0 = -2
  更新 maxSum = max(-1, -2) = -1

结果: -1 ✅（最优路径只包含节点 -1）
```

> 💡 如果不用 `max(0, ...)` 舍弃负数贡献，`leftGain` 会取 -1，导致 `currentPathSum = -1 + (-2) + 0 = -3`，错误地选了一条更差的路径。

---

## 复杂度分析

| 项目 | 分析 |
|------|------|
| **时间复杂度** | O(n)，每个节点被访问恰好一次 |
| **空间复杂度** | O(h)，其中 h 是树的高度。递归栈深度最坏 O(n)（退化成链表），平均 O(log n)（平衡树） |

> - **时间复杂度**：后序遍历每个节点只访问一次，在节点内部做常数次 `max` 和加法操作，所以是严格的 O(n)
> - **空间复杂度**：递归调用栈的深度等于树的高度。在平衡树中为 O(log n)，在退化成链表的极端情况下为 O(n)。如果要求严格 O(1) 额外空间，可以使用 Morris 遍历后序版，但实现复杂度远高于收益

---

## 举一反三

### 本题在二叉树递归体系中的位置

```
LeetCode 104 二叉树的最大深度（Easy）
  → 后序遍历：当前深度 = 1 + max(左深度, 右深度)
  ↓
LeetCode 543 二叉树的直径（Easy / Medium）
  → 后序遍历：当前直径 = 左深度 + 右深度，返回 1 + max(左深度, 右深度)
  ↓
LeetCode 124 二叉树中的最大路径和（Hard）⭐ 本题
  → 后序遍历：当前路径 = 左贡献 + 当前值 + 右贡献，返回 当前值 + max(左贡献, 右贡献)
  ↓
LeetCode 687 最长同值路径（Medium）
  → 后序遍历 + 条件传递：如果子节点值等于当前值才传递，否则贡献为 0
```

四道题的**递归框架完全相同**：在后序遍历中，用左右子树的返回值更新全局最优解，然后返回当前节点能向上提供的"单侧贡献"。区别只在于"贡献"的定义不同。

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [543. 二叉树的直径](https://leetcode.com/problems/diameter-of-binary-tree/) | 后序求深度，最大直径 = 左深度 + 右深度 | 结构几乎一致——返回值取 `max`，全局更新取"左+右"。把"深度"换成"路径和"就是 124 |
| [687. 最长同值路径](https://leetcode.com/problems/longest-univalue-path/) | 后序 + 条件贡献：值相等才传递 | 在 124 的基础上增加了一个"条件"——子节点值等于当前节点值才计入贡献 |
| [112. 路径总和](https://leetcode.com/problems/path-sum/) | 自顶向下：检查是否存在根→叶子的目标和 | 同样是"路径和"但方向相反（自顶向下 vs 自底向上），且路径必须从根到叶子 |
| [437. 路径总和 III](https://leetcode.com/problems/path-sum-iii/) | 前缀和 + 回溯：统计所有路径条数 | 路径和问题的计数版，用哈希表加速 |

### 后序遍历的通用框架

反复出现的模式提炼：

```
后序递归模板 = 以下三步：

1. 递归基：空节点返回 0（或适合题意的默认值）

2. 递归左右：
   left  = 递归函数(node.left)
   right = 递归函数(node.right)

3. 双步处理（核心）：
   a. 用 left 和 right 组合成"以当前节点为中心"的完整答案
      更新全局最优 → ans = max(ans, left + node.val + right)
   b. 计算当前节点向上传递的"单侧贡献"
      返回 node.val + max(left, right)（或其变体）
```

这个框架适用于一大类"在树上找最大路径/直径/同值链"的题目。一旦你识别出题目属于这个家族，直接套用模板就能快速写出代码。

> **一句话总结**：二叉树中的最大路径和 = 后序遍历时，每个节点同时计算两件事——"以我为最高点的完整路径和"（左+我+右，用于更新全局答案）和"我能给父节点提供的最优单侧贡献"（我+max(左,右)，作为返回值）。`max(0, 子树贡献)` 是保证负数子树不被计入的关键操作。

### 延伸思考

在实际面试中，当你用上述思路写完代码后，面试官可能会追问：

1. **"为什么用后序遍历而不是前序？"** → 因为必须先知道左右子树的贡献，才能计算当前节点的贡献。前序遍历时子树的贡献还是未知的。

2. **"如果节点值全是负数会怎样？"** → `max(0, gain)` 会让所有子树贡献为 0，但 `currentPathSum = 0 + node.val + 0 = node.val` 仍然取到了"单个节点"的情况，而 `maxSum` 初始化为 `-Infinity` 保证最终会取到最大的那个负数节点值。

3. **"能不能不用全局变量？"** → 可以，改为返回值是一个 `[maxGain, maxSum]` 的元组，但会让代码复杂性增加。对于这种"递归过程中需要维护全局最优"的问题，全局变量（或类的实例属性）是最自然、最清晰的写法。

4. **"如果路径定义是'必须从根到叶子'呢？"** → 那就变成 LeetCode 112 路径总和，变成自顶向下的 DFS，完全不同的思路。

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **124 二叉树中的最大路径和** | 🔴 Hard | 后序遍历 + 单侧贡献 + 全局最优 | 递归 DFS |

这道题的"难"不在于代码量（不到 20 行），而在于三点：

| 难点 | 解析 |
|------|------|
| **视角转变** | 从"找一条路径"变成"每个节点做拐点"，这是思路突破的关键 |
| **双重角色** | 同一个递归函数要同时完成"更新全局答案"和"返回单侧贡献"两件事 |
| **负数处理** | `max(0, ...)` 看似简单，但理解"为什么要舍弃负数子树"需要想明白"路径可以不包含任何子树" |

**三步记忆法：**

1. **递归函数返回**：以当前节点为起点，沿着一条路往下走的最大路径和（`node.val + max(0, leftGain, rightGain)`）
2. **全局答案更新**：以当前节点为拐点，左+自己+右的完整路径和（`leftGain + node.val + rightGain`）
3. **负数处理**：左右子树贡献先和 0 取 `max`，确保不拖累全局答案

> **记住一句话：二叉树中的最大路径和 = 站在每个节点上问——"如果最佳路径以我为最高点，左子树最多能给我多少？右子树最多能给我多少？"。然后取所有节点中的最大值。负数直接舍弃，不如图中不画。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 543. 二叉树的直径](https://leetcode.com/problems/diameter-of-binary-tree/)（同框架简化版） | [LeetCode 687. 最长同值路径](https://leetcode.com/problems/longest-univalue-path/)（条件贡献版） | [LeetCode 112. 路径总和](https://leetcode.com/problems/path-sum/)（自顶向下路径和） | 二叉树系列持续更新中，关注不迷路。
