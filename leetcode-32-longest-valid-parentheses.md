---
title: LeetCode 32. 最长有效括号
tags: [栈, 动态规划, 双指针, 括号匹配]
difficulty: Hard
category: 动态规划
date: 2026-05-29
---

# LeetCode 32. 最长有效括号 —— 栈、DP、双指针三种解法全覆盖

## 前言

"最长有效括号"（Longest Valid Parentheses）是一道经典的字符串 + DP 综合题，也是面试中高频出现的中等偏难题。它看似简单——不就是找最长的合法括号子串吗？但真正写起来才发现，**暴力枚举所有子串再逐一判断合法性**的时间复杂度根本扛不住。

这道题的精妙之处在于：它可以用**三种完全不同的思路**来解——栈、动态规划、双指针，每种方法的空间复杂度逐步优化，是理解"同一问题多角度切入"的绝佳案例。

本文将覆盖三种解法：**栈** → **动态规划（DP）** → **双指针（空间优化）**，从 O(n) 空间到 O(1) 空间，一步步拆解这道题的每一个细节。

---

## 问题描述

### LeetCode 32. Longest Valid Parentheses（最长有效括号）

> 给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（格式正确且连续）括号子串的长度。

**示例：**

```
输入：s = "(()"
输出：2
解释：最长有效括号子串是 "()"
```

```
输入：s = ")()())"
输出：4
解释：最长有效括号子串是 "()()"
```

```
输入：s = ""
输出：0
```

---

## 核心思想

### 什么是"有效括号"？

有效括号字符串的定义：
1. 每个 `')'` 都有且仅有一个对应的 `'('` 与之匹配
2. 匹配顺序正确（先开后关）

```
有效:  "()",  "(())",  "()()",  "(()())"
无效:  ")(",  "())",  "(()",  ")()("
```

### 关键难点

难点不在于判断括号是否有效（那是 LeetCode 20），而在于**找到最长的连续有效子串**：

```
s = ")()())"

所有有效子串：
  [0]   = ")"      → 无效
  [1,2] = "()"     → 有效，长度 2
  [3,4] = "()"     → 有效，长度 2
  [1,4] = "()()"   → 有效，长度 4  ← 答案
  [2,3] = ")("     → 无效
```

直接枚举所有子串是 O(n²)，再判断每个子串是否有效是 O(n)，总复杂度 O(n³)——不可接受。

我们需要更聪明的方法。

---

## 思路分析

### 解法一：栈

栈的核心思想：**用栈存放下标，遇到匹配时计算长度**。

为什么存下标而不是字符？因为我们需要**计算长度**，而长度 = 当前下标 - 上一个未匹配位置的下标。

```
遍历字符串：
  遇到 '(' → 下标入栈
  遇到 ')' → 弹栈，计算长度

关键：栈底始终存放"上一个无法匹配的 ')' 的下标"，作为长度计算的起点。
```

**为什么栈底存的是"上一个无法匹配的位置"？**

```
s = ")()())"

初始：栈 = [-1]    ← 哨兵，表示"位置 -1 有一个虚拟的无法匹配点"

i=0: ')' → 弹出 -1，栈空了 → 把 0 入栈 → 栈 = [0]
     （0 是一个新的"无法匹配的 ')' 的位置"）

i=1: '(' → 入栈 → 栈 = [0, 1]

i=2: ')' → 弹出 1，栈 = [0] → 长度 = 2 - 0 = 2

i=3: '(' → 入栈 → 栈 = [0, 3]

i=4: ')' → 弹出 3，栈 = [0] → 长度 = 4 - 0 = 4

i=5: ')' → 弹出 0，栈空了 → 把 5 入栈 → 栈 = [5]

最终答案：4
```

栈底的哨兵值就像一个"围墙"，标记了有效子串的左边界。

### 解法二：动态规划（DP）

定义 `dp[i]` 为**以 `s[i]` 结尾的最长有效括号子串的长度**。

为什么定义是"以 i 结尾"而不是"以 i 开头"？——因为我们需要从左到右递推，每次只看当前位置能否和之前的子串拼接。

**状态转移：**

