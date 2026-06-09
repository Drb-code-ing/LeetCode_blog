---
title: LeetCode 224. 基本计算器
tags: [栈, 表达式求值, 括号, 递归]
difficulty: Hard
category: 栈
date: 2026-06-09
---

# LeetCode 224. 基本计算器 —— 栈的经典应用：括号展开与符号状态管理

## 前言

表达式求值是栈的经典应用场景。LeetCode 224 **基本计算器**（Basic Calculator）是一道 Hard 题，但它考察的核心思想非常纯粹：**如何处理括号嵌套带来的"上下文切换"**。

这道题只包含 `+`、`-`、`(`、`)` 四种运算符（没有乘除），但引入了两个挑战：

1. **括号嵌套**：进入括号时要保存外层计算结果，等括号内算完再合并回来
2. **一元负号**：`-` 可以出现在表达式开头或 `(` 后面（如 `-(2+3)` 或 `1-(-2)`）

本文将用三种思路逐步深入：**单栈法（括号展开）** → **递归法（隐式栈）** → **双栈法（通用表达式求值）**，每种方法附带 JavaScript 和 Python 实现。

---

## 问题描述

### LeetCode 224. Basic Calculator（基本计算器）

> 给你一个字符串表达式 `s`，请你实现一个基本计算器来计算并返回它的值。
>
> 注意：不允许使用任何将字符串作为数学表达式计算的内置函数，比如 `eval()`。

**示例：**

```
输入：s = "1 + 1"
输出：2

输入：s = " 2-1 + 2 "
输出：3

输入：s = "(1+(4+5+2)-3)+(6+8)"
输出：23
```

**约束条件：**
- `1 <= s.length <= 3 * 10^5`
- `s` 由数字、`'+'`、`'-'`、`'('`、`')'`、`' '` 组成
- `s` 表示一个有效的表达式
- `'+'` **不会**用作一元运算符（即不会出现 `+5` 这种写法）
- `'-'` **可以**用作一元运算符（即 `-5` 或 `-(2+3)` 是合法的）
- 数字均为非负整数
- 计算结果保证在 32 位有符号整数范围内

---

## 核心思想

### 括号 = 上下文切换

表达式求值的核心难点在于括号：遇到 `(` 时，你需要"暂停"当前计算，先算出括号内的结果，然后回来继续。这不是和函数调用一模一样吗？

```
表达式: 3 + ( 4 + 5 ) - 2
              ↓
         "暂停"外层计算
         进入括号内部算 4+5=9
              ↓
         回到外层: 3 + 9 - 2 = 10
```

这恰好对应两种实现策略：

```
策略一：用显式栈保存上下文 → 单栈法（手动模拟"调用栈"）
策略二：用递归/函数调用栈 → 递归法（让语言帮你管栈）
策略三：通用的运算符+操作数双栈 → 双栈法（可扩展到乘除）
```

### 一元负号怎么处理？

一元负号（如 `-2` 或 `-(3+4)`）是本题的隐藏难点。两种通用处理方式：

1. **视为 `0 - expr`**：在负号前补一个 0，将一元负号转为二元减法
2. **用 sign 变量维护符号**：遍历时维护当前符号（`+1` 或 `-1`），遇到数字时 `result += sign * num`

本文三种解法均采用 **sign 变量法**，它能自然地处理一元负号，无需预处理字符串。

---

## 思路分析

### 解法一：单栈法（括号展开）⭐ 推荐

核心思路：用 **一个栈** 保存"进入括号前的上下文"。

什么是上下文？进入 `(` 之前，有两个东西需要记住：
- **之前算出的结果** `result`（外层累积值）
- **紧贴在 `(` 前面的符号** `sign`（决定括号内整体是加还是减）

遇到 `)` 时，从栈中恢复这两个值，与括号内的计算结果合并。

