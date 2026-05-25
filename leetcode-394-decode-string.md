## 前言

栈（Stack）是解决字符串处理问题的一把利器，特别是当问题涉及**嵌套结构**时——括号匹配、表达式求值、HTML 解析，无一不依赖栈的"先进后出"特性。

**字符串解码**（LeetCode 394）是栈应用领域最经典的题目之一：给你一个经过编码的字符串，按照 `k[encoded_string]` 的规则重复展开，你需要处理嵌套层叠的情况。这道题不仅考察栈的使用，还考验对字符串状态管理的理解，是面试中的高频题。

---

## 问题描述

### LeetCode 394. Decode String（字符串解码）

> 给定一个经过编码的字符串，返回它解码后的字符串。
>
> 编码规则为: `k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。
>
> 你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
> 此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 `k`，例如不会出现像 `3a` 或 `2[4]` 的输入。

**示例：**

```
输入：s = "3[a]2[bc]"
输出："aaabcbc"

输入：s = "3[a2[c]]"
输出："accaccacc"

输入：s = "2[abc]3[cd]ef"
输出："abcabccdcdcdef"

输入：s = "abc3[cd]xyz"
输出："abccdcdcdxyz"
```

---

## 栈的核心思想

解码过程的核心难点在于**嵌套**：`3[a2[c]]` 需要先解码内部的 `2[c]` 得到 `cc`，然后和外层的 `a` 拼接，再重复 3 次。

这种"从最内层开始，层层向外"的处理顺序，天然对应栈的 LIFO（后进先出）特性：

1. 遇到 `[` 时——将当前已构建的字符串和重复次数**入栈保存**，然后重置当前字符串
2. 遇到 `]` 时——从栈中弹出之前的字符串和重复次数，将当前字符串重复后**拼接到之前的字符串后面**
3. 遇到数字——累加构建重复次数（可能有多位数字）
4. 遇到字母——追加到当前正在构建的字符串

```
整体流程示意图：

输入: 3 [ a 2 [ c ] ]
      ↓
     读取数字 3
      ↓
     遇到 '[' → 将当前字符串("")和数字(3)入栈，重置
      ↓
     读取字母 'a'
      ↓
     遇到数字 2
      ↓
     遇到 '[' → 将当前字符串("a")和数字(2)入栈，重置
      ↓
     读取字母 'c'
      ↓
     遇到 ']' → 出栈得到("a", 2)，当前"c"重复2次→"cc"，拼接到"a"后→"acc"
      ↓
     遇到 ']' → 出栈得到("", 3)，当前"acc"重复3次→"accaccacc"
