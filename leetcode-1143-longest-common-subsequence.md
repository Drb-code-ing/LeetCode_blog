---
title: LeetCode 1143. 最长公共子序列
tags: [动态规划, 二维DP, 序列匹配, 回溯]
difficulty: Medium
category: 动态规划
date: 2026-05-28
---

# LeetCode 1143. 最长公共子序列 —— 从 DP 表格到序列重建，两种方法彻底掌握 LCS

## 前言

"最长公共子序列"（Longest Common Subsequence，简称 LCS）是动态规划领域的经典问题，也是面试中考察二维 DP 理解深度的标杆题。

很多人在学 DP 时都遇到过这个问题：**"为什么两个字符串的 LCS 能用二维数组求？"** 答案在于——子序列问题天然具有"选或不选"的决策结构，而两个序列的匹配正好可以映射到二维表格的坐标轴上。

本文将覆盖两种方法：**动态规划（DP）** → **序列重建（栈回溯）**，从求长度到还原具体序列，一步步拆解 LCS 的每一个细节。

---

## 问题描述

### LeetCode 1143. Longest Common Subsequence（最长公共子序列）

> 给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长**公共子序列**的长度。如果不存在公共子序列，返回 `0`。
>
> 一个字符串的**子序列**是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是。

**示例：**

```
输入：text1 = "abcde", text2 = "ace"
输出：3
解释：最长公共子序列是 "ace"，它的长度为 3。
```

```
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc"，它的长度为 3。
```

```
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0。
```

---

## 核心思想

### 什么是子序列？

子序列不要求连续，只要求**相对顺序不变**。这是子序列和子串（substring）最本质的区别：

```
字符串: "abcde"

子串（连续）:  "abc", "bcd", "cde", "ab", ...
子序列（可不连续）: "ace", "ade", "abe", "bde", ...
```

正因为不要求连续，我们在匹配两个字符串时，可以跳过任意多个字符来寻找匹配——这给了动态规划用武之地。

### 子序列问题的特征

子序列问题通常有两种状态选择：**选或不选**。

对于 LCS，考虑两个字符串的最后一个字符：

```
text1[0..i] 和 text2[0..j] 的 LCS
```

- 如果 `text1[i] === text2[j]`，这个字符一定在 LCS 中，问题规模缩小为 `text1[0..i-1]` 和 `text2[0..j-1]`
- 如果 `text1[i] !== text2[j]`，则要么丢弃 `text1[i]`，要么丢弃 `text2[j]`，取两种情况的最大值

这就是 LCS 的核心递推关系。

---

## 思路分析

### 解法一：动态规划（DP）

定义 `dp[i][j]` 为 `text1[0..i-1]` 和 `text2[0..j-1]` 的最长公共子序列的长度。

为什么用 `i-1` 和 `j-1` 而不是 `i` 和 `j`？——为了方便处理空串的情况，`dp[0][j]` 和 `dp[i][0]` 表示其中一个串为空，LCS 长度为 0。

**状态转移方程：**

```
dp[i][j] =
    dp[i-1][j-1] + 1                     如果 text1[i-1] === text2[j-1]
    max(dp[i-1][j], dp[i][j-1])          如果 text1[i-1] !== text2[j-1]
```

**直观理解：**

```
       text2[j-1]
         ↓
text1[i-1] →  如果相等 → 对角线 + 1
              如果不等 → 取左边和上边的最大值
```

**手动推演：** `text1 = "abcde"`, `text2 = "ace"`

```
    ""  a  c  e
""   0  0  0  0
a    0  1  1  1
b    0  1  1  1
c    0  1  2  2
d    0  1  2  2
e    0  1  2  3  ← 答案是 3
```

推演过程：