```
遍历表达式 "3+(4+5)-2"：

状态变量：result=当前结果, sign=当前符号(+1/-1)
栈：保存"进入括号前的 (result, sign)"

读到 3     → result = 0 + 1*3 = 3
读到 +     → sign = +1
读到 (     → 入栈: (result=3, sign=+1)；重置 result=0, sign=+1
读到 4     → result = 0 + 1*4 = 4
读到 +     → sign = +1
读到 5     → result = 4 + 1*5 = 9
读到 )     → 出栈: sign=+1, old_result=3
             result = 3 + 1*9 = 12
读到 -     → sign = -1
读到 2     → result = 12 + (-1)*2 = 10

最终 result = 10 ✅
```

处理流程：

| 字符类型 | 操作 |
|---------|------|
| `0-9` | 解析完整数字，`result += sign * num` |
| `+` | `sign = 1` |
| `-` | `sign = -1` |
| `(` | 将 `result` 和 `sign` 先后入栈；重置 `result=0, sign=1` |
| `)` | `result = stack.pop() * result + stack.pop()`（符号 × 括号内结果 + 外层结果） |
| 空格 | 跳过 |

### 解法二：递归法（利用函数调用栈）

既然括号的嵌套结构天然对应递归，那直接用递归写最干净：

- 遇到 `(` → 递归进入子表达式，返回后累加到结果
- 遇到 `)` → 返回当前结果
- 遇到 `+` / `-` → 更新符号
- 遇到数字 → 解析并累加

核心思想：**函数调用栈本身就是"保存上下文"的栈**——进入递归时，当前的状态被自动保存；返回时，状态自动恢复。

```
表达式: 3+(4+5)-2

evalExpr() [外层]:
  读到 3    → result=3
  读到 +    → sign=+1
  读到 (    → 递归调用 evalExpr() [内层]
              读到 4 → result=4
              读到 + → sign=+1
              读到 5 → result=9
              读到 ) → return 9
  回到外层 → result = 3 + 1*9 = 12
  读到 -    → sign=-1
  读到 2    → result = 12 + (-1)*2 = 10
  表达式结束 → return 10
```

### 解法三：双栈法（通用表达式求值）

双栈法（操作数栈 + 运算符栈）是最通用的表达式求值方案，可以扩展到 `*`、`/`、`^` 等任何运算符。

```
两个栈：
  nums = []   // 操作数栈
  ops = []    // 运算符栈

核心操作 evaluate()：
  从 nums 弹出 b，再弹出 a
  从 ops 弹出 op
  计算 a op b，结果压入 nums

规则：
  - 数字 → 压入 nums
  - '(' → 压入 ops
  - '+' / '-' → 先计算 ops 中已有的 '+' / '-'，再将当前运算符压入 ops
  - ')' → 不断 evaluate() 直到 ops 栈顶是 '('，然后弹出 '('
  - 遍历结束后 → 计算 ops 中剩余运算符

处理一元负号：
  - 如果 '-' 出现在开头或 '(' 后面 → 先向 nums 压入 0
```

> 💡 双栈法虽然代码量稍大，但它是**可扩展的通用方案**——只要调整运算符优先级，就能直接解决 Basic Calculator II（加了 `*` `/`）。

---

## 代码实现

### JavaScript 版本