```
情况 1：s[i] = '('
  dp[i] = 0   （以 '(' 结尾的子串不可能是有效的）

情况 2：s[i] = ')' 且 s[i-1] = '('
  dp[i] = dp[i-2] + 2
  （"....()" 形式，前面的有效长度 + 2）

情况 3：s[i] = ')' 且 s[i-1] = ')'
  先看 s[i - dp[i-1] - 1] 是不是 '('
  如果是：dp[i] = dp[i-1] + 2 + dp[i - dp[i-1] - 2]
  （"....))" 形式，跳过内层有效子串，找外层的 '('）
```

**手动推演：** `s = ")()())"`

```
索引:   0  1  2  3  4  5
字符:   )  (  )  (  )  )
dp:     0  0  2  0  4  0

i=0: ')' → dp[0] = 0
i=1: '(' → dp[1] = 0
i=2: ')' → s[1]='(' → dp[2] = dp[0] + 2 = 2
i=3: '(' → dp[3] = 0
i=4: ')' → s[3]='(' → dp[4] = dp[2] + 2 = 4
i=5: ')' → s[4]=')' → dp[4]=4 → 看 s[5-4-1]=s[0]=')' → 不是 '(' → dp[5] = 0

答案：max(dp) = 4
```

### 解法三：双指针（空间优化）

双指针的核心思想：**从左到右扫一遍，从右到左再扫一遍**，用两个计数器 `left` 和 `right` 记录左右括号的数量。

为什么需要双向扫描？

```
s = "(()"

从左到右：
  i=0: left=1, right=0
  i=1: left=2, right=0
  i=2: left=2, right=1 → left===right → 长度=3? 不对！
  但实际上 "(()" 的最长有效子串是 "()"（长度 2）

问题：当 left > right 时，多出来的 '(' 没有被正确处理
```

双向扫描可以解决这个问题：
- 从左到右：当 `left === right` 时记录长度，当 `right > left` 时重置
- 从右到左：当 `left === right` 时记录长度，当 `left > right` 时重置

这样无论哪种方向的"多余"括号都能被正确处理。

---

## 代码实现

### JavaScript 版本

#### 方法一：栈

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var longestValidParentheses = function(s) {
    let maxLen = 0;
    const stack = [-1]; // 哨兵：上一个无法匹配的 ')' 的位置

    for (let i = 0; i < s.length; i++) {
        if (s[i] === '(') {
            stack.push(i); // '(' 的下标入栈
        } else {
            stack.pop(); // ')' 弹出栈顶匹配
            if (stack.length === 0) {
                stack.push(i); // 栈空了，当前 ')' 无法匹配，成为新的起点
            } else {
                maxLen = Math.max(maxLen, i - stack[stack.length - 1]);
            }
        }
    }

    return maxLen;
};
```

```python
def longest_valid_parentheses(s: str) -> int:
    """方法一：栈"""
    max_len = 0
    stack = [-1]  # 哨兵：上一个无法匹配的 ')' 的位置

    for i, ch in enumerate(s):
        if ch == '(':
            stack.append(i)  # '(' 的下标入栈
        else:
            stack.pop()  # ')' 弹出栈顶匹配
            if not stack:
                stack.append(i)  # 栈空了，成为新的起点
            else:
                max_len = max(max_len, i - stack[-1])

    return max_len
```

#### 方法二：动态规划（DP）

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var longestValidParentheses = function(s) {
    const n = s.length;
    const dp = new Array(n).fill(0);
    let maxLen = 0;

    for (let i = 1; i < n; i++) {
        if (s[i] === ')') {
            if (s[i - 1] === '(') {
                // 情况 2：....()
                dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
            } else {
                // 情况 3：....))
                // 跳过内层有效子串，找外层的 '('
                const matchIdx = i - dp[i - 1] - 1;
                if (matchIdx >= 0 && s[matchIdx] === '(') {
                    dp[i] = dp[i - 1] + 2 + (matchIdx >= 1 ? dp[matchIdx - 1] : 0);
                }
            }
            maxLen = Math.max(maxLen, dp[i]);
        }
    }

    return maxLen;
};
```

```python
def longest_valid_parentheses(s: str) -> int:
    """方法二：动态规划（DP）"""
    n = len(s)
    dp = [0] * n
    max_len = 0

    for i in range(1, n):
        if s[i] == ')':
            if s[i - 1] == '(':
                # 情况 2：....()
                dp[i] = (dp[i - 2] if i >= 2 else 0) + 2
            else:
                # 情况 3：....))
                # 跳过内层有效子串，找外层的 '('
                match_idx = i - dp[i - 1] - 1
                if match_idx >= 0 and s[match_idx] == '(':
                    dp[i] = dp[i - 1] + 2 + (dp[match_idx - 1] if match_idx >= 1 else 0)
            max_len = max(max_len, dp[i])

    return max_len
```

