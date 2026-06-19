---
title: LeetCode 17. 电话号码的字母组合
tags: [回溯, DFS, 决策树, 迭代, 笛卡尔积]
difficulty: Medium
category: 回溯
date: 2026-06-19
---

# LeetCode 17. 电话号码的字母组合 —— 决策树上的回溯搜索

## 前言

看到这道题，很多人的第一反应是："不就是个多层 for 循环吗？" —— 如果 digits 长度固定（比如 3），确实可以写 3 层 for 循环。但 digits 的长度是**可变的**（0 到 4），这就意味着你需要一种**能应对未知层数的循环机制**。

而**回溯算法（Backtracking）**正是解决这类"在多个集合中各选一个元素，列出所有组合"问题的标准解法。

回溯的本质是在**决策树上做 DFS**：每一位数字是一个决策层，每一层有 3~4 个字母可选。穷尽所有选择路径 = 所有可能的字母组合。

除了回溯，本文还会介绍一种**更直观的迭代法**（笛卡尔积思想），让你彻底搞明白这道经典的回溯入门题。

---

## 问题描述

### LeetCode 17. Letter Combinations of a Phone Number（电话号码的字母组合）

> 给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按**任意顺序**返回。
>
> 数字到字母的映射与电话按键相同。注意 `1` 不对应任何字母。

**数字-字母映射表：**

| 数字 | 字母 |
|------|------|
| 2 | abc |
| 3 | def |
| 4 | ghi |
| 5 | jkl |
| 6 | mno |
| 7 | pqrs |
| 8 | tuv |
| 9 | wxyz |

**示例 1：**

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

**示例 2：**

```
输入：digits = ""
输出：[]
```

**示例 3：**

```
输入：digits = "2"
输出：["a","b","c"]
```

**提示：**
- `0 <= digits.length <= 4`
- `digits[i]` 是范围 `['2', '9']` 的一个数字。

---

## 核心思想

### 把问题抽象成决策树

以 `digits = "23"` 为例，决策树如下：

```
                              ""  (根节点，空字符串)
                    /         |         \
     第1位(2)     a           b           c     ← 每个分支 = 在"abc"中选一个字母
                / | \       / | \       / | \
     第2位(3)  d  e  f     d  e  f     d  e  f   ← 每个分支 = 在"def"中选一个字母
               |  |  |     |  |  |     |  |  |
              ad ae af    bd be bf    cd ce cf   ← 叶子节点 = 最终组合
```

每一条**从根到叶子的完整路径**就是一个字母组合。回溯算法要做的就是**遍历这棵决策树的所有叶子节点**。

### 为什么是回溯？

回溯 = **DFS 遍历决策树 + 撤销选择**

```
回溯三步走：
1. 做选择：在当前位置，选一个字母加入路径
2. 递归：带着更新后的路径，进入下一层决策
3. 撤销选择：回到当前位置，尝试下一个字母
```

用"字符串拼接"替代显式的"撤销"可以简化代码——因为 `"a" + "d"` 生成的是新字符串 `"ad"`，不会影响下一次拼接 `"a" + "e"`。

### 两种思路的比较

```
思路一：回溯 DFS（递归）     ← 经典解法，回溯基本功
思路二：迭代 BFS（笛卡尔积）  ← 更直观，逐层拼接

两者的本质相同：都是在 N 个集合中各选一个元素，求笛卡尔积。
回溯是"深度优先"走完一条路再走下一条；
迭代是"广度优先"一层一层地构建。
```

---

## 思路分析

### 解法一：回溯 DFS（递归）⭐ 经典回溯模板

**核心思路**：递归遍历决策树。`backtrack(index, path)` 表示当前处理到第 `index` 个数字，已构建的字符串为 `path`。当 `index == digits.length` 时，说明走完所有层，将 `path` 加入结果集。

```
算法框架（回溯 DFS）：

1. 建立数字 → 字母映射表：phoneMap = {2:"abc", 3:"def", ...}

2. 定义回溯函数 backtrack(index, path)：
   - 终止条件：index === digits.length
     → result.push(path)，返回
   - 获取当前数字对应的字母串 letters = phoneMap[digits[index]]
   - for 每个字母 ch in letters：
       backtrack(index + 1, path + ch)   // 做选择 + 递归，字符串不可变所以不需要显式撤销

3. 主函数：
   - if digits 为空 → return []
   - backtrack(0, "")
   - return result
```