#### 方法一：单栈法（括号展开）⭐

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var calculate = function(s) {
    let result = 0;       // 当前计算结果
    let sign = 1;         // 当前符号：1 表示 +，-1 表示 -
    const stack = [];     // 保存进入括号前的 (result, sign)
    let i = 0;

    while (i < s.length) {
        const ch = s[i];

        // 跳过空格
        if (ch === ' ') {
            i++;
            continue;
        }

        // 解析数字（可能是多位数）
        if (ch >= '0' && ch <= '9') {
            let num = 0;
            while (i < s.length && s[i] >= '0' && s[i] <= '9') {
                num = num * 10 + (s[i].charCodeAt(0) - 48);
                i++;
            }
            result += sign * num;
            continue; // 注意：i 已经在 while 中前进了，跳过外层 i++
        }

        if (ch === '+') {
            sign = 1;
        } else if (ch === '-') {
            sign = -1;
        } else if (ch === '(') {
            // 保存外层上下文，进入括号
            stack.push(result);
            stack.push(sign);
            result = 0;
            sign = 1;
        } else if (ch === ')') {
            // 合并：外层结果 + 外层符号 × 括号内结果
            // 栈顶顺序：[result, sign]，所以先 pop 出 sign 再 pop 出 result
            result = stack.pop() * result + stack.pop();
        }

        i++;
    }

    return result;
};
```

#### 方法二：递归法

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var calculate = function(s) {
    let i = 0;

    function evalExpr() {
        let result = 0;
        let sign = 1;

        while (i < s.length) {
            const ch = s[i];

            if (ch === ' ') {
                i++;
                continue;
            }

            if (ch >= '0' && ch <= '9') {
                let num = 0;
                while (i < s.length && s[i] >= '0' && s[i] <= '9') {
                    num = num * 10 + (s[i].charCodeAt(0) - 48);
                    i++;
                }
                result += sign * num;
                continue;
            }

            if (ch === '+') {
                sign = 1;
                i++;
            } else if (ch === '-') {
                sign = -1;
                i++;
            } else if (ch === '(') {
                i++;                    // 跳过 '('
                result += sign * evalExpr(); // 递归计算括号内
                // evalExpr 返回时 i 已指向 ')' 的下一个字符
                continue;
            } else if (ch === ')') {
                i++;                    // 跳过 ')'
                break;                  // 返回当前结果
            }
        }

        return result;
    }

    return evalExpr();
};
```

#### 方法三：双栈法（通用表达式求值）

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var calculate = function(s) {
    const nums = [];  // 操作数栈
    const ops = [];   // 运算符栈

    /**
     * 执行一次计算：nums 弹出两个数，ops 弹出一个运算符
     */
    function evaluate() {
        const b = nums.pop();
        const a = nums.pop();
        const op = ops.pop();
        if (op === '+') nums.push(a + b);
        else if (op === '-') nums.push(a - b);
    }

    let i = 0;
    while (i < s.length) {
        const ch = s[i];

        if (ch === ' ') {
            i++;
            continue;
        }

        if (ch >= '0' && ch <= '9') {
            let num = 0;
            while (i < s.length && s[i] >= '0' && s[i] <= '9') {
                num = num * 10 + (s[i].charCodeAt(0) - 48);
                i++;
            }
            nums.push(num);
            continue;
        }

        if (ch === '(') {
            ops.push(ch);
        } else if (ch === ')') {
            // 计算到最近的 '('
            while (ops[ops.length - 1] !== '(') {
                evaluate();
            }
            ops.pop(); // 弹出 '('
        } else if (ch === '+' || ch === '-') {
            // 处理一元负号：如果上一个字符是 '(' 或开头
            if (ch === '-' && (i === 0 || s[i - 1] === '(')) {
                nums.push(0);
            }
            // 计算栈中已有的同优先级运算符
            while (ops.length > 0 && ops[ops.length - 1] !== '(') {
                evaluate();
            }
            ops.push(ch);
        }

        i++;
    }

    // 计算剩余运算符
    while (ops.length > 0) {
        evaluate();
    }

    return nums[0];
};
```

### Python 版本

#### 方法一：单栈法（括号展开）⭐

```python
def calculate(s: str) -> int:
    """方法一：单栈法（括号展开）"""
    result = 0          # 当前计算结果
    sign = 1            # 当前符号：1 表示 +，-1 表示 -
    stack = []          # 保存进入括号前的 (result, sign)
    i = 0

    while i < len(s):
        ch = s[i]

        # 跳过空格
        if ch == ' ':
            i += 1
            continue

        # 解析数字（可能是多位数）
        if ch.isdigit():
            num = 0
            while i < len(s) and s[i].isdigit():
                num = num * 10 + int(s[i])
                i += 1
            result += sign * num
            continue  # i 已在 while 中前进，跳过外层 i += 1

        if ch == '+':
            sign = 1
        elif ch == '-':
            sign = -1
        elif ch == '(':
            # 保存外层上下文，进入括号
            stack.append(result)
            stack.append(sign)
            result = 0
            sign = 1
        elif ch == ')':
            # 合并：外层结果 + 外层符号 × 括号内结果
            # 栈顶顺序：[result, sign]，所以先 pop sign 再 pop result
            result = stack.pop() * result + stack.pop()

        i += 1

    return result
