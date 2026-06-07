---
title: LeetCode 49. 字母异位词分组
tags: [哈希表, 字符串, 排序, 计数]
difficulty: Medium
category: 哈希表
date: 2026-06-07
---

# LeetCode 49. 字母异位词分组 —— 三种思路彻底讲透

## 前言

字母异位词分组（Group Anagrams）是 LeetCode 上一道经典的哈希表应用题。它的核心思想非常直觉：**把"长得一样"的字符串归为一组**。

什么是字母异位词？两个字符串如果包含完全相同的字符、只是排列顺序不同，它们就是字母异位词。比如 `"eat"`、`"tea"`、`"ate"` 互为字母异位词。

这道题的关键在于：**如何定义"长得一样"？** 不同的定义方式就衍生出不同的解法。本文将用三种思路讲透这道题：**排序法** → **计数法** → **质数乘积法**，每种方法都附带 JavaScript 和 Python 双语言实现。

---

## 问题描述

### LeetCode 49. Group Anagrams（字母异位词分组）

> 给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。
>
> **字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

**示例：**

```
输入：strs = ["eat","tea","tan","ate","nat","bat"]
输出：[["bat"],["nat","tan"],["ate","eat","tea"]]
解释：
  "bat" 没有异位词同伴，独占一组
  "nat" 和 "tan" 互为异位词
  "ate"、"eat"、"tea" 互为异位词

输入：strs = [""]
输出：[[""]]

输入：strs = ["a"]
输出：[["a"]]
```

**约束条件：**
- `1 <= strs.length <= 10^4`
- `0 <= strs[i].length <= 100`
- `strs[i]` 仅包含小写英文字母

---

## 核心思想

### 什么是字母异位词？

```
字母异位词的本质：两个字符串的字符频次完全相同

"eat" → {a:1, e:1, t:1}
"tea" → {a:1, e:1, t:1}
"ate" → {a:1, e:1, t:1}
→ 三者字符频次相同，互为异位词 ✅

"eat" → {a:1, e:1, t:1}
"bat" → {a:1, b:1, t:1}
→ 字符频次不同，不是异位词 ❌
```

### 如何判断两个字符串是否是异位词？

有三种经典方式：

```
方式一：排序
  "eat" → sort → "aet"
  "tea" → sort → "aet"
  排序后相同 → 是异位词 ✅

方式二：字符计数
  "eat" → [1,0,0,...,1,...,1,...] (26维计数向量)
  "tea" → [1,0,0,...,1,...,1,...]
  计数向量相同 → 是异位词 ✅

方式三：质数乘积
  给每个字母分配一个质数：a=2, b=3, c=5, ...
  "eat" → 7*2*67 = 938  (e=7, a=2, t=67)
  "tea" → 67*7*2 = 938
  乘积相同 → 是异位词 ✅（算术基本定理保证唯一性）
```

### 问题转化

```
原始问题：把异位词分组
    ↓ 转化为
核心问题：为每个字符串找一个"签名"（key），使得异位词拥有相同的签名
    ↓
用哈希表：签名 → [拥有该签名的所有字符串]
```

这就是哈希表的典型应用：**用一个精心设计的 key 把等价类映射到同一个桶里**。

---

## 思路分析

### 解法一：排序法（最直观）

既然异位词排序后都相同，那就直接排序作为 key。

```
对于每个字符串 s：
  key = sort(s)    // 排序后的字符串作为签名
  把 s 放入 hashMap[key] 对应的列表中

最后 hashMap 中的每个 value 就是一组异位词
```

```
示例：strs = ["eat","tea","tan","ate","nat","bat"]

"eat" → sort → "aet" → hashMap["aet"] = ["eat"]
"tea" → sort → "aet" → hashMap["aet"] = ["eat", "tea"]
"tan" → sort → "ant" → hashMap["ant"] = ["tan"]
"ate" → sort → "aet" → hashMap["aet"] = ["eat", "tea", "ate"]
"nat" → sort → "ant" → hashMap["ant"] = ["tan", "nat"]
"bat" → sort → "abt" → hashMap["abt"] = ["bat"]

结果：[["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

### 解法二：计数法（最优）

排序有 O(k log k) 的开销。既然我们只关心字符频次，为什么不直接统计频次作为 key？

```
对于每个字符串 s：
  统计 26 个字母的出现次数 → count = [0,0,...,0]
  key = count 转为字符串（如 "1,0,0,0,1,0,...,1,..."）
  把 s 放入 hashMap[key] 对应的列表中
