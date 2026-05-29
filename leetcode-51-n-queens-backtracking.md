---
title: LeetCode 51/52. N 皇后
tags: [回溯, DFS, 剪枝, 位运算]
difficulty: Hard
category: 回溯
date: 2026-05-25
---

## 前言

回溯算法（Backtracking）是 LeetCode 面试中最高频的算法题型之一，它的本质是**在决策树上进行深度优先搜索（DFS）+ 剪枝**——当你需要在一系列选择中寻找所有可行解，且每一步的选择会影响到后续路径时，就该想到回溯。

本文将用一道经典中的经典——**N 皇后问题**（LeetCode 51/52），庖丁解牛式地讲透回溯算法的思维模型和代码模板，让你做到"做一道题，会一类题"。

---

## 问题描述

### LeetCode 51. N-Queens（返回所有解）

> 按照国际象棋的规则，皇后可以攻击同一行、同一列、同一斜线上的棋子。
>
> **n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n × n` 的棋盘上，并且使皇后彼此之间不能相互攻击。
>
> 给你一个整数 `n`，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 n 皇后问题的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

```
示例：
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],
      ["..Q.","Q...","...Q",".Q.."]]
```

解释：4 皇后只有两种解法（八皇后的缩小版）：

```
. Q . .    . . Q .
. . . Q    Q . . .
Q . . .    . . . Q
. . Q .    . Q . .
```

### LeetCode 52. N-Queens II（仅返回解的数量）

52 题与 51 题完全一致，唯一的区别是**只要求返回解的个数**，不需要返回具体棋盘。在 51 题的基础上稍加改动即可。

---

## 回溯算法核心思想

回溯可以理解成一个人在迷宫里的走法：

1. 在每一个岔路口做**选择**
2. 走进去，如果发现走不通，就**撤销选择**，退回上一个路口
3. 换一条路继续尝试

用代码模板表达就是：

```javascript
function backtrack(当前状态, 选择列表) {
    if (满足结束条件) {
        记录答案;
        return;
    }

    for (let 选择 of 选择列表) {
        做选择;
        backtrack(新状态, 新的选择列表);
        撤销选择;  // 这就是"回溯"的由来
    }
}
```

**"走进去，退出来"** 六个字概括回溯。

---

## 思路分析：N 皇后问题

N 皇后的约束有三个：
1. **同行不能有**两个皇后 → 我们每行只放一个，天然满足
2. **同列不能有**两个皇后 → 需要记录哪些列被占用了
3. **同对角线不能有**两个皇后 → 需要记录两条对角线

关键观察：对于 `(row, col)` 位置的皇后：
- **主对角线**（从左上到右下）满足 `row - col` 为常数
- **副对角线**（从右上到左下）满足 `row + col` 为常数

所以我们只需要三个集合（或布尔数组）来记录冲突信息：
- `cols[row]`：第 col 列是否已有皇后
- `diag1[row - col + n - 1]`：主对角线（加偏移避免负数下标）
- `diag2[row + col]`：副对角线

### 搜索策略

**逐行放置**：从第 0 行开始，每一行尝试所有列，检查当前列和对角线是否可用。如果可用则放置皇后，递归到下一行；不可用则跳过（剪枝）。

---

## 代码实现

### JavaScript 版本

```javascript
/**
 * @param {number} n
 * @return {string[][]}
 */
var solveNQueens = function(n) {
    const res = [];
    // 构建棋盘初始状态
    const board = Array.from({ length: n }, () => Array(n).fill('.'));

    const cols = new Array(n).fill(false);       // 列占用
    const diag1 = new Array(2 * n).fill(false);  // 主对角线 (row - col + n - 1)
    const diag2 = new Array(2 * n).fill(false);  // 副对角线 (row + col)

    function backtrack(row) {
        // 所有行都放完了，找到一个解
        if (row === n) {
            res.push(board.map(r => r.join('')));
            return;
        }

        for (let col = 0; col < n; col++) {
            const d1 = row - col + n - 1;  // 主对角线索引
            const d2 = row + col;          // 副对角线索引

            // 剪枝：列或对角线已被占用
            if (cols[col] || diag1[d1] || diag2[d2]) continue;

            // 做选择
            board[row][col] = 'Q';
            cols[col] = true;
            diag1[d1] = true;
            diag2[d2] = true;

            backtrack(row + 1);

            // 撤销选择
            board[row][col] = '.';
            cols[col] = false;
            diag1[d1] = false;
            diag2[d2] = false;
        }
    }

    backtrack(0);
    return res;
};
```

```python
def solve_n_queens(n: int) -> list[list[str]]:
    """返回所有 N 皇后解"""
    res = []
    board = [['.'] * n for _ in range(n)]
    cols = [False] * n
    diag1 = [False] * (2 * n)  # 主对角线 (row - col + n - 1)
    diag2 = [False] * (2 * n)  # 副对角线 (row + col)

    def backtrack(row: int) -> None:
        if row == n:
            res.append([''.join(r) for r in board])
            return

        for col in range(n):
            d1 = row - col + n - 1
            d2 = row + col
            if cols[col] or diag1[d1] or diag2[d2]:
                continue

            # 做选择
            board[row][col] = 'Q'
            cols[col] = diag1[d1] = diag2[d2] = True

            backtrack(row + 1)

            # 撤销选择
            board[row][col] = '.'
            cols[col] = diag1[d1] = diag2[d2] = False

    backtrack(0)
    return res