```
i=1 (text1='a'), j=1 (text2='a'): 'a' === 'a' → dp[1][1] = dp[0][0] + 1 = 1
i=1 (text1='a'), j=2 (text2='c'): 'a' !== 'c' → dp[1][2] = max(dp[0][2]=0, dp[1][1]=1) = 1
i=1 (text1='a'), j=3 (text2='e'): 'a' !== 'e' → dp[1][3] = max(dp[0][3]=0, dp[1][2]=1) = 1

i=2 (text1='b'), j=1 (text2='a'): 'b' !== 'a' → dp[2][1] = max(dp[1][1]=1, dp[2][0]=0) = 1
i=2 (text1='b'), j=2 (text2='c'): 'b' !== 'c' → dp[2][2] = max(dp[1][2]=1, dp[2][1]=1) = 1
i=2 (text1='b'), j=3 (text2='e'): 'b' !== 'e' → dp[2][3] = max(dp[1][3]=1, dp[2][2]=1) = 1

i=3 (text1='c'), j=1 (text2='a'): 'c' !== 'a' → dp[3][1] = max(dp[2][1]=1, dp[3][0]=0) = 1
i=3 (text1='c'), j=2 (text2='c'): 'c' === 'c' → dp[3][2] = dp[2][1] + 1 = 2
i=3 (text1='c'), j=3 (text2='e'): 'c' !== 'e' → dp[3][3] = max(dp[2][3]=1, dp[3][2]=2) = 2

... 最终 dp[5][3] = 3
```

### 解法二：序列重建（栈回溯法）

DP 表格只能告诉我们 LCS 的**长度**，不能告诉我们 LCS 具体是**什么**。

要还原出具体的 LCS 序列，我们需要在 DP 表填完后，从 `dp[m][n]` 开始**反向回溯**。

回溯规则：

```
从 dp[m][n] 开始：
  如果 text1[i-1] === text2[j-1]：
      这个字符在 LCS 中 → 记录它
      i--, j--          → 沿对角线移动
  否则：
      看 dp[i-1][j] 和 dp[i][j-1] 哪个大
      往大的方向移动

回溯结束后，记录的字符是逆序的（从后往前找到的），
所以需要用栈（后进先出）来反转顺序，得到正确的 LCS 序列。
```

**可视化回溯过程：** `text1 = "abcde"`, `text2 = "ace"`

```
DP 表：
    ""  a  c  e
""   0  0  0  0
a    0  1  1  1
b    0  1  1  1
c    0  1  2  2
d    0  1  2  2
e    0  1  2  3

回溯路径（从 dp[5][3] 到 dp[0][0]）：

dp[5][3]='e'、text2[2]='e' → 相等！入栈 'e' → (4, 2)
dp[4][2]='d'、text2[1]='c' → 不等 → dp[3][2]=2 >= dp[4][1]=1 → 向上 → (3, 2)
dp[3][2]='c'、text2[1]='c' → 相等！入栈 'c' → (2, 1)
dp[2][1]='b'、text2[0]='a' → 不等 → dp[1][1]=1 >= dp[2][0]=0 → 向上 → (1, 1)
dp[1][1]='a'、text2[0]='a' → 相等！入栈 'a' → (0, 0)

出栈 → "a" → "c" → "e" → 得到 "ace" ✅
```

注意：**回溯路径不唯一**。当 `dp[i-1][j] === dp[i][j-1]` 时，向左或向上都可能得到不同的 LCS（但长度相同）。例如上面 `text1 = "abcde"`, `text2 = "ace"` 的 LCS 只有 `"ace"` 一种，但如果多个字符有相同优先级，不同走法会得到不同的有效 LCS。

---

## 代码实现

### JavaScript 版本

#### 方法一：动态规划（DP）

```javascript
/**
 * @param {string} text1
 * @param {string} text2
 * @return {number}
 */
var longestCommonSubsequence = function(text1, text2) {
    const m = text1.length;
    const n = text2.length;

    // dp[i][j] 表示 text1[0..i-1] 和 text2[0..j-1] 的 LCS 长度
    const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));

    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (text1[i - 1] === text2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }

    return dp[m][n];
};
```

```python
def longest_common_subsequence(text1: str, text2: str) -> int:
    """方法一：动态规划（DP）"""
    m, n = len(text1), len(text2)
    # dp[i][j] 表示 text1[0..i-1] 和 text2[0..j-1] 的 LCS 长度
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    return dp[m][n]
```

#### 方法二：DP + 栈回溯（还原 LCS 序列）