```

```
示例：strs = ["eat","tea","tan","ate","nat","bat"]

"eat" → count: a=1,e=1,t=1 → key="1,0,0,0,1,0,...,1,..."
"tea" → count: a=1,e=1,t=1 → key 相同，归入同一组
"tan" → count: a=1,n=1,t=1 → key 不同，新组
"bat" → count: a=1,b=1,t=1 → key 不同，新组

时间更优：O(k) 计数 vs O(k log k) 排序
```

### 解法三：质数乘积法（巧妙）

给 26 个字母各分配一个质数，字符串的 key = 所有字母对应质数的乘积。

```
质数分配：a=2, b=3, c=5, d=7, e=11, f=13, ... z=101

"eat" → key = 11 * 2 * 67 = 1474  (e=11, a=2, t=67)
"tea" → key = 67 * 11 * 2 = 1474  ← 相同！
"ate" → key = 2 * 67 * 11 = 1474  ← 相同！
```

为什么有效？**算术基本定理**：每个正整数的质因数分解是唯一的。如果两个字符串的字符频次相同，它们的质数乘积一定相同；反之亦然。

> ⚠️ **注意**：这种方法虽然巧妙，但当字符串很长时，乘积可能溢出（JavaScript 的安全整数范围是 2^53）。实际面试中推荐方法二，但质数法可以展示你的数学思维。

---

## 代码实现

### 方法一：排序法

#### JavaScript 版本

```javascript
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
var groupAnagrams = function(strs) {
    const map = new Map();

    for (const s of strs) {
        // 排序后的字符串作为 key
        const key = s.split('').sort().join('');
        if (!map.has(key)) {
            map.set(key, []);
        }
        map.get(key).push(s);
    }

    return [...map.values()];
};
```

#### Python 版本

```python
from collections import defaultdict

def groupAnagrams(strs: list[str]) -> list[list[str]]:
    """方法一：排序法"""
    map_dict = defaultdict(list)

    for s in strs:
        # 排序后的字符串作为 key
        key = ''.join(sorted(s))
        map_dict[key].append(s)

    return list(map_dict.values())
```

---

### 方法二：计数法（最优）

#### JavaScript 版本

```javascript
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
var groupAnagrams = function(strs) {
    const map = new Map();

    for (const s of strs) {
        // 统计 26 个字母的频次
        const count = new Array(26).fill(0);
        for (const ch of s) {
            count[ch.charCodeAt(0) - 97]++;
        }
        // 频次数组转为字符串 key
        const key = count.join(',');
        if (!map.has(key)) {
            map.set(key, []);
        }
        map.get(key).push(s);
    }

    return [...map.values()];
};
```

#### Python 版本

```python
from collections import defaultdict

def groupAnagrams(strs: list[str]) -> list[list[str]]:
    """方法二：计数法（最优）"""
    map_dict = defaultdict(list)

    for s in strs:
        # 统计 26 个字母的频次
        count = [0] * 26
        for ch in s:
            count[ord(ch) - ord('a')] += 1
        # 频次元组作为 key（tuple 可哈希）
        key = tuple(count)
        map_dict[key].append(s)

    return list(map_dict.values())
```

> 💡 Python 中用 `tuple(count)` 比 `str(count)` 更高效，因为 tuple 可以直接作为字典 key，不需要额外的字符串转换。

---

### 方法三：质数乘积法

#### JavaScript 版本

```javascript
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
var groupAnagrams = function(strs) {
    // 26 个质数对应 a-z
    const primes = [2,3,5,7,11,13,17,19,23,29,31,37,41,
                    43,47,53,59,61,67,71,73,79,83,89,97,101];

    const map = new Map();

    for (const s of strs) {
        let key = 1;
        for (const ch of s) {
            key *= primes[ch.charCodeAt(0) - 97];
        }
        if (!map.has(key)) {
            map.set(key, []);
        }
        map.get(key).push(s);
    }

    return [...map.values()];
};
```

> ⚠️ JavaScript 中当字符串较长时，乘积可能超出 `Number.MAX_SAFE_INTEGER`（2^53），导致精度丢失。此方法更适合用于面试展示思路，不推荐在生产环境使用。

#### Python 版本

```python
from collections import defaultdict