### 解法二：迭代法（笛卡尔积）⭐ 更直观

**核心思路**：从 `[""]` 开始，每遇到一个新数字，就将已有组合与当前数字的字母列表做笛卡尔积。

```
算法框架（迭代法）：

1. 如果 digits 为空 → return []
2. result = [""]   // 初始只有一个空字符串
3. for 每个数字 digit in digits：
     letters = phoneMap[digit]
     temp = []
     for 每个已有组合 s in result：
       for 每个字母 ch in letters：
         temp.push(s + ch)
     result = temp
4. return result
```

这个过程本质上就是 BFS 逐层构建：第 i 轮 result 中存放的是前 i 个数字能构成的所有组合。

---

## 代码实现

### JavaScript 版本

#### 方法一：回溯 DFS（递归）

```javascript
/**
 * @param {string} digits
 * @return {string[]}
 */
var letterCombinations = function(digits) {
    if (digits.length === 0) return [];

    const phoneMap = {
        '2': 'abc', '3': 'def', '4': 'ghi',
        '5': 'jkl', '6': 'mno', '7': 'pqrs',
        '8': 'tuv', '9': 'wxyz'
    };

    const result = [];

    function backtrack(index, path) {
        // 终止条件：已经处理完所有数字
        if (index === digits.length) {
            result.push(path);
            return;
        }

        // 获取当前数字对应的所有字母
        const letters = phoneMap[digits[index]];

        // 尝试每个字母
        for (const ch of letters) {
            backtrack(index + 1, path + ch);
        }
    }

    backtrack(0, '');
    return result;
};
```

#### 方法二：迭代法（笛卡尔积）

```javascript
/**
 * @param {string} digits
 * @return {string[]}
 */
var letterCombinations = function(digits) {
    if (digits.length === 0) return [];

    const phoneMap = {
        '2': 'abc', '3': 'def', '4': 'ghi',
        '5': 'jkl', '6': 'mno', '7': 'pqrs',
        '8': 'tuv', '9': 'wxyz'
    };

    let result = [''];

    for (const digit of digits) {
        const letters = phoneMap[digit];
        const temp = [];

        for (const s of result) {
            for (const ch of letters) {
                temp.push(s + ch);
            }
        }

        result = temp;
    }

    return result;
};
```

### Python 版本

#### 方法一：回溯 DFS（递归）

```python
from typing import List


class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        phone_map = {
            '2': 'abc', '3': 'def', '4': 'ghi',
            '5': 'jkl', '6': 'mno', '7': 'pqrs',
            '8': 'tuv', '9': 'wxyz'
        }

        result = []

        def backtrack(index: int, path: str) -> None:
            """处理第 index 个数字，当前已构建的字符串为 path"""
            if index == len(digits):
                result.append(path)
                return

            letters = phone_map[digits[index]]
            for ch in letters:
                backtrack(index + 1, path + ch)

        backtrack(0, '')
        return result
```

#### 方法二：迭代法（笛卡尔积）

```python
from typing import List


class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        phone_map = {
            '2': 'abc', '3': 'def', '4': 'ghi',
            '5': 'jkl', '6': 'mno', '7': 'pqrs',
            '8': 'tuv', '9': 'wxyz'
        }

        result = ['']

        for digit in digits:
            letters = phone_map[digit]
            temp = []
            for s in result:
                for ch in letters:
                    temp.append(s + ch)
            result = temp

        return result
```

#### 方法三：Python 一行流（itertools.product）

```python
from itertools import product
from typing import List


class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        phone_map = {
            '2': 'abc', '3': 'def', '4': 'ghi',
            '5': 'jkl', '6': 'mno', '7': 'pqrs',
            '8': 'tuv', '9': 'wxyz'
        }

        return [''.join(combo) for combo in product(*[phone_map[d] for d in digits])]
```

> `itertools.product(*iterables)` 直接计算多个可迭代对象的笛卡尔积，一行搞定。面试时可以先写回溯版本展示基本功，再提这个语法糖表示你是 Python 熟手。

---

## 逐步推演

以 `digits = "23"` 为例，演示两种方法的完整过程。

### 回溯 DFS 推演