```

#### 方法二：递归法

```python
def calculate(s: str) -> int:
    """方法二：递归法（利用函数调用栈）"""
    n = len(s)
    i = 0

    def eval_expr() -> int:
        nonlocal i
        result = 0
        sign = 1

        while i < n:
            ch = s[i]

            if ch == ' ':
                i += 1
                continue

            if ch.isdigit():
                num = 0
                while i < n and s[i].isdigit():
                    num = num * 10 + int(s[i])
                    i += 1
                result += sign * num
                continue

            if ch == '+':
                sign = 1
                i += 1
            elif ch == '-':
                sign = -1
                i += 1
            elif ch == '(':
                i += 1                      # 跳过 '('
                result += sign * eval_expr() # 递归计算括号内
                # eval_expr 返回时 i 已指向 ')' 的下一个字符
                continue
            elif ch == ')':
                i += 1                      # 跳过 ')'
                break                        # 返回当前结果

        return result

    return eval_expr()
```

#### 方法三：双栈法（通用表达式求值）

```python
def calculate(s: str) -> int:
    """方法三：双栈法（通用表达式求值）"""
    nums = []   # 操作数栈
    ops = []    # 运算符栈

    def evaluate():
        """执行一次计算：nums 弹出两个数，ops 弹出一个运算符"""
        b = nums.pop()
        a = nums.pop()
        op = ops.pop()
        if op == '+':
            nums.append(a + b)
        else:  # op == '-'
            nums.append(a - b)

    i = 0
    while i < len(s):
        ch = s[i]

        if ch == ' ':
            i += 1
            continue

        if ch.isdigit():
            num = 0
            while i < len(s) and s[i].isdigit():
                num = num * 10 + int(s[i])
                i += 1
            nums.append(num)
            continue

        if ch == '(':
            ops.append(ch)
        elif ch == ')':
            # 计算到最近的 '('
            while ops and ops[-1] != '(':
                evaluate()
            ops.pop()  # 弹出 '('
        elif ch in ('+', '-'):
            # 处理一元负号：如果上一个字符是 '(' 或开头
            if ch == '-' and (i == 0 or s[i - 1] == '('):
                nums.append(0)
            # 计算栈中已有的同优先级运算符
            while ops and ops[-1] != '(':
                evaluate()
            ops.append(ch)

        i += 1

    # 计算剩余运算符
    while ops:
        evaluate()

    return nums[0]
```

---

## 逐步推演

### 示例一：`s = "1 + 1"`（无括号基础情况）

```
方法一（单栈法）推演：

初始化: result=0, sign=1, stack=[]

=== i=0: ch='1' ===
  解析数字: num=1
  result = 0 + 1*1 = 1, i=1

=== i=1: ch=' ' ===
  跳过, i=2

=== i=2: ch='+' ===
  sign = 1, i=3

=== i=3: ch=' ' ===
  跳过, i=4

=== i=4: ch='1' ===
  解析数字: num=1
  result = 1 + 1*1 = 2, i=5

最终: result = 2 ✅
```

### 示例二：`s = "(1+(4+5+2)-3)+(6+8)"`（多层嵌套）

```
方法一（单栈法）推演：

初始化: result=0, sign=1, stack=[]

=== i=0: ch='(' ===
  入栈: result=0, sign=1
  stack = [0, 1]
  重置: result=0, sign=1, i=1

=== i=1: ch='1' ===
  num=1, result=0+1*1=1, i=2

