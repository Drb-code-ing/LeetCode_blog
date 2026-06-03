---
title: LeetCode 6. Z 字形变换
tags: [字符串, 模拟, 数学]
difficulty: Medium
category: 字符串
date: 2026-06-03
---

# LeetCode 6. Z 字形变换 —— 从模拟到数学公式，三种思路彻底讲透

## 前言

Z 字形变换（ZigZag Conversion）是一道经典的字符串模拟题。题目要求将字符串按 Z 字形排列后逐行读取，看似简单，但如何**高效地找到每一行的字符**是关键。

很多人的第一反应是"画出来再读"——这确实能解题，但有更聪明的办法。本文将从三种思路入手：**暴力模拟（排序法）** → **逐行追踪法（最优）** → **数学公式法**，从直观到高效，彻底吃透 Z 字形变换。

每种方法都附带 JavaScript 和 Python 双语言实现，以及详细的手动推演过程。

---

## 问题描述

### LeetCode 6. ZigZag Conversion（Z 字形变换）

> 将一个给定字符串 `s` 根据给定的行数 `numRows`，以从上往下、从左到右进行 Z 字形排列。
>
> 比如输入字符串为 `"PAYPALISHIRING"`，`numRows = 3` 时，排列如下：
>
> ```
> P   A   H   N
> A P L S I I G
> Y   I   R
> ```
>
> 之后，你的输出需要从左往右逐行读取，产生出一个新的字符串：`"PAHNAPLSIIGYIR"`。
>
> 请你实现这个将字符串进行指定行数变换的函数：`string convert(string s, int numRows);`

**示例：**

```
输入：s = "PAYPALISHIRING", numRows = 3
输出："PAHNAPLSIIGYIR"
解释：
P   A   H   N
A P L S I I G
Y   I   R
→ 逐行读取：PAHNAPLSIIGYIR
```

```
输入：s = "PAYPALISHIRING", numRows = 4
输出："PINALSIGYAHRPI"
解释：
P     I     N
A   L S   I G
Y A   H R
P     I
→ 逐行读取：PINALSIGYAHRPI
```

```
输入：s = "A", numRows = 1
输出："A"
解释：只有一行，直接返回原字符串。
```

---

## 核心思想

### 先理解 Z 字形是什么

把字符串想象成一个弹球，从第一行开始向下走，到底后弹回顶部，再向下……如此反复：

```
numRows = 4 时：

行0: P           I           N
行1: A         L S         I G
行2: Y       A   H       R
行3: P           I

弹球轨迹：0→1→2→3→2→1→0→1→2→3→2→1→0→1
向下(down) ↓↓↓↑↑↑↓↓↓↑↑↑↓  到底反弹，到顶再下
```

**关键规律：** 行号的变化是 `0→1→2→3→2→1→0→1→2→3→...`，即向下到底后折返向上，到顶后再折返向下。

### 两个重要特例

1. **numRows = 1**：只有一行，Z 字形就是原字符串本身，直接返回。
2. **numRows >= s.length**：行数比字符还多，字符排不满一列，每行最多一个字符，结果就是原字符串。

理解了这个"弹球模型"，三种解法都从这里出发。

---

## 思路分析

### 解法一：暴力模拟（排序法）

最直观的想法：**按 Z 字形的走法把字符一个个填到对应行，然后逐行拼接。**

```
维护一个 rows 数组（numRows 个字符串）
用一个变量 currentRow 记录当前行号
用一个变量 direction 记录方向（+1 向下，-1 向上）

遍历每个字符：
  1. 把当前字符追加到 rows[currentRow]
  2. 如果到达顶部（行0）或底部（行numRows-1），翻转方向
  3. currentRow += direction
```

最终拼接所有行即可。

### 解法二：逐行追踪法（最优）

换个角度思考：与其按字符的原始顺序逐个放置，不如**直接找每一行有哪些字符**。

```
总周期（cycleLen）= 2 * numRows - 2
（一个完整的"向下+向上"周期包含的字符数）

对于第 i 行（0-indexed），该行的字符位置为：
  - 第一个字符：i
  - 每个周期的两个字符：i + k*cycleLen 和 (k+1)*cycleLen - i
  - 但第一行（i=0）和最后一行（i=numRows-1）每个周期只有一个字符
```

不需要额外数组，直接在原字符串上按索引取字符。