```
调用 backtrack(0, "")

────────────────────────────────────────────────────────────
第 1 层：index=0, path="", digits[0]="2", letters="abc"

  尝试 'a' → backtrack(1, "a")
  ─────────────────────────────────────────────────────────
  第 2 层：index=1, path="a", digits[1]="3", letters="def"

    尝试 'd' → backtrack(2, "ad")
      第 3 层：index=2 === len("23")=2
        → result.push("ad")  ✅ 到达第一个叶子
        → return（回溯到第 2 层，继续尝试下一个字母）

    尝试 'e' → backtrack(2, "ae")
        → result.push("ae")  ✅
        → return

    尝试 'f' → backtrack(2, "af")
        → result.push("af")  ✅
        → return

    'a' 分支全部尝试完毕 → return（回溯到第 1 层）
  ─────────────────────────────────────────────────────────

  尝试 'b' → backtrack(1, "b")
    类似地产生 "bd", "be", "bf"  ✅

  尝试 'c' → backtrack(1, "c")
    类似地产生 "cd", "ce", "cf"  ✅

  '2' 的所有字母尝试完毕 → return（回溯到最外层）

────────────────────────────────────────────────────────────
最终 result = ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

### 回溯调用栈可视化

```
backtrack(0, "")
├── backtrack(1, "a")
│   ├── backtrack(2, "ad") → push "ad"
│   ├── backtrack(2, "ae") → push "ae"
│   └── backtrack(2, "af") → push "af"
├── backtrack(1, "b")
│   ├── backtrack(2, "bd") → push "bd"
│   ├── backtrack(2, "be") → push "be"
│   └── backtrack(2, "bf") → push "bf"
└── backtrack(1, "c")
    ├── backtrack(2, "cd") → push "cd"
    ├── backtrack(2, "ce") → push "ce"
    └── backtrack(2, "cf") → push "cf"

共 3 × 3 = 9 个叶子节点 = 9 种组合
```

### 迭代法推演

```
输入：digits = "23"
初始：result = [""]

────────────────────────────────────────────────────────────
第 1 轮：处理 digits[0] = "2", letters = "abc"

  result = [""]
  
  s=""  ×  ch='a' → "a"  push
  s=""  ×  ch='b' → "b"  push
  s=""  ×  ch='c' → "c"  push
  
  新 result = ["a", "b", "c"]

────────────────────────────────────────────────────────────
第 2 轮：处理 digits[1] = "3", letters = "def"

  result = ["a", "b", "c"]

  s="a" × ch='d' → "ad" push
  s="a" × ch='e' → "ae" push
  s="a" × ch='f' → "af" push

  s="b" × ch='d' → "bd" push
  s="b" × ch='e' → "be" push
  s="b" × ch='f' → "bf" push

  s="c" × ch='d' → "cd" push
  s="c" × ch='e' → "ce" push
  s="c" × ch='f' → "cf" push

  新 result = ["ad","ae","af","bd","be","bf","cd","ce","cf"]

────────────────────────────────────────────────────────────
digits 处理完毕，返回 result ✅
```

### 边界情况推演

```
情况 1：digits = ""（空输入）
  → 直接返回 []（不是 [""]！）
  → 空输入表示没有任何数字，没有组合是合理的

情况 2：digits = "2"（单个数字）
  → 回溯法：第 1 层遍历 abc，index+1=1==len("2") → 直接 push
  → 结果：["a","b","c"]

情况 3：digits = "79"（含 4 个字母的数字 7 和 9）
  → "7" → "pqrs"（4 个字母）
  → "9" → "wxyz"（4 个字母）
  → 总组合数：4 × 4 = 16
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 说明 |
|------|-----------|-----------|------|
| **回溯 DFS** | O(3^m × 4^n) | O(m + n) | 每个叶子产生一个结果，递归栈深 = 数字长度 |
| **迭代法** | O(3^m × 4^n) | O(3^m × 4^n) | 结果集需要存储所有组合 |

### 详细分析

设输入中有 `m` 个对应 3 个字母的数字（2,3,4,5,6,8），`n` 个对应 4 个字母的数字（7,9）。

**时间复杂度：**

- 每个叶子节点（最终组合）都需要被构造一次，叶子节点总数 = 3^m × 4^n。
- 每条路径上构建字符串的代价是 O(m + n)（每次拼接新字符串），但也可以认为每次拼接都是 O(1)（在递归中逐步构建，总共只产生 3^m × 4^n 个结果字符串）。
- **总计**：O(3^m × 4^n × (m + n))。当 digits.length ≤ 4 时，最坏情况 4^4 = 256 个组合，完全可行。

**空间复杂度：**