=== i=2: ch='+' ===
  sign=1, i=3

=== i=3: ch='(' ===
  入栈: result=1, sign=1
  stack = [0, 1, 1, 1]
  重置: result=0, sign=1, i=4

=== i=4: ch='4' ===
  num=4, result=0+1*4=4, i=5

=== i=5: ch='+' ===
  sign=1, i=6

=== i=6: ch='5' ===
  num=5, result=4+1*5=9, i=7

=== i=7: ch='+' ===
  sign=1, i=8

=== i=8: ch='2' ===
  num=2, result=9+1*2=11, i=9

=== i=9: ch=')' ===
  stack.pop() = 1 (sign)
  stack.pop() = 1 (old_result)
  result = 1*11 + 1 = 12
  stack = [0, 1], i=10

=== i=10: ch='-' ===
  sign=-1, i=11

=== i=11: ch='3' ===
  num=3, result=12+(-1)*3=9, i=12

=== i=12: ch=')' ===
  stack.pop() = 1 (sign)
  stack.pop() = 0 (old_result)
  result = 1*9 + 0 = 9
  stack = [], i=13

=== i=13: ch='+' ===
  sign=1, i=14

=== i=14: ch='(' ===
  入栈: result=9, sign=1
  stack = [9, 1]
  重置: result=0, sign=1, i=15

=== i=15: ch='6' ===
  num=6, result=0+1*6=6, i=16

=== i=16: ch='+' ===
  sign=1, i=17

=== i=17: ch='8' ===
  num=8, result=6+1*8=14, i=18

=== i=18: ch=')' ===
  stack.pop() = 1 (sign)
  stack.pop() = 9 (old_result)
  result = 1*14 + 9 = 23
  stack = [], i=19

最终: result = 23 ✅
```

### 示例三：`s = "-(2+3)"`（一元负号）

```
方法一（单栈法）推演：

初始化: result=0, sign=1, stack=[]

=== i=0: ch='-' ===
  这是一元负号！sign = -1, i=1
  （关键：result 仍然是 0，符号已翻转）

=== i=1: ch='(' ===
  入栈: result=0, sign=-1
  stack = [0, -1]
  重置: result=0, sign=1, i=2

=== i=2: ch='2' ===
  num=2, result=0+1*2=2, i=3

=== i=3: ch='+' ===
  sign=1, i=4

=== i=4: ch='3' ===
  num=3, result=2+1*3=5, i=5

=== i=5: ch=')' ===
  stack.pop() = -1 (sign = 括号外的符号！)
  stack.pop() = 0  (old_result)
  result = (-1)*5 + 0 = -5
  stack = [], i=6