#### 方法三：双指针（空间优化，O(1) 空间）

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var longestValidParentheses = function(s) {
    let maxLen = 0;
    let left = 0, right = 0;

    // 从左到右扫描
    for (let i = 0; i < s.length; i++) {
        if (s[i] === '(') {
            left++;
        } else {
            right++;
        }
        if (left === right) {
            maxLen = Math.max(maxLen, 2 * right);
        } else if (right > left) {
            left = 0;
            right = 0;
        }
    }

    left = 0;
    right = 0;

    // 从右到左扫描
    for (let i = s.length - 1; i >= 0; i--) {
        if (s[i] === '(') {
            left++;
        } else {
            right++;
        }
        if (left === right) {
            maxLen = Math.max(maxLen, 2 * left);
        } else if (left > right) {
            left = 0;
            right = 0;
        }
    }

    return maxLen;
};
```

```python
def longest_valid_parentheses(s: str) -> int:
    """方法三：双指针（O(1) 空间）"""
    max_len = 0
    left = right = 0

    # 从左到右扫描
    for ch in s:
        if ch == '(':
            left += 1
        else:
            right += 1
        if left == right:
            max_len = max(max_len, 2 * right)
        elif right > left:
            left = right = 0

    left = right = 0

    # 从右到左扫描
    for ch in reversed(s):
        if ch == '(':
            left += 1
        else:
            right += 1
        if left == right:
            max_len = max(max_len, 2 * left)
        elif left > right:
            left = right = 0

    return max_len
```

### 逐步推演

以 `s = ")()())"` 为例，用三种方法分别推演：

**栈方法：**

```
初始化：栈 = [-1], maxLen = 0

i=0: s[0]=')' → 弹出 -1 → 栈空 → push 0 → 栈 = [0]
i=1: s[1]='(' → push 1 → 栈 = [0, 1]
i=2: s[2]=')' → 弹出 1 → 栈 = [0] → maxLen = 2 - 0 = 2
i=3: s[3]='(' → push 3 → 栈 = [0, 3]
i=4: s[4]=')' → 弹出 3 → 栈 = [0] → maxLen = max(2, 4 - 0) = 4
i=5: s[5]=')' → 弹出 0 → 栈空 → push 5 → 栈 = [5]

最终 maxLen = 4
```

**DP 方法：**

```
初始化：dp = [0,0,0,0,0,0], maxLen = 0

i=1: s[1]='(' → dp[1] = 0
i=2: s[2]=')', s[1]='(' → dp[2] = dp[0] + 2 = 2, maxLen = 2
i=3: s[3]='(' → dp[3] = 0
i=4: s[4]=')', s[3]='(' → dp[4] = dp[2] + 2 = 4, maxLen = 4
i=5: s[5]=')', s[4]=')' → matchIdx = 5 - 4 - 1 = 0, s[0]=')' → dp[5] = 0

最终 maxLen = 4
```

**双指针方法：**

```
从左到右：
i=0: ')' → right=1 > left=0 → 重置 left=0, right=0
i=1: '(' → left=1, right=0
i=2: ')' → left=1, right=1 → maxLen = max(0, 2) = 2
i=3: '(' → left=2, right=1
i=4: ')' → left=2, right=2 → maxLen = max(2, 4) = 4
i=5: ')' → left=2, right=3 > left → 重置

从右到左：
i=5: ')' → right=1, left=0
i=4: ')' → right=2, left=0
i=3: '(' → right=2, left=1
i=2: ')' → right=3, left=1 → right > left → 重置（不会超过已有的 4）

最终 maxLen = 4
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|-----------|-----------|---------|
| **栈** | O(n) | O(n) | 最直观，面试首选 |
| **动态规划** | O(n) | O(n) | 适合 DP 思维训练 |
| **双指针** | O(n) | O(1) | 空间敏感场景，进阶解法 |

---

## 优化方向

### 1. 三种方法的对比与选择

```
                    空间    直观性   适用面
栈（Stack）         O(n)    ★★★     通用，最容易理解
动态规划（DP）       O(n)    ★★☆     适合已掌握 DP 的同学
双指针（Two Ptr）   O(1)    ★☆☆     空间极致优化
```