```

---

## 思路分析：字符串解码

### 解法一：双栈法（数字栈 + 字符串栈）

维护两个栈：
- **数字栈** `numStack`：存储每个 `[` 前面的重复次数
- **字符串栈** `strStack`：存储每个 `[` 之前已经构建好的字符串

同时维护两个变量：
- `currNum`：累加当前正在读取的数字
- `currStr`：构建当前层级的字符串

遍历字符串的每个字符：

| 字符类型 | 操作 |
|---------|------|
| `0-9` | 累加到 `currNum`（注意处理多位数） |
| `[` | 将 `currStr` 和 `currNum` 分别压入对应栈，然后重置 |
| `]` | 出栈：`prevStr = strStack.pop()`，`num = numStack.pop()`，将 `currStr.repeat(num)` 拼接到 `prevStr` 后 |
| 字母 | 直接追加到 `currStr` |

### 解法二：递归法（DFS）

递归的本质是利用**系统调用栈**代替显式栈，每次遇到 `[` 就递归进入新一层，遇到 `]` 就返回结果。

用一个指针 `i` 全局遍历字符串，遇到数字就构建重复次数，遇到字母就追加，遇到 `[` 就递归调用，遇到 `]` 就返回当前层的结果。

---

## 代码实现

### JavaScript 版本（双栈法）

```javascript
/**
 * @param {string} s
 * @return {string}
 */
var decodeString = function(s) {
    const numStack = [];   // 存储重复次数
    const strStack = [];   // 存储每一层的字符串
    let currNum = 0;       // 当前正在读取的数字
    let currStr = '';      // 当前正在构建的字符串

    for (const ch of s) {
        if (ch >= '0' && ch <= '9') {
            // 构建多位数：例如 "12" → 1*10 + 2 = 12
            currNum = currNum * 10 + (ch.charCodeAt(0) - '0'.charCodeAt(0));
        } else if (ch === '[') {
            // 遇到 '['：将当前状态入栈，然后重置
            numStack.push(currNum);
            strStack.push(currStr);
            currNum = 0;
            currStr = '';
        } else if (ch === ']') {
            // 遇到 ']'：出栈，构建当前层的重复字符串
            const repeatTimes = numStack.pop();
            const prevStr = strStack.pop();
            currStr = prevStr + currStr.repeat(repeatTimes);
        } else {
            // 普通字母，直接追加
            currStr += ch;
        }
    }

    return currStr;
};
```

### 递归法实现

```javascript
/**
 * @param {string} s
 * @return {string}
 */
var decodeString = function(s) {
    // 全局指针，在递归函数间共享状态
    let i = 0;

    function dfs() {
        let result = '';
        let num = 0;

        while (i < s.length) {
            const ch = s[i];

            if (ch >= '0' && ch <= '9') {
                num = num * 10 + (ch.charCodeAt(0) - '0'.charCodeAt(0));
                i++;
            } else if (ch === '[') {
                i++; // 跳过 '['
                const innerStr = dfs();    // 递归解码内部字符串
                result += innerStr.repeat(num);
                num = 0;                   // 重置数字
            } else if (ch === ']') {
                i++; // 跳过 ']'
                return result;             // 返回当前层结果
            } else {
                result += ch;
                i++;
            }
        }

        return result;
    }

    return dfs();
};
```

---

## 复杂度分析

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | O(n) | 每个字符仅被处理一次，n 为解码后字符串的长度 |
| 空间复杂度 | O(n) | 栈空间在最坏情况下（全嵌套）存储所有层级的字符串，n 为解码后字符串的长度 |

时间复杂度的精确表述是 O(输出字符串的长度)，因为重复展开操作本身就会产生线性于输出长度的结果。

---

## 优化方向

### 1. 单栈法（统一存储）

将数字和字符串作为**一个对象**存入同一个栈中，减少栈维护的数量：

```javascript
var decodeString = function(s) {
    const stack = [];
    let currStr = '';
    let currNum = 0;

    for (const ch of s) {
        if (ch >= '0' && ch <= '9') {
            currNum = currNum * 10 + (ch.charCodeAt(0) - '0'.charCodeAt(0));
        } else if (ch === '[') {
            stack.push({ str: currStr, num: currNum });
            currStr = '';
            currNum = 0;
        } else if (ch === ']') {
            const { str, num } = stack.pop();
            currStr = str + currStr.repeat(num);
        } else {
            currStr += ch;
        }
    }

    return currStr;
};
```

### 2. 正则替换（不推荐面试使用）

利用正则 + 递归替换的方式可以一行实现，但可读性差且效率低：

```javascript
var decodeString = function(s) {
    const pattern = /(\d+)\[([^\[\]]*)\]/g;
    while (pattern.test(s)) {
        s = s.replace(pattern, (_, num, str) => str.repeat(+num));
    }
    return s;
};
```

这种方法每次只处理最内层，需要多次遍历，效率远不如栈解法。

---

## 举一反三

理解了"用栈处理嵌套字符串"，以下题目都可以用类似的思路解决：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 20. 有效的括号** | 栈的入门题，遇到左括号入栈，右括号匹配出栈 |
| **LeetCode 71. 简化路径** | 用栈处理路径中的 `..` 和 `.` 跳转 |
| **LeetCode 224. 基本计算器** | 双栈处理表达式，数字栈 + 操作符栈，遇到括号时计算子表达式 |
| **LeetCode 736. Lisp 语法解析** | 嵌套作用域求值，栈保存当前作用域的变量 |
| **LeetCode 1087. 花括号展开** | 类似字符串解码的展开逻辑，处理嵌套括号 |

它们的共同模式：**遇到嵌套结构，就用栈保存上下文；遇到闭合标志，就出栈恢复上下文。**

```
function solveNested(input) {
    const stack = [];
    let current = 初始值;

    for (const ch of input) {
        if (ch === 进入嵌套的标志) {
            stack.push(保存当前上下文);
            current = 重置;
        } else if (ch === 退出嵌套的标志) {
            const prev = stack.pop();
            current = 合并(prev, current);
        } else {
            正常处理;
        }
    }

    return current;
}
```

---

## 总结

字符串解码这道题的精髓在于一句话：**"遇到 `[` 入栈存状态，遇到 `]` 出栈恢复状态"**。

两个关键点值得反复体会：

1. **数字的多位处理**：`currNum = currNum * 10 + digit`——这是字符串转整数的标准手法，面试中忘记处理多位数是常见失误
2. **栈保存的是"之前的上下文"**：每次遇到 `[` 时入栈的，是**进入当前括号之前**已经构建好的字符串和数字；出栈时，把当前层展开的结果拼回到之前的状态上

面试试这道题时，建议先用双栈法写出清晰版本，如果有时间再讨论递归版本。双栈法不仅直观，而且不易出错——每个字符只处理一次，逻辑线性清晰，是面试中的安全选择。

建议在纸上手动模拟一遍 `3[a2[c]]` 的栈变化过程——把每一步的 `numStack`、`strStack`、`currNum`、`currStr` 写下来。纸上跑完一遍，比看十遍代码都管用。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：后续会继续更新栈与递归系列的其他经典题目，关注不迷路。