def groupAnagrams(strs: list[str]) -> list[list[str]]:
    """方法三：质数乘积法"""
    # 26 个质数对应 a-z
    primes = [2,3,5,7,11,13,17,19,23,29,31,37,41,
              43,47,53,59,61,67,71,73,79,83,89,97,101]

    map_dict = defaultdict(list)

    for s in strs:
        key = 1
        for ch in s:
            key *= primes[ord(ch) - ord('a')]
        map_dict[key].append(s)

    return list(map_dict.values())
```

> 💡 Python 的整数没有溢出问题（任意精度），所以质数法在 Python 中完全可行。但面试中仍然推荐计数法，因为更通用、更直观。

---

## 逐步推演

### 示例：`strs = ["eat","tea","tan","ate","nat","bat"]`

#### 方法一（排序法）推演

```
初始化：map = {}

处理 "eat":
  key = sort("eat") = "aet"
  map 中无 "aet" → 创建 map["aet"] = []
  map["aet"].push("eat") → map = {"aet": ["eat"]}

处理 "tea":
  key = sort("tea") = "aet"
  map 中有 "aet" ✅
  map["aet"].push("tea") → map = {"aet": ["eat","tea"]}

处理 "tan":
  key = sort("tan") = "ant"
  map 中无 "ant" → 创建
  map["ant"].push("tan") → map = {"aet":["eat","tea"], "ant":["tan"]}

处理 "ate":
  key = sort("ate") = "aet"
  map 中有 "aet" ✅
  map["aet"].push("ate") → map = {"aet":["eat","tea","ate"], "ant":["tan"]}

处理 "nat":
  key = sort("nat") = "ant"
  map 中有 "ant" ✅
  map["ant"].push("nat") → map = {"aet":["eat","tea","ate"], "ant":["tan","nat"]}

处理 "bat":
  key = sort("bat") = "abt"
  map 中无 "abt" → 创建
  map["abt"].push("bat") → map = {"aet":["eat","tea","ate"], "ant":["tan","nat"], "abt":["bat"]}

返回所有 value → [["eat","tea","ate"], ["tan","nat"], ["bat"]] ✅
```

#### 方法二（计数法）推演

```
初始化：map = {}

处理 "eat":
  count = [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0]
          a=1              e=1                              t=1
  key = "1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0"
  map 中无此 key → 创建 map[key] = ["eat"]

处理 "tea":
  count = [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0]
          t=1              e=1                              a=1
  → 统计结果与 "eat" 完全相同！key 相同 ✅
  map[key].push("tea") → map[key] = ["eat","tea"]

处理 "tan":
  count = [1,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,0]
          a=1                            n=1              t=1
  key 不同 → 新组 map[key2] = ["tan"]

处理 "ate":
  count = [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0]
  key 与 "eat" 相同 ✅ → map[key] = ["eat","tea","ate"]

处理 "nat":
  count = [1,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,0]
  key 与 "tan" 相同 ✅ → map[key2] = ["tan","nat"]

处理 "bat":
  count = [1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0]
          a=1  b=1                           t=1
  key 不同 → 新组 map[key3] = ["bat"]

返回所有 value → [["eat","tea","ate"], ["tan","nat"], ["bat"]] ✅
```

#### 方法三（质数乘积法）推演

```
质数表：a=2, b=3, e=11, t=67, n=41

处理 "eat":
  key = 11 * 2 * 67 = 1474
  map[1474] = ["eat"]

处理 "tea":
  key = 67 * 11 * 2 = 1474  ← 乘法交换律，结果相同！
  map[1474] = ["eat","tea"]

处理 "tan":
  key = 67 * 2 * 41 = 5494
  map[5494] = ["tan"]

处理 "ate":
  key = 2 * 67 * 11 = 1474  ← 同上
  map[1474] = ["eat","tea","ate"]

处理 "nat":
  key = 41 * 2 * 67 = 5494  ← 与 "tan" 相同
  map[5494] = ["tan","nat"]