**面试建议**：先写栈解法（最稳），再提 DP 解法（展示思维深度），最后提双指针（加分项）。

### 2. 栈方法的变体：记录匹配长度

栈方法还有一种变体——栈中存放**已匹配的有效长度**，而非下标：

```javascript
var longestValidParentheses = function(s) {
    let maxLen = 0;
    const stack = [0]; // 存放累计有效长度

    for (let i = 0; i < s.length; i++) {
        if (s[i] === '(') {
            stack.push(0); // 新的 '(' 开始，长度归零
        } else {
            if (stack.length <= 1) {
                stack[0] = 0; // 无法匹配，重置
            } else {
                const prev = stack.pop();
                stack[stack.length - 1] += prev + 2;
                maxLen = Math.max(maxLen, stack[stack.length - 1]);
            }
        }
    }

    return maxLen;
};
```

这种写法更紧凑，但不如下标栈法直观。

### 3. DP 方法的空间优化

DP 方法理论上也可以用滚动变量优化空间，但由于状态转移需要随机访问 `dp[i - dp[i-1] - 1]`（跳跃式访问），无法用简单的滚动数组实现，空间优化意义不大。如果需要 O(1) 空间，直接用双指针更好。

### 4. 处理嵌套与并列的区别

```
嵌套：  "(())"   → dp[3] = dp[1] + 2 = 4
并列：  "()()"   → dp[3] = dp[1] + 2 = 4
混合：  "(()())" → dp[5] = dp[3] + 2 + dp[0] = 6
```

DP 方法天然能处理这三种情况，因为状态转移方程已经覆盖了"跳过内层子串"的逻辑。

---

## 举一反三

最长有效括号的框架可以扩展到一系列括号匹配问题：

| 题目 | 不同点 | 关键思路 |
|------|--------|---------|
| **LeetCode 20. 有效的括号** | 判断整体是否有效 | 栈 + 匹配映射 |
| **LeetCode 32. 最长有效括号** | 求最长有效子串 | 栈/DP/双指针 |
| **LeetCode 22. 括号生成** | 生成所有有效组合 | 回溯 + 剪枝 |
| **LeetCode 921. 使括号有效的最少添加** | 求最少添加数 | 计数器 |
| **LeetCode 1541. 平衡括号串的最少插入** | 求最少插入数 | 计数器 + 分类讨论 |
| **LeetCode 678. 有效的括号字符串** | 通配符 `*` | 贪心 / DP |

它们的共同模式——**计数器框架**：

```javascript
// 括号问题通用框架：计数器法
function solve(s) {
    let left = 0, right = 0, result = 0;

    for (const ch of s) {
        if (ch === '(') {
            left++;
        } else {
            right++;
        }

        if (left === right) {
            // 当前子串有效，更新结果
            result = Math.max(result, 2 * right);
        } else if (right > left) {
            // 右括号多于左括号，重置
            left = 0;
            right = 0;
        }
    }

    return result;
}
```

---

## 总结

最长有效括号的精髓可以概括为三点：

1. **栈方法最实用**：用栈存下标，栈底放哨兵标记有效左边界，每次弹栈后用 `i - stack.top()` 算长度。思路清晰，代码简短，面试首选

2. **DP 方法最严谨**：`dp[i]` 表示以 `i` 结尾的最长有效长度，三种情况（`'('`、`"....()"`、`"....))"）的转移方程覆盖所有可能。适合系统学习 DP 的同学

3. **双指针方法最高效**：双向扫描 + 计数器，O(1) 空间。核心洞察是——单向扫描可能漏判，双向扫描互补覆盖

面试中遇到这道题，建议按以下步骤展示：

- 先跟面试官确认输入约束（只含 `'('` 和 `')'`？可能含其他字符？）
- 先说暴力解法（枚举子串 + 判断有效），指出 O(n³) 不可接受
- 写栈解法，画出栈的变化过程，解释哨兵的作用
- 主动提出 DP 解法，画出状态转移的三种情况
- 如果面试官要求空间优化，提出双指针方法

建议用 `")()())"` 和 `"(()"` 两个例子手动推演一遍栈方法——把栈的每一步变化画出来，哨兵的作用就一目了然了。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 1143. 最长公共子序列](./leetcode-1143-longest-common-subsequence.md) | [LeetCode 207. 课程表](./leetcode-207-course-schedule.md) | 后续继续更新动态规划系列，关注不迷路。