```javascript
/**
 * 求 LCS 长度 + 还原具体序列
 * @param {string} text1
 * @param {string} text2
 * @return {{ length: number, sequence: string }}
 */
function longestCommonSubsequenceWithSequence(text1, text2) {
    const m = text1.length;
    const n = text2.length;

    // 1. 标准 DP 填表
    const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));

    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (text1[i - 1] === text2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }

    // 2. 栈回溯：从 dp[m][n] 反向追踪
    const stack = [];
    let i = m, j = n;

    while (i > 0 && j > 0) {
        if (text1[i - 1] === text2[j - 1]) {
            // 匹配到了！入栈，沿对角线移动
            stack.push(text1[i - 1]);
            i--;
            j--;
        } else if (dp[i - 1][j] >= dp[i][j - 1]) {
            // 向上移动（优先选择值更大的方向）
            i--;
        } else {
            // 向左移动
            j--;
        }
    }

    // 3. 栈中元素出栈即为正确的顺序
    const sequence = stack.reverse().join('');

    return {
        length: dp[m][n],
        sequence: sequence
    };
}
```

```python
def longest_common_subsequence_with_sequence(text1: str, text2: str) -> dict:
    """方法二：DP + 栈回溯（还原 LCS 序列）"""
    m, n = len(text1), len(text2)

    # 1. 标准 DP 填表
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    # 2. 栈回溯：从 dp[m][n] 反向追踪
    stack = []
    i, j = m, n

    while i > 0 and j > 0:
        if text1[i - 1] == text2[j - 1]:
            # 匹配到了！入栈，沿对角线移动
            stack.append(text1[i - 1])
            i -= 1
            j -= 1
        elif dp[i - 1][j] >= dp[i][j - 1]:
            # 向上移动（优先选择值更大的方向）
            i -= 1
        else:
            # 向左移动
            j -= 1

    # 3. 栈中元素出栈即为正确的顺序
    sequence = ''.join(reversed(stack))

    return {"length": dp[m][n], "sequence": sequence}
```

#### 方法三：空间优化版（滚动数组）

由于 `dp[i][j]` 只依赖 `dp[i-1][j-1]`（左上）、`dp[i-1][j]`（上）、`dp[i][j-1]`（左），我们可以用两行（甚至一行加一个变量）来优化空间。

```javascript
/**
 * 空间优化版：O(n) 空间
 * @param {string} text1
 * @param {string} text2
 * @return {number}
 */
var longestCommonSubsequence = function(text1, text2) {
    const m = text1.length;
    const n = text2.length;

    // 只保留两行：prev 是上一行，curr 是当前行
    let prev = new Array(n + 1).fill(0);
    let curr = new Array(n + 1).fill(0);

    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (text1[i - 1] === text2[j - 1]) {
                curr[j] = prev[j - 1] + 1;
            } else {
                curr[j] = Math.max(prev[j], curr[j - 1]);
            }
        }
        // 交换两行
        [prev, curr] = [curr, prev];
    }

    return prev[n];
};
```

```python
def longest_common_subsequence(text1: str, text2: str) -> int:
    """方法三：空间优化版（滚动数组，O(n) 空间）"""
    m, n = len(text1), len(text2)

    # 只保留两行：prev 是上一行，curr 是当前行
    prev = [0] * (n + 1)
    curr = [0] * (n + 1)

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                curr[j] = prev[j - 1] + 1
            else:
                curr[j] = max(prev[j], curr[j - 1])
        # 交换两行
        prev, curr = curr, prev

    return prev[n]
```

> **注意**：空间优化版只适用于求 LCS 长度，无法用于回溯还原序列（因为只保留了两行的 DP 值，回溯需要完整表格）。

### 逐步推演

以 `text1 = "abcde"`, `text2 = "ace"` 为例，用 DP + 栈回溯法完整推演：