### 解法三：数学公式法

和解法二思路相同，但用更严格的数学公式描述：

```
对于第 i 行，字符索引的通用公式：
  第一个索引：j = i
  第二个索引：j + cycleLen - 2*i （当 j + cycleLen - 2*i != j 且 < n 时）

每个周期步进 cycleLen 个位置。
```

本质上和解法二一样，只是用公式代替了"观察规律"。

---

## 代码实现

### JavaScript 版本

#### 方法一：暴力模拟（排序法）

```javascript
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
    // 特殊情况：只有一行或行数 >= 字符串长度
    if (numRows === 1 || numRows >= s.length) return s;

    // 创建 numRows 个空字符串，每行一个
    const rows = new Array(numRows).fill('');
    let currentRow = 0;
    let direction = 1;  // 1 = 向下, -1 = 向上

    for (let i = 0; i < s.length; i++) {
        rows[currentRow] += s[i];

        // 到达顶部或底部时翻转方向
        if (currentRow === 0) direction = 1;
        if (currentRow === numRows - 1) direction = -1;

        currentRow += direction;
    }

    return rows.join('');
};
```

#### 方法二：逐行追踪法（最优解）

```javascript
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
    if (numRows === 1 || numRows >= s.length) return s;

    const n = s.length;
    const cycleLen = 2 * numRows - 2;  // 一个完整周期的长度
    let result = '';

    for (let i = 0; i < numRows; i++) {
        for (let j = i; j < n; j += cycleLen) {
            // 每个周期的第一个字符（向下阶段的字符）
            result += s[j];

            // 每个周期的第二个字符（向上阶段的字符）
            // 条件：不是第一行、不是最后一行、索引不越界
            const secondJ = j + cycleLen - 2 * i;
            if (i !== 0 && i !== numRows - 1 && secondJ < n) {
                result += s[secondJ];
            }
        }
    }

    return result;
};
```

#### 方法三：数学公式法

```javascript
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
    if (numRows === 1 || numRows >= s.length) return s;

    const n = s.length;
    const cycleLen = 2 * numRows - 2;
    const result = [];

    for (let row = 0; row < numRows; row++) {
        let j = row;
        while (j < n) {
            result.push(s[j]);
            j += cycleLen;

            // 中间行：每个周期还有一个"斜线上的字符"
            if (row > 0 && row < numRows - 1) {
                const diagIdx = j - 2 * row;
                if (diagIdx < n) {
                    result.push(s[diagIdx]);
                }
            }
        }
    }

    return result.join('');
};
```

### Python 版本

#### 方法一：暴力模拟（排序法）

```python
def convert(s: str, numRows: int) -> str:
    """方法一：暴力模拟（排序法）"""
    if numRows == 1 or numRows >= len(s):
        return s

    rows = [''] * numRows
    current_row = 0
    direction = 1  # 1 = 向下, -1 = 向上

    for ch in s:
        rows[current_row] += ch

        # 到达顶部或底部时翻转方向
        if current_row == 0:
            direction = 1
        elif current_row == numRows - 1:
            direction = -1

        current_row += direction

    return ''.join(rows)
```

#### 方法二：逐行追踪法（最优解）

```python
def convert(s: str, numRows: int) -> str:
    """方法二：逐行追踪法（最优解）"""
    if numRows == 1 or numRows >= len(s):
        return s

    n = len(s)
    cycle_len = 2 * numRows - 2  # 一个完整周期的长度
    result = []

    for i in range(numRows):
        j = i
        while j < n:
            result.append(s[j])  # 每个周期的第一个字符

            # 每个周期的第二个字符（向上阶段）
            # 条件：不是第一行、不是最后一行、索引不越界
            second_j = j + cycle_len - 2 * i
            if i != 0 and i != numRows - 1 and second_j < n:
                result.append(s[second_j])

            j += cycle_len

    return ''.join(result)
```

#### 方法三：数学公式法

```python
def convert(s: str, numRows: int) -> str:
    """方法三：数学公式法"""
    if numRows == 1 or numRows >= len(s):
        return s

    n = len(s)
    cycle_len = 2 * numRows - 2
    result = []

    for row in range(numRows):
        j = row
        while j < n:
            result.append(s[j])
            j += cycle_len

            # 中间行：每个周期还有一个"斜线上的字符"
            if 0 < row < numRows - 1:
                diag_idx = j - 2 * row
                if diag_idx < n:
                    result.append(s[diag_idx])

    return ''.join(result)
```