处理 "bat":
  key = 3 * 2 * 67 = 402
  map[402] = ["bat"]

返回所有 value → [["eat","tea","ate"], ["tan","nat"], ["bat"]] ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **排序法** | O(n · k log k) | O(n · k) | 最直观，代码简洁 | 排序开销 |
| **计数法** | O(n · k) | O(n · k) | 最优时间，线性扫描 | key 是 26 维向量 |
| **质数乘积法** | O(n · k) | O(n · k) | 思路巧妙，key 是单个整数 | 可能溢出（JS） |

> 注：n = 字符串数组长度，k = 单个字符串的最大长度。
>
> - 排序法的瓶颈在于每个字符串都要排序，O(k log k)
> - 计数法只需遍历每个字符串一次统计频次，O(k)
> - 质数乘积法也是 O(k)，但存在大数溢出风险

---

## 优化方向对比

### 1. 排序法 vs 计数法：什么时候选哪个？

```
排序法：
  - 代码最简洁（Python 一行搞定）
  - 适合 k 较小的场景（字符串短，排序开销可忽略）
  - 面试中最容易想到、最容易写对

计数法：
  - 时间复杂度更优 O(k) vs O(k log k)
  - 适合 k 较大的场景（长字符串）
  - 面试中的"最优解"，展现优化能力

实际选择：
  - 面试时先写排序法（快速且正确），再优化到计数法
  - 如果 k ≤ 10，两者性能差距微乎其微
  - 如果 k 很大（如 10^5），计数法优势明显
```

### 2. 质数乘积法的隐患

```
隐患一：溢出
  26 个质数的乘积（即使每个字母只出现一次）= 2*3*5*7*...*101 ≈ 2.3 × 10^38
  JavaScript Number.MAX_SAFE_INTEGER = 9 × 10^15
  → 字符串超过约 15 个不同字母就会溢出！

  Python 没有这个问题（任意精度整数）

隐患二：碰撞（理论上）
  如果不用质数而用普通整数，不同字母组合可能产生相同乘积
  用质数则不会（算术基本定理保证）

结论：质数法适合展示数学思维，但不是生产级方案
```

### 3. 三种方法的本质联系

```
排序法：把异位词"标准化"为同一个字符串
计数法：把异位词"标准化"为同一个频次向量
质数法：把异位词"编码"为同一个整数

三者殊途同归：都是在构造一个"签名函数" f(s)，
使得 f(s1) = f(s2) 当且仅当 s1 和 s2 是异位词。

哈希表负责把相同签名的字符串归到一起。
```

---

## 举一反三

字母异位词的核心是**用哈希表对等价类分组**，以下题目都用到了类似思路：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 49. 字母异位词分组** | 排序/计数作为 key 分组 |
| **LeetCode 242. 有效的字母异位词** | 判断两个字符串是否是异位词（计数比较） |
| **LeetCode 438. 找到字符串中所有字母异位词** | 滑动窗口 + 频次比较 |
| **LeetCode 205. 同构字符串** | 双射映射检查，结构模式匹配 |
| **LeetCode 290. 单词规律** | 字符→单词的双射映射 |
| **LeetCode 383. 赎金信** | 字符频次统计的简化版 |

它们的共同模式：**构造一个"签名"来描述字符串的本质特征，然后用哈希表根据签名进行匹配或分组。**

---

## 总结

字母异位词分组是一道**哈希表 + 字符串处理**的经典题，核心在于设计一个好的签名函数：

1. **核心洞察**：异位词 = 字符频次相同的字符串。找到一个能唯一标识字符频次的 key，问题就变成了哈希表分组。

2. **三种解法**：
   - 排序法：最直观，面试首选开场
   - **计数法（推荐）**：时间最优 O(n·k)，面试的标准答案
   - 质数乘积法：数学巧妙，适合展示思维广度

3. **面试建议**：
   - 先说排序法，30 秒讲清思路，2 分钟写完代码
   - 再分析排序的瓶颈（O(k log k)），优化到计数法（O(k)）
   - 如果时间允许，提一下质数法展示数学功底
   - 主动分析三种方法的 trade-off，展现工程思维

最后记住一句话：**异位词分组的本质是"设计签名函数 + 哈希表分组"，这是一类问题的通用模式。**