- **回溯法**：递归栈深度 = O(m + n)，即数字的个数。输出结果不计入辅助空间。
- **迭代法**：临时数组 `temp` 和 `result` 的最大大小为 O(3^m × 4^n)，即所有组合的数量。

> 💡 题目约束 `digits.length ≤ 4`，所以最坏情况也只有 4^4 = 256 种组合，两种方法在空间上都完全够用。但如果数字长度扩展到 10+，迭代法的高峰内存占用会成为一个问题。

---

## 举一反三

### 本题在"回溯"体系中的位置

```
LeetCode 51/52. N 皇后（Hard）
  → 二维棋盘回溯 + 复杂剪枝条件
  ↑
LeetCode 39/40. 组合总和（Medium）
  → 回溯 + 可重复选择 / 去重
  ↑
LeetCode 22. 括号生成（Medium）
  → 回溯 + 左右括号数量约束
  ↑
LeetCode 17. 电话号码的字母组合（Medium）⭐ 本题
  → 回溯入门：多个集合各选一个元素，无剪枝
  ↓
LeetCode 77. 组合（Medium）
  → 回溯 + 剪枝（剩余元素不够时提前停止）
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| **22. 括号生成** | 回溯 + 约束剪枝 | 同样是字符串构建型回溯，但多了合法性约束 |
| **39. 组合总和** | 回溯 + 可重复选择 | 回溯框架相同，多了"一个元素可选多次" |
| **77. 组合** | 回溯 + 剪枝 | 最基础的回溯题型，不需要映射表 |
| **46. 全排列** | 回溯 + visited 标记 | 回溯框架相同，但需要避免同一元素重复使用 |
| **78. 子集** | 回溯 / 迭代 | 同样是组合枚举，但每个元素只有选/不选两种状态 |

### 回溯算法的通用模板

```javascript
// 回溯算法通用模板
function backtrack(路径, 选择列表, 当前位置) {
    if (满足终止条件) {
        记录结果;
        return;
    }

    for (选择 of 当前层可选列表) {
        做选择;           // 将选择加入路径
        backtrack(路径, 选择列表, 下一位置);
        撤销选择;          // 将选择从路径中移除
    }
}
```

> **第 17 题是回溯的最简形态**——每个数字层的选择互相独立，不需要 visited 数组、不需要剪枝、不需要去重。把这道题搞懂，回溯的"做选择 → 递归 → 撤销选择"三件套就通了。

### 笛卡尔积思想的延伸

迭代法本质是在求多个集合的笛卡尔积。这个思想在以下场景中也常见：

- **多个筛选条件的组合测试**：生成所有测试用例
- **配置项的枚举**：如颜色 × 尺寸 × 材质的所有组合
- **多层 for 循环的扁平化**：用数组 `result` 作为累加器，逐层展开

---

## 总结

电话号码的字母组合是一道经典的回溯入门题，核心就两层：

1. **回溯 DFS**：在决策树上做深度优先搜索。每一位数字是一层决策，3~4 个字母是候选选项。走完所有层 = 找到一个组合。

2. **迭代法（笛卡尔积）**：从 `[""]` 开始，逐层将已有结果与当前字母集合做拼接。更直观，但空间占满。

| 步骤 | 操作 |
|------|------|
| ① | 建立数字 → 字母映射表 |
| ② | 判空：digits 为空 → 直接返回 `[]` |
| ③ | 回溯 / 迭代：遍历决策树或逐层构建组合 |
| ④ | 返回所有组合 |

**三步记忆法**：
1. **建映射**：`{2:"abc", 3:"def", ..., 9:"wxyz"}`
2. **写回溯**：`backtrack(index, path)` → 终止条件 `index == digits.length`
3. **做选择**：遍历 `phoneMap[digits[index]]` 的每个字母，递归到 `index + 1`

> **记住一句话：电话号码的字母组合 = 多集合笛卡尔积 = 决策树的 DFS 遍历。回溯核心三件套：做选择 → 递归 → 撤销选择（本题字符串不可变，天然支持隐式撤销）。**

在纸上画出 `"23"` 的决策树，手动走一遍 `backtrack(0, "")` 的递归过程——你会发现回溯其实非常直觉。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 22. 括号生成](https://leetcode.com/problems/generate-parentheses/)（回溯+剪枝） | [LeetCode 39. 组合总和](https://leetcode.com/problems/combination-sum/)（回溯+可重复选） | [LeetCode 46. 全排列](https://leetcode.com/problems/permutations/)（回溯+visited） | 回溯系列持续更新中，关注不迷路。