---

### 逐步推演

以 `s = "PAYPALISHIRING"`, `numRows = 3` 为例（答案：`"PAHNAPLSIIGYIR"`）。

#### 方法一（暴力模拟）推演

```
s = "PAYPALISHIRING", numRows = 3
cycleLen = 2 * 3 - 2 = 4

rows = ["", "", ""]
currentRow = 0, direction = 1

i=0: 'P' → rows[0]="P",  currentRow=0+1=1
i=1: 'A' → rows[1]="A",  currentRow=1+1=2
i=2: 'Y' → rows[2]="Y",  到底→direction=-1, currentRow=2-1=1
i=3: 'P' → rows[1]="AP", currentRow=1-1=0
i=4: 'A' → rows[0]="PA", 到顶→direction=1, currentRow=0+1=1
i=5: 'L' → rows[1]="APL", currentRow=1+1=2
i=6: 'I' → rows[2]="YI", 到底→direction=-1, currentRow=2-1=1
i=7: 'S' → rows[1]="APLS", currentRow=1-1=0
i=8: 'H' → rows[0]="PAH", 到顶→direction=1, currentRow=0+1=1
i=9: 'I' → rows[1]="APLSI", currentRow=1+1=2
i=10:'R' → rows[2]="YIR", 到底→direction=-1, currentRow=2-1=1
i=11:'I' → rows[1]="APLSII", currentRow=1-1=0
i=12:'N' → rows[0]="PAHN", 到顶→direction=1, currentRow=0+1=1
i=13:'G' → rows[1]="APLSIIG", currentRow=1+1=2

rows[0] = "PAHN"
rows[1] = "APLSIIG"
rows[2] = "YIR"

拼接："PAHN" + "APLSIIG" + "YIR" = "PAHNAPLSIIGYIR" ✅
```

#### 方法二（逐行追踪法）推演

```
s = "PAYPALISHIRING", n=14, numRows=3
cycleLen = 2*3-2 = 4

逐行找字符：

第0行 (i=0): j=0,4,8,12
  j=0:  s[0]='P'  → result: "P"
  j=4:  s[4]='A'  → result: "PA"
  j=8:  s[8]='H'  → result: "PAH"
  j=12: s[12]='N' → result: "PAHN"
  (i=0 是第一行，跳过第二个字符)

第1行 (i=1): j=1,5,9,13
  j=1:  s[1]='A'  → result: "PAHA"
        secondJ = 1+4-2=3, s[3]='P' → result: "PAHAP"
  j=5:  s[5]='L'  → result: "PAHAPL"
        secondJ = 5+4-2=7, s[7]='S' → result: "PAHAPLS"
  j=9:  s[9]='I'  → result: "PAHAPLSI"
        secondJ = 9+4-2=11, s[11]='I' → result: "PAHAPLSII"
  j=13: s[13]='G' → result: "PAHAPLSIIG"
        secondJ = 13+4-2=15 ≥ 14 → 跳过

第2行 (i=2): j=2,6,10
  j=2:  s[2]='Y'  → result: "PAHAPLSIIGY"
  j=6:  s[6]='I'  → result: "PAHAPLSIIGYI"
  j=10: s[10]='R' → result: "PAHAPLSIIGYIR"
  (i=2 是最后一行，跳过第二个字符)

最终结果："PAHNAPLSIIGYIR" ✅
```

#### numRows = 4 的推演