```

### LeetCode 52（只计数）

```javascript
/**
 * @param {number} n
 * @return {number}
 */
var totalNQueens = function(n) {
    let count = 0;
    const cols = new Array(n).fill(false);
    const diag1 = new Array(2 * n).fill(false);
    const diag2 = new Array(2 * n).fill(false);

    function backtrack(row) {
        if (row === n) {
            count++;
            return;
        }

        for (let col = 0; col < n; col++) {
            const d1 = row - col + n - 1;
            const d2 = row + col;
            if (cols[col] || diag1[d1] || diag2[d2]) continue;

            cols[col] = true; diag1[d1] = true; diag2[d2] = true;
            backtrack(row + 1);
            cols[col] = false; diag1[d1] = false; diag2[d2] = false;
        }
    }

    backtrack(0);
    return count;
};
```

```python
def total_n_queens(n: int) -> int:
    """返回解的数量"""
    count = 0
    cols = [False] * n
    diag1 = [False] * (2 * n)
    diag2 = [False] * (2 * n)

    def backtrack(row: int) -> None:
        nonlocal count
        if row == n:
            count += 1
            return

        for col in range(n):
            d1 = row - col + n - 1
            d2 = row + col
            if cols[col] or diag1[d1] or diag2[d2]:
                continue

            cols[col] = diag1[d1] = diag2[d2] = True
            backtrack(row + 1)
            cols[col] = diag1[d1] = diag2[d2] = False

    backtrack(0)
    return count
```

---

## 复杂度分析

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | O(N!) | 第一行有 N 个选择，第二行最多 N-1 个，第三行 N-2 个……呈阶乘下降。配合剪枝后实际运行远小于 N! |
| 空间复杂度 | O(N²) | 棋盘本身占用 N²，递归调用栈深度为 N |

实际上 N 皇后的解空间增长非常快（N=8 时有 92 个解，N=12 时有 14200 个解），但回溯的剪枝效率很高，实战中 N 一般不超过 15。

---

## 优化方向

### 1. 位运算优化（Bitmask）

用整数的二进制位代替布尔数组，将空间压缩到 O(N)，同时位运算的常数极小，是最高效的实现方式：

```javascript
// 核心逻辑（n <= 31 时可用，JS 位运算操作 32 位有符号整数）
function backtrack(colMask, diag1Mask, diag2Mask, row, n) {
    if (row === n) { count++; return; }
    // 当前行所有可用的列（二进制位为 1 表示可用）
    let available = ((1 << n) - 1) & ~(colMask | diag1Mask | diag2Mask);
    while (available) {
        const pick = available & -available;     // 取最低位的 1
        available &= available - 1;              // 移除最低位的 1
        backtrack(
            colMask | pick,
            (diag1Mask | pick) << 1,
            (diag2Mask | pick) >> 1,
            row + 1,
            n
        );
    }
}
```

### 2. 对称性剪枝

棋盘关于中心轴对称，找到一个解后可以通过旋转和镜像对称性直接生成其他解，但实现较复杂，面试时用标准回溯足够了。

---

## 举一反三

理解了 N 皇后，以下题目都可以用相同的回溯模板解决：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 46. 全排列** | 最基础的排列树回溯，N 皇后的简化版 |
| **LeetCode 39. 组合总和** | 元素可重复选，注意剪枝排序 |
| **LeetCode 37. 解数独** | 二维回溯，每个空格尝试 1-9 |
| **LeetCode 79. 单词搜索** | 在二维网格中回溯，方向数组 |
| **LeetCode 22. 括号生成** | 用左右括号计数剪枝 |

它们共享同一个骨架：

```
function backtrack(路径, 选择列表) {
    if (满足条件) { 加入结果; return; }
    for (let 选择 of 选择列表) {
        剪枝判断;
        做选择;
        backtrack(...);
        撤销选择;
    }
}
```

---

## 总结

N 皇后之所以是回溯的经典例题，是因为它清晰地展现了回溯的三个核心要素：

1. **路径**：已放置皇后的棋盘状态
2. **选择列表**：当前行中可以放置皇后的列
3. **结束条件**：所有 N 行都放置了皇后

再抽象一步：**回溯 = DFS + 状态重置**。

面试中遇到"求所有方案""枚举所有可能""列出全部解"这类字眼时，90% 可以用回溯解决。把模板背熟、把状态怎么记录和撤销想清楚，剩下的就是剪枝优化了。

建议初学者在纸上画出 N=4 的递归树，用手动模拟一遍代码的执行流程——纸上跑完一遍，比看十遍代码都管用。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：后续会继续更新回溯系列的其他经典题目，关注不迷路。