```
初始化 dp 表（6×4）：
    ""  a  c  e
""   0  0  0  0
a    0  ?  ?  ?
b    0  ?  ?  ?
c    0  ?  ?  ?
d    0  ?  ?  ?
e    0  ?  ?  ?

填充 dp 表：

i=1 (a):
  j=1 (a): a===a → dp[1][1]=dp[0][0]+1=1
  j=2 (c): a!==c → dp[1][2]=max(dp[0][2]=0, dp[1][1]=1)=1
  j=3 (e): a!==e → dp[1][3]=max(dp[0][3]=0, dp[1][2]=1)=1

i=2 (b):
  j=1 (a): b!==a → dp[2][1]=max(dp[1][1]=1, dp[2][0]=0)=1
  j=2 (c): b!==c → dp[2][2]=max(dp[1][2]=1, dp[2][1]=1)=1
  j=3 (e): b!==e → dp[2][3]=max(dp[1][3]=1, dp[2][2]=1)=1

i=3 (c):
  j=1 (a): c!==a → dp[3][1]=max(dp[2][1]=1, dp[3][0]=0)=1
  j=2 (c): c===c → dp[3][2]=dp[2][1]+1=2
  j=3 (e): c!==e → dp[3][3]=max(dp[2][3]=1, dp[3][2]=2)=2

i=4 (d):
  j=1 (a): d!==a → dp[4][1]=max(dp[3][1]=1, dp[4][0]=0)=1
  j=2 (c): d!==c → dp[4][2]=max(dp[3][2]=2, dp[4][1]=1)=2
  j=3 (e): d!==e → dp[4][3]=max(dp[3][3]=2, dp[4][2]=2)=2

i=5 (e):
  j=1 (a): e!==a → dp[5][1]=max(dp[4][1]=1, dp[5][0]=0)=1
  j=2 (c): e!==c → dp[5][2]=max(dp[4][2]=2, dp[5][1]=1)=2
  j=3 (e): e===e → dp[5][3]=dp[4][2]+1=3

最终 dp 表：
    ""  a  c  e
""   0  0  0  0
a    0  1  1  1
b    0  1  1  1
c    0  1  2  2
d    0  1  2  2
e    0  1  2  3    ← 结果是 3

栈回溯过程：
从 (5,3) 开始：
  text1[4]='e', text2[2]='e' → 匹配 → 入栈 'e', i=4, j=2
  text1[3]='d', text2[1]='c' → 不匹配 → dp[3][2]=2 >= dp[4][1]=1 → i=3, j=2
  text1[2]='c', text2[1]='c' → 匹配 → 入栈 'c', i=2, j=1
  text1[1]='b', text2[0]='a' → 不匹配 → dp[1][1]=1 >= dp[2][0]=0 → i=1, j=1
  text1[0]='a', text2[0]='a' → 匹配 → 入栈 'a', i=0, j=0

栈 (从底到顶): ['e', 'c', 'a']
出栈反转 → "ace" ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|-----------|-----------|---------|
| **标准 DP** | O(m × n) | O(m × n) | 只求长度，便于理解 |
| **DP + 栈回溯** | O(m × n) | O(m × n + L) | 需要还原具体 LCS 序列 |
| **滚动数组优化** | O(m × n) | O(n) | 只求长度，空间敏感场景 |

其中 `m = text1.length`, `n = text2.length`, `L = LCS.length`

---

## 优化方向

### 1. 空间优化：从完整表格到滚动数组

标准 DP 用 O(m×n) 的空间存储整个表格，但状态转移只依赖三格：`dp[i-1][j-1]`、`dp[i-1][j]`、`dp[i][j-1]`。

```
依赖关系：

  (i-1,j-1)  ←  (i-1,j)
       ↓            ↓
  (i, j-1)  ←  (i, j)      ← 当前位置
```

这意味着我们只需要**上一行**和**当前行**两行数据就够了。

但滚动数组的代价是**无法回溯还原序列**——回溯需要完整的 DP 表来确定每一步的走向。

### 2. 栈回溯的多种路径

当 `dp[i-1][j] === dp[i][j-1]` 时，向上或向左走会得到不同的 LCS（但长度相同）：

```
text1 = "abcdef", text2 = "acf"