```
s = "PAYPALISHIRING", numRows = 4
cycleLen = 2*4-2 = 6

排列形状：
行0: P           I           N        → j=0, 6, 12
行1: A         L S         I G        → j=1,5  7,11  13
行2: Y       A   H       R            → j=2,4  8,10
行3: P           I                    → j=3, 9

第0行 (i=0): j=0,6,12
  s[0]='P', s[6]='I', s[12]='N'  → "PIN"

第1行 (i=1): j=1,7,13
  j=1:  s[1]='A', secondJ=1+6-2=5, s[5]='L'  → "AL"
  j=7:  s[7]='S', secondJ=7+6-2=11, s[11]='I' → "ALSI"
  j=13: s[13]='G', secondJ=13+6-2=17≥14 → "ALSIG"

第2行 (i=2): j=2,8
  j=2:  s[2]='Y', secondJ=2+6-4=4, s[4]='A'  → "YA"
  j=8:  s[8]='H', secondJ=8+6-4=10, s[10]='R' → "YAHR"

第3行 (i=3): j=3,9
  s[3]='P', s[9]='I'  → "PI"

拼接："PIN" + "ALSIG" + "YAHR" + "PI" = "PINALSIGYAHRPI" ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **暴力模拟（排序法）** | O(n) | O(n) | 最直观，好理解 | 需要额外的 rows 数组 |
| **逐行追踪法** | O(n) | O(1) | 最优解，面试首选 | 需要理解周期规律 |
| **数学公式法** | O(n) | O(1) | 与方法二本质相同 | 公式稍抽象 |

> 注：三种方法的时间复杂度都是 O(n)，因为每个字符最多被访问一次（或常数次）。空间复杂度的 O(1) 指的是除输出外不需要额外空间。

---

## 优化方向对比

### 1. 为什么逐行追踪法是最优的？

```
暴力模拟法：  遍历字符串一次 → 填入 rows → 拼接
逐行追踪法：  遍历字符串一次 → 直接按索引取字符拼接

两种方法时间复杂度相同，但逐行追踪法：
  - 不需要 rows 数组（省 O(n) 空间）
  - 直接构建结果字符串（无中间数据结构）
  - 在字符串很长时，减少内存分配
```

### 2. cycleLen 的几何意义

```
cycleLen = 2 * numRows - 2

为什么？一个完整的 Z 字形周期：
  - 向下走：numRows 步（从行0到行numRows-1）
  - 向上走：numRows - 2 步（从行numRows-2到行1，不重复首尾）
  - 总计：numRows + numRows - 2 = 2 * numRows - 2

例如 numRows = 4:
  向下：0→1→2→3（4步）
  向上：2→1     （2步，不重复行3和行0）
  总计：6步 = 2*4-2 ✅
```

### 3. 第一行和最后一行的特殊性

```
中间行（i=1 到 i=numRows-2）：每个周期有 2 个字符
  - 一个来自"向下"阶段
  - 一个来自"向上"阶段（斜线上的字符）

第一行（i=0）和最后一行（i=numRows-1）：每个周期只有 1 个字符
  - 因为"向下"和"向上"在首尾行重合
```

这就是为什么代码里有 `if i !== 0 && i !== numRows - 1` 的判断。

---

## 举一反三

Z 字形变换的核心模式是"**用周期和索引公式直接定位字符**"，这个思路在很多字符串题中都有应用：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 6. Z 字形变换** | 周期 = 2*numRows-2，逐行按索引取字符 |
| **LeetCode 8. 字符串转换整数 (atoi)** | 字符串模拟，逐字符处理状态机 |
| **LeetCode 14. 最长公共前缀** | 字符串逐列比较 |
| **LeetCode 28. 找出字符串中第一个匹配项的下标** | 字符串模式匹配 |
| **LeetCode 68. 文本左右对齐** | 字符串按行分组，模拟排版 |

它们的共同模式：**理解字符串的"排列结构"后，用数学公式或模拟规则高效地重建结果。**

---

## 总结

Z 字形变换这道题的精髓在于**从"画出来再读"的模拟思维跃迁到"用公式直接定位"的数学思维**：

1. **核心洞察**：Z 字形的行号变化遵循"向下→反弹→向上→反弹"的周期规律，周期长度 = 2*numRows-2
2. **三种解法**：
   - 暴力模拟（排序法）：逐字符填入行数组，O(n) 时间 + O(n) 空间
   - **逐行追踪法（推荐）**：用周期公式直接按行取字符，O(n) 时间 + O(1) 空间
   - 数学公式法：与方法二本质相同，用更严格的公式描述
3. **边界处理**：numRows=1 直接返回原字符串；首行和末行每周期只有一个字符

面试中建议这样展示：先用暴力模拟法把"Z 字形的走法"讲清楚，让面试官看到你理解了题意；然后自然地优化到逐行追踪法，说明"与其按顺序放，不如直接找每行有哪些字符"。如果能解释清楚 cycleLen 的几何意义和首末行的特殊性，就是完美的面试表现。

最后记住一句话：**当暴力模拟太慢时，试着反过来——不是"放进去再读"，而是"直接找到该读什么"。**
