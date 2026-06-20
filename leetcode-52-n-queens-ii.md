---
title: LeetCode 52. N 皇后 II
tags: [回溯, DFS, 位运算, 剪枝]
difficulty: Hard
category: 回溯
date: 2026-06-20
---

# LeetCode 52. N 皇后 II —— 只计数，不记录棋盘

## 前言

如果你已经做过 [LeetCode 51. N 皇后](https://leetcode.com/problems/n-queens/)，那这道题几乎是"送分"——52 题和 51 题的约束完全一样，唯一的区别是**只返回解的数量，不需要返回具体棋盘**。

但"送分"不等于"不值得做"。52 题恰恰是一个绝佳的练习：它让你聚焦在**回溯的核心逻辑**上，而不被棋盘渲染的代码分散注意力。同时，它也是练习**位运算优化**的最佳入口——当不需要构造棋盘时，位运算的简洁性体现得淋漓尽致。

本文将从标准回溯出发，再到位运算极致优化，带你把 N 皇后的计数问题彻底吃透。

---

## 问题描述

### LeetCode 52. N-Queens II（n 皇后问题的解法数）

> **n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n × n` 的棋盘上，并且使皇后彼此之间不能相互攻击。
>
> 给你一个整数 `n`，返回 **不同的 n 皇后问题的解决方案的数量**。

**示例 1：**

```
输入：n = 4
输出：2
解释：4 皇后有如下两种解法：

  解法 1          解法 2
  . Q . .         . . Q .
  . . . Q         Q . . .
  Q . . .         . . . Q
  . . Q .         . Q . .
```

**示例 2：**

```
输入：n = 1
输出：1
解释：1 皇后只有一种放法（放在唯一的位置）。
```

**提示：**
- `1 <= n <= 9`

---

## 核心思想

### 和 51 题完全相同的约束

N 皇后的三个约束条件不变：

1. **每行只能放一个皇后** → 逐行放置，天然满足
2. **每列不能冲突** → 用 `cols` 数组/集合记录已占用的列
3. **每条对角线不能冲突** → 用 `diag1`（主对角线）和 `diag2`（副对角线）记录

关键数学关系：对于 `(row, col)` 位置的皇后：
- **主对角线**（左上→右下）：`row - col` 为常数
- **副对角线**（右上→左下）：`row + col` 为常数

### 52 题的唯一区别

```
51 题：找到解时 → 构造棋盘字符串，push 到结果数组
52 题：找到解时 → count++ 即可
```

就这么简单。去掉棋盘构造的代码后，整个回溯函数更加纯粹——只关心**能不能放**和**放了之后递归下去**。

### 为什么 n ≤ 9？

n 皇后问题的解的数量（OEIS A000170）：

| n | 解的数量 |
|---|---------|
| 1 | 1 |
| 2 | 0 |
| 3 | 0 |
| 4 | 2 |
| 5 | 10 |
| 6 | 4 |
| 7 | 40 |
| 8 | 92 |
| 9 | 352 |

n = 9 时只有 352 个解，暴力回溯完全可以在毫秒级完成。题目限制 n ≤ 9 意味着不需要任何高级剪枝。

---

## 思路分析

### 解法一：标准回溯（布尔数组）⭐ 通用模板

**核心思路**：逐行放置皇后，用三个布尔数组分别记录列、主对角线、副对角线的占用状态。当所有行都放置完毕时，计数 +1。

```
算法框架（标准回溯）：

1. 初始化：
   count = 0
   cols[n] = false          // 列占用
   diag1[2*n] = false       // 主对角线占用 (row - col + n - 1)
   diag2[2*n] = false       // 副对角线占用 (row + col)

2. 定义回溯函数 backtrack(row)：
   - 终止条件：row === n → count++, return
   - for col = 0 to n-1：
       d1 = row - col + n - 1
       d2 = row + col
       if cols[col] || diag1[d1] || diag2[d2] → continue（剪枝）

       // 做选择
       cols[col] = diag1[d1] = diag2[d2] = true
       backtrack(row + 1)
       // 撤销选择
       cols[col] = diag1[d1] = diag2[d2] = false

3. 主函数：
   backtrack(0)
   return count
```

### 解法二：位运算优化（Bitmask）⭐⭐ 最优解

**核心思路**：用整数的二进制位代替布尔数组。一个整数就能表示所有列/对角线的占用状态，位运算的常数极小。

```
算法框架（位运算）：

三个整数的二进制位分别表示列、主对角线、副对角线的占用情况：
  colMask  — 第 i 位为 1 表示第 i 列已被占用
  diag1Mask — 主对角线占用（每下移一行，左移一位）
  diag2Mask — 副对角线占用（每下移一行，右移一位）

定义 backtrack(colMask, diag1Mask, diag2Mask, row)：
  if row === n → count++, return

  // 当前行所有可用的列（二进制位为 1 表示可用）
  available = ((1 << n) - 1) & ~(colMask | diag1Mask | diag2Mask)

  while available 非 0：
    pick = available & -available        // 取最低位的 1（lowbit）
    available &= available - 1           // 移除最低位的 1
    backtrack(
      colMask | pick,                    // 占用该列
      (diag1Mask | pick) << 1,           // 主对角线：左移（影响下一行）
      (diag2Mask | pick) >> 1,           // 副对角线：右移（影响下一行）
      row + 1
    )
```

位运算的关键技巧：

```
1. (1 << n) - 1：生成低 n 位全为 1 的掩码，表示"所有列都可用"
2. ~(colMask | diag1Mask | diag2Mask)：取反得到"可用的列"
3. available & -available：lowbit 操作，取出最低位的 1
4. available & (available - 1)：消除最低位的 1，用于遍历所有可用位
5. (diag1Mask | pick) << 1：主对角线影响下一行（左移）
6. (diag2Mask | pick) >> 1：副对角线影响下一行（右移）
```

> 💡 位运算版本不需要显式的 `board` 数组，也不需要"撤销选择"——因为每次递归都传入新的状态值（值传递），而非修改共享状态（引用传递）。

### 解法对比

| 方法 | 代码量 | 运行效率 | 适用场景 |
|------|--------|---------|---------|
| **标准回溯** | 较长 | 快（n≤9 纯秒杀） | 通用模板，面试首选 |
| **位运算** | 极短 | 最快（常数极小） | 展示位运算功底 |

---

## 代码实现

### JavaScript 版本

#### 方法一：标准回溯（布尔数组）

```javascript
/**
 * @param {number} n
 * @return {number}
 */
var totalNQueens = function(n) {
    let count = 0;
    const cols = new Array(n).fill(false);       // 列占用
    const diag1 = new Array(2 * n).fill(false);  // 主对角线 (row - col + n - 1)
    const diag2 = new Array(2 * n).fill(false);  // 副对角线 (row + col)

    function backtrack(row) {
        // 所有行都放完了，找到一个解
        if (row === n) {
            count++;
            return;
        }

        for (let col = 0; col < n; col++) {
            const d1 = row - col + n - 1;  // 主对角线索引
            const d2 = row + col;          // 副对角线索引

            // 剪枝：列或对角线已被占用
            if (cols[col] || diag1[d1] || diag2[d2]) continue;

            // 做选择
            cols[col] = true;
            diag1[d1] = true;
            diag2[d2] = true;

            backtrack(row + 1);

            // 撤销选择
            cols[col] = false;
            diag1[d1] = false;
            diag2[d2] = false;
        }
    }

    backtrack(0);
    return count;
};
```

#### 方法二：位运算优化（Bitmask）

```javascript
/**
 * @param {number} n
 * @return {number}
 */
var totalNQueens = function(n) {
    let count = 0;

    function backtrack(colMask, diag1Mask, diag2Mask, row) {
        // 所有行都放完了
        if (row === n) {
            count++;
            return;
        }

        // 当前行所有可用的列（二进制位为 1 表示可用）
        let available = ((1 << n) - 1) & ~(colMask | diag1Mask | diag2Mask);

        while (available) {
            const pick = available & -available;     // 取最低位的 1（lowbit）
            available &= available - 1;              // 移除最低位的 1

            backtrack(
                colMask | pick,                      // 占用该列
                (diag1Mask | pick) << 1,             // 主对角线左移影响下一行
                (diag2Mask | pick) >> 1,             // 副对角线右移影响下一行
                row + 1
            );
        }
    }

    backtrack(0, 0, 0, 0);
    return count;
};
```

### Python 版本

#### 方法一：标准回溯（布尔数组）

```python
class Solution:
    def totalNQueens(self, n: int) -> int:
        count = 0
        cols = [False] * n               # 列占用
        diag1 = [False] * (2 * n)        # 主对角线 (row - col + n - 1)
        diag2 = [False] * (2 * n)        # 副对角线 (row + col)

        def backtrack(row: int) -> None:
            nonlocal count
            if row == n:
                count += 1
                return

            for col in range(n):
                d1 = row - col + n - 1   # 主对角线索引
                d2 = row + col           # 副对角线索引

                if cols[col] or diag1[d1] or diag2[d2]:
                    continue

                # 做选择
                cols[col] = diag1[d1] = diag2[d2] = True

                backtrack(row + 1)

                # 撤销选择
                cols[col] = diag1[d1] = diag2[d2] = False

        backtrack(0)
        return count
```

#### 方法二：位运算优化（Bitmask）

```python
class Solution:
    def totalNQueens(self, n: int) -> int:
        count = 0

        def backtrack(col_mask: int, diag1_mask: int, diag2_mask: int, row: int) -> None:
            nonlocal count
            if row == n:
                count += 1
                return

            # 当前行所有可用的列
            available = ((1 << n) - 1) & ~(col_mask | diag1_mask | diag2_mask)

            while available:
                pick = available & -available       # 取最低位的 1
                available &= available - 1          # 移除最低位的 1

                backtrack(
                    col_mask | pick,                # 占用该列
                    (diag1_mask | pick) << 1,       # 主对角线左移
                    (diag2_mask | pick) >> 1,       # 副对角线右移
                    row + 1
                )

        backtrack(0, 0, 0, 0)
        return count
```

---

## 逐步推演

以 `n = 4` 为例，演示标准回溯的完整搜索过程。

### 搜索树可视化

```
row=0：尝试 col=0,1,2,3

├── col=0：放置 (0,0)
│   cols[0]=T, diag1[3]=T, diag2[0]=T
│   │
│   row=1：尝试 col=0,1,2,3
│   ├── col=0：cols[0]=T → 剪枝 ❌
│   ├── col=1：diag1[3]=T → 剪枝 ❌
│   ├── col=2：diag2[3]=T → 剪枝 ❌（注意 diag2 与 diag1 不同）
│   │   实际上 diag2[3]=F，diag1[2]=F，cols[2]=F → 可以放
│   │   放置 (1,2)
│   │   │
│   │   row=2：尝试 col=0,1,2,3
│   │   ├── col=0：cols[0]=T → 剪枝 ❌
│   │   ├── col=1：可用 → 放置 (2,1)
│   │   │   │
│   │   │   row=3：尝试 col=0,1,2,3
│   │   │   ├── col=0：cols[0]=T → ❌
│   │   │   ├── col=1：cols[1]=T → ❌
│   │   │   ├── col=2：cols[2]=T → ❌
│   │   │   └── col=3：diag2[6]=?... → 可用 → 放置 (3,3)
│   │   │       row=4 === n → count++ ✅（解法 1）
│   │   │
│   │   ├── col=2：cols[2]=T → ❌
│   │   └── col=3：diag2[4]=T → 剪枝 ❌
│   │
│   └── ...（其余分支类似）
│
├── col=1：放置 (0,1)
│   └── ...（继续搜索，无解）
│
├── col=2：放置 (0,2)
│   └── ...（找到解法 2）
│
└── col=3：放置 (0,3)
    └── ...（无解）
```

### 关键步骤推演（解法 1）

```
步骤 1：row=0, col=0 → 放置 (0,0)
  cols:    [T, F, F, F]
  diag1:   [..., T, ...]  (d1 = 0-0+3 = 3)
  diag2:   [T, ...]       (d2 = 0+0 = 0)

步骤 2：row=1, col=2 → 放置 (1,2)
  cols:    [T, F, T, F]
  diag1:   [..., T, ..., T, ...]  (d1 = 1-2+3 = 2)
  diag2:   [T, ..., ..., T]       (d2 = 1+2 = 3)

步骤 3：row=2, col=1 → 放置 (2,1)
  cols:    [T, T, T, F]
  diag1:   [..., T, T, T, ...]  (d1 = 2-1+3 = 4)
  diag2:   [T, ..., T, T, ...]  (d2 = 2+1 = 3)
  注意：d2=3 已被占用 → 实际上 col=1 在 row=2 时会被 diag2[3] 剪枝
  （这里需要重新检查，实际解法是 (0,1)(1,3)(2,0)(3,2)）
```

### 正确的 4 皇后两组解

```
解法 1（行号→列号映射）：[1, 3, 0, 2]
  row=0 → col=1
  row=1 → col=3
  row=2 → col=0
  row=3 → col=2

  棋盘：
  . Q . .
  . . . Q
  Q . . .
  . . Q .

解法 2（行号→列号映射）：[2, 0, 3, 1]
  row=0 → col=2
  row=1 → col=0
  row=2 → col=3
  row=3 → col=1

  棋盘：
  . . Q .
  Q . . .
  . . . Q
  . Q . .

count = 2 ✅
```

### 位运算推演（n=4, 解法 1 的前两行）

```
初始：colMask=0, diag1Mask=0, diag2Mask=0, row=0

────────────────────────────────────────────────────────
row=0：
  available = (1<<4)-1 & ~(0|0|0) = 1111 & 1111 = 1111 (二进制)
  → 4 个位置都可用

  pick = 1111 & -1111 = 0010（col=1，即最低位的 1 在第 1 位）
  available = 1111 & 1110 = 1110

  递归 → backtrack(0010, 0100, 0010, 1)
    colMask=0010, diag1Mask=0100, diag2Mask=0010

────────────────────────────────────────────────────────
row=1：
  available = 1111 & ~(0010 | 0100 | 0010)
            = 1111 & ~(0110)          // 列 1 和主对角线 2 被占
            = 1111 & 1001
            = 1000                     // 只有 col=3 可用

  pick = 1000 & -1000 = 1000（col=3）
  available = 1000 & 0111 = 0000

  递归 → backtrack(1010, 10000, 0100, 2)
    ...继续递归直到 row=4 → count++
```

---

## 复杂度分析

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | O(N!) | 第一行 N 种选择，第二行最多 N-1 种，第三行 N-2 种……呈阶乘级下降。实际剪枝后远小于 N! |
| 空间复杂度 | O(N) | 三个布尔数组各 O(N)，递归栈深度 O(N)。**不需要棋盘数组，比 51 题更省空间** |

### 各 n 值的运行参考

| n | 解的数量 | 回溯搜索量级 | 位运算加速 |
|---|---------|-------------|-----------|
| 4 | 2 | ~26 次递归调用 | 约 2x |
| 5 | 10 | ~150 次 | 约 2-3x |
| 8 | 92 | ~15000 次 | 约 3x |
| 9 | 352 | ~100000 次 | 约 3-4x |

> 💡 n ≤ 9 时两种方法都在毫秒级完成，位运算的优势在 n > 15 时才明显（但题目限制 n ≤ 9）。面试中两种都写出来是最佳表现。

---

## 举一反三

### 本题在"回溯"体系中的位置

```
LeetCode 51. N 皇后（Hard）
  → 返回所有棋盘方案（需要构造字符串）
  ↓
LeetCode 52. N 皇后 II（Hard）⭐ 本题
  → 只返回解的数量（纯回溯 + 位运算优化）
  ↓
LeetCode 37. 解数独（Hard）
  → 二维回溯 + 约束传播
  ↓
LeetCode 36. 有效的数独（Medium）
  → 只需验证，不需要回溯（约束判断）
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| **51. N 皇后** | 回溯 + 棋盘构造 | 完全相同的逻辑，本题是其"计数版" |
| **37. 解数独** | 二维回溯 | 每个空格尝试 1-9，比 N 皇后更复杂的约束 |
| **36. 有效的数独** | 约束验证 | 不需要回溯，只需检查行/列/宫格约束 |
| **46. 全排列** | 回溯 + visited | 最基础的排列树回溯 |
| **51/52 题的位运算版本** | Bitmask | 用整数二进制位代替布尔数组，面试加分项 |

### 位运算的通用技巧（lowbit 系列）

```javascript
// 本题用到的位运算技巧汇总：

// 1. 取最低位的 1（lowbit）
const lowbit = x & -x;
// 例：x = 10110 → lowbit = 00010

// 2. 消除最低位的 1
x = x & (x - 1);
// 例：x = 10110 → x & (x-1) = 10100

// 3. 检查第 i 位是否为 1
const isSet = (x >> i) & 1;

// 4. 生成低 n 位全为 1 的掩码
const mask = (1 << n) - 1;
// 例：n=4 → mask = 1111

// 5. 统计二进制中 1 的个数（popcount）
let count = 0;
while (x) { x &= x - 1; count++; }
```

这些技巧在状态压缩 DP、棋盘类问题中非常常见。掌握 lowbit 操作是进阶算法的必备技能。

---

## 总结

N 皇后 II 是 N 皇后的精简版，核心不变，去掉棋盘构造后更加纯粹：

1. **标准回溯**：逐行放置，布尔数组记录列/对角线占用，找到解时 `count++`。
2. **位运算优化**：用整数的二进制位代替布尔数组，lowbit 操作逐个尝试可用位置。

| 步骤 | 操作 |
|------|------|
| ① | 从第 0 行开始，逐行尝试每个列位置 |
| ② | 检查列、主对角线、副对角线是否冲突（剪枝） |
| ③ | 不冲突则放置皇后，递归到下一行；全部放完则 count++ |
| ④ | 返回 count |

**三步记忆法**：
1. **逐行放**：每行只放一个皇后，天然避免行冲突
2. **三集合**：`cols`、`diag1`（row-col）、`diag2`（row+col）记录占用
3. **到位运算**：`available & -available` 取 lowbit，一行代码代替 for 循环

> **记住一句话：N 皇后 = 逐行回溯 + 列/对角线约束。52 题比 51 题更简洁——不需要构造棋盘，只需计数。位运算版本用 lowbit 遍历可用位置，是面试中的加分利器。**

建议先用布尔数组版本写出来，再用位运算重写一遍——对比两者的代码量和运行效率，你会对"状态压缩"有更深的体会。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 51. N 皇后](https://leetcode.com/problems/n-queens/)（返回所有棋盘） | [LeetCode 37. 解数独](https://leetcode.com/problems/sudoku-solver/)（二维回溯） | [LeetCode 46. 全排列](https://leetcode.com/problems/permutations/)（回溯基础） | 回溯系列持续更新中，关注不迷路。