最终: result = -5 ✅
```

> 💡 **关键洞察**：上面的一元负号被自然地处理了！因为遇到 `-` 时我们只是设置 `sign = -1`，然后遇到 `(` 时将 `sign = -1` 一起入栈。出栈时 `result = (-1) × 5 + 0 = -5`，完美！

### 示例四：`s = "1-(-2)"`（括号内的负号）

```
方法一（单栈法）推演：

初始化: result=0, sign=1

=== i=0: ch='1' ===
  num=1, result=0+1*1=1, i=1

=== i=1: ch='-' ===
  sign=-1, i=2

=== i=2: ch='(' ===
  入栈: result=1, sign=-1
  stack = [1, -1]
  重置: result=0, sign=1, i=3

=== i=3: ch='-' ===
  括号内的负号 → sign=-1, i=4

=== i=4: ch='2' ===
  num=2, result=0+(-1)*2=-2, i=5

=== i=5: ch=')' ===
  stack.pop() = -1 (外层 sign)
  stack.pop() = 1  (外层 result)
  result = (-1)*(-2) + 1 = 3
  stack = [], i=6

最终: result = 3 ✅
验证: 1-(-2) = 1+2 = 3 ✅
```

### 方法二（递归法）推演同一示例

```
表达式: "1-(-2)"
调用 evalExpr():

i=0: ch='1', num=1, result=0+1*1=1, i=1
i=1: ch='-', sign=-1, i=2
i=2: ch='(', i=3 → 递归调用 evalExpr()
      i=3: ch='-', sign=-1, i=4
      i=4: ch='2', num=2, result=0+(-1)*2=-2, i=5
      i=5: ch=')', i=6, break → return -2
回到外层: result = 1 + (-1)*(-2) = 3
i=6: 循环结束 → return 3 ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **单栈法** ⭐ | O(n) | O(n) | 最简洁，只有一层循环 | 需要理解 sign 和 result 的 push/pop 顺序 |
| **递归法** | O(n) | O(n) | 代码结构清晰，天然反映嵌套 | 递归深度可能引起栈溢出（极深嵌套时） |
| **双栈法** | O(n) | O(n) | 通用性强，可扩展到 `*` `/` | 代码量稍大，需要处理运算符优先级 |

> 注：n = 字符串长度。每个字符只被遍历一次。空间复杂度 O(n) 来源于栈的深度（最深嵌套层数 = n/2，即全括号情况）。

---

## 三种方法的本质联系

```
单栈法 ──── 手动保存 (result, sign) 到栈 ────┐
                                              │
递归法 ──── 利用函数调用栈自动保存 ─────────┤ → 本质相同
                                              │    都是"上下文切换"
双栈法 ──── 分别保存操作数和运算符 ─────────┘
             ↑
            通用表达式求值框架
            调整优先级即可处理 + - * / ^
```

---

## 举一反三

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [227. 基本计算器 II](https://leetcode.com/problems/basic-calculator-ii/) | `+` `-` `*` `/`，无括号 | 双栈法加优先级即可无缝扩展 |
| [772. 基本计算器 III](https://leetcode.com/problems/basic-calculator-iii/) | `+` `-` `*` `/` `(` `)` 全有 | 双栈法直接通杀 |
| [150. 逆波兰表达式求值](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | 后缀表达式（无括号） | 表达式求值的另一种范式 |
| [394. 字符串解码](https://leetcode.com/problems/decode-string/) | 嵌套结构 + 栈保存上下文 | 完全相同的"入栈保存→出栈恢复"模式 |
| [20. 有效的括号](https://leetcode.com/problems/valid-parentheses/) | 栈匹配括号 | 括号处理的最简原型 |

### 表达式求值通用框架

```
表达式求值问题三步走：
1. 选择栈策略：
   - 只有 + - 和括号 → 单栈法最简洁
   - 有 * / 优先级 → 双栈法（运算符栈+操作数栈）
   - 嵌套结构天然递归 → 递归法最直观

2. 处理一元负号：
   - 维护 sign 变量（推荐，本题采用）
   - 或在负号前补 0（将一元转为二元）

3. 括号处理：
   - 显式栈：push 当前上下文 → 重置 → 计算 → pop 恢复
   - 递归法：函数调用 → 返回 → 合并
```

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **224 基本计算器** | 🔴 Hard | 栈保存上下文 + sign 状态管理 | 单栈法 |

三种方法层层递进：

1. **单栈法（推荐）**：最简洁，用栈保存进入括号前的 `(result, sign)`。遇到 `(` push，遇到 `)` pop 并合并。**核心口诀：push 保存上下文，pop 恢复并合并。**

2. **递归法**：利用函数调用栈自动管理上下文。遇到 `(` 递归进入，遇到 `)` 返回。代码结构最贴近问题的自然描述。

3. **双栈法**：操作数栈 + 运算符栈，通用表达式求值框架。学会了这一个，224 和 227 全部通杀。

三个方法的本质是一样的：**括号就是上下文的保存与恢复，而栈（无论是显式还是隐式）就是这个"保存-恢复"机制的最佳载体。**

> **记住一句话：括号 = 上下文切换 = push 当前状态，算完括号内，pop 恢复状态。这是表达式求值题的万能钥匙。**