DP 表回溯时，'a' 和 'c' 匹配后，向上/向左的选择会影响后续匹配的字符。
不同走法可能得到 "acf" 或 "abf" 等不同序列，但长度相同。
```

如果需要输出**所有** LCS 序列，需要用 DFS 搜索所有可能的回溯路径。

### 3. 如果两个字符串长度差异很大

当 `m << n` 或 `n << m` 时，可以用较短的字符串作为列来优化空间：

```javascript
// 确保 text1 是较短的字符串
if (text1.length > text2.length) {
    [text1, text2] = [text2, text1];
}
// 此时 dp 列数为较短字符串的长度 → 空间 O(min(m, n))
```

这只是微优化，但在面试中提出来会是加分项。

### 4. LCS 与编辑距离的关系

LCS 和编辑距离（LeetCode 72）共享同一个 DP 框架：

```
编辑距离：dp[i][j] = min(dp[i-1][j] + 1, dp[i][j-1] + 1, dp[i-1][j-1] + cost)
LCS：     dp[i][j] = max(dp[i-1][j],     dp[i][j-1],     dp[i-1][j-1] + match)
```

两者都是二维序列匹配的经典问题，区别在于 LCS 求的是**最长公共部分**，而编辑距离求的是**最小转换代价**。

---

## 举一反三

LCS 的 DP 框架可以扩展到一系列子序列匹配问题：

| 题目 | 不同点 | 关键思路 |
|------|--------|---------|
| **LeetCode 1143. 最长公共子序列** | 标准 LCS | 二维 DP 基础模板 |
| **LeetCode 583. 两个字符串的删除操作** | 求删除次数 | `m + n - 2 × LCS` |
| **LeetCode 712. 两个字符串的最小 ASCII 删除和** | 删除有代价 | 类似 LCS，但存最小代价而非长度 |
| **LeetCode 1035. 不相交的线** | 连接数字 | 本质就是 LCS（数组版本的 LCS） |
| **LeetCode 516. 最长回文子序列** | 单个字符串 | LCS(s, reverse(s)) |
| **LeetCode 72. 编辑距离** | 增删改操作 | LCS 的扩展，三维状态 |

它们的共同模式：

```javascript
// 二维字符串匹配的通用框架
function solve(s1, s2) {
    const m = s1.length, n = s2.length;
    const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(base));

    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (s1[i - 1] === s2[j - 1]) {
                dp[i][j] = f(dp[i - 1][j - 1]); // 匹配
            } else {
                dp[i][j] = g(dp[i - 1][j], dp[i][j - 1]); // 不匹配
            }
        }
    }

    return dp[m][n];
}
```

### 进阶：最短公共超序列（Shortest Common Supersequence, LeetCode 1092）

给定两个字符串 `str1` 和 `str2`，返回同时以 `str1` 和 `str2` 作为子序列的最短字符串。

解法：**先求 LCS，然后合并两个字符串**——LCS 中的字符只出现一次，非 LCS 的字符按顺序填入。

```
str1 = "abac", str2 = "cab"

LCS = "ab" 或 "ac"

合并策略：
  "c" + "a" + "b" + "a" + "c" = "cabac"
   ┗ LCS "ab" 只出现一次
```

这也说明了 LCS 是构建最短公共超序列的基础。

---

## 总结

最长公共子序列的精髓可以概括为三点：

1. **DP 状态定义**：`dp[i][j]` 表示两个前缀子串的 LCS 长度，`[i-1][j-1]` 这个偏移设计是为了优雅处理空串边界

2. **两种方法**：
   - **DP 求长度**：二维表填完，右下角即为答案。空间可优化到 O(n) 滚动数组
   - **栈回溯还原序列**：从右下角反向追踪，用栈来反转顺序得到正确的 LCS

3. **思维框架**：二维序列匹配问题几乎都可以套用这个 DP 模板——匹配时取对角线，不匹配时取邻格最优值

面试中遇到 LCS 及其变体，建议按以下步骤展示：

- 先跟面试官确认是**子序列（不连续）**还是**子串（连续）**——这直接决定了算法
- 在白板上画出 DP 表格，手动推演一行一列，展示你对转移方程的理解
- 写出标准 DP 代码后，主动提出空间优化（滚动数组）和序列重建（栈回溯）
- 如果能说出 LCS 和编辑距离的关系、以及 LCS 可以用来求最长回文子序列，就是完美的面试表现

建议拿起笔在纸上手动填一遍 `"abcde"` 和 `"ace"` 的 DP 表，再走一遍回溯路径——把每一个箭头画出来。纸上跑完一遍，比看十遍代码更管用。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 207. 课程表](./leetcode-207-course-schedule.md) | [LeetCode 152. 乘积最大子数组](./leetcode-152-maximum-product-subarray.md) | 后续继续更新动态规划系列，关注不迷路。
