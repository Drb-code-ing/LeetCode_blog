---
title: LeetCode 205. 同构字符串 & 290. 单词规律
tags: [哈希表, 字符串, 双射映射]
difficulty: Easy
category: 哈希表
date: 2026-06-06
---

# LeetCode 205. 同构字符串 & 290. 单词规律 —— 双射映射一题通

## 前言

同构字符串（Isomorphic Strings）和单词规律（Word Pattern）是 LeetCode 上一对"孪生题"。它们的核心思想完全一致：**判断两个序列之间是否存在合法的一一映射（双射）**。

区别仅在于映射的对象不同：
- **205 题**：字符 → 字符
- **290 题**：字符 → 单词

理解了一道，另一道自然就会了。本文将用三种思路彻底讲透双射映射：**双向哈希表** → **标准化编码** → **首次出现索引**，每种方法都附带 JavaScript 和 Python 双语言实现。

---

## 问题描述

### LeetCode 205. Isomorphic Strings（同构字符串）

> 给定两个字符串 `s` 和 `t`，判断它们是否是同构的。
>
> 如果 `s` 中的字符可以按某种映射关系替换得到 `t`，那么这两个字符串是同构的。
>
> 每个字符必须映射到另一个字符，同一字符必须映射到同一个字符，不同字符不能映射到同一个字符。

**示例：**

```
输入：s = "egg", t = "add"
输出：true
解释：e → a, g → d，映射合法。

输入：s = "foo", t = "bar"
输出：false
解释：f → b, o → a, 但第二个 o 应该也映射到 a，而 t 中对应的是 r，矛盾。

输入：s = "paper", t = "title"
输出：true
解释：p → t, a → i, e → l, r → e，映射合法。
```

### LeetCode 290. Word Pattern（单词规律）

> 给定一种规律 `pattern` 和一个字符串 `s`，判断 `s` 是否遵循这种规律。
>
> 这里的遵循指完全匹配，即 pattern 中的每个字符和 `s` 中每个单词之间存在双向的映射关系。

**示例：**

```
输入：pattern = "abba", s = "dog cat cat dog"
输出：true
解释：a → dog, b → cat，映射合法。

输入：pattern = "abba", s = "dog cat cat fish"
输出：false
解释：a → dog，但第二个 a 对应 fish，矛盾。

输入：pattern = "aaaa", s = "dog cat cat dog"
输出：false
解释：所有 a 都应该映射到同一个词，但对应了 dog 和 cat，矛盾。
```

---

## 核心思想

### 什么是双射（Bijection）？

双射要求映射**双向唯一**：

```
合法映射（双射）：
  s = "egg"    t = "add"
  e → a        a → e
  g → d        d → g
  ✅ 每个字符唯一对应，没有冲突

非法映射（多对一）：
  s = "foo"    t = "bar"
  f → b        b → f
  o → a        a → o
  o → r ❌     第二个 o 映射到了 r，与之前的 a 矛盾！
```

**关键：必须同时检查两个方向！**
- 正向：s 的字符不能映射到多个不同的 t 字符
- 反向：t 的字符不能被多个不同的 s 字符映射到

只有两个方向都合法，才是真正的双射。

### 205 题 vs 290 题的本质区别

```
205 题（同构字符串）：
  - 映射粒度：字符 → 字符
  - s 和 t 长度必须相等
  - 逐字符比较

290 题（单词规律）：
  - 映射粒度：字符 → 单词
  - pattern 长度必须等于 s.split(' ').length
  - 先分割再逐项比较
```

核心算法完全一样，只是 290 题多了一步"按空格分割字符串"。

---

## 思路分析

### 解法一：双向哈希表

最直观的方法：**用两个哈希表分别记录正向和反向映射**。

```
正向映射表 mapST：记录 s 中每个字符映射到 t 的什么字符
反向映射表 mapTS：记录 t 中每个字符被 s 的什么字符映射

遍历每一对字符 (s[i], t[i])：
  1. 如果 s[i] 已经在 mapST 中，检查映射值是否等于 t[i]
     - 不等于 → 冲突，返回 false
  2. 如果 t[i] 已经在 mapTS 中，检查映射值是否等于 s[i]
     - 不等于 → 冲突，返回 false
  3. 都没有冲突，记录双向映射

遍历结束，没有冲突 → 返回 true
```

### 解法二：标准化编码

换个角度：如果两个字符串是同构的，那么它们的"结构"应该完全相同。

```
给每个字符赋予一个"首次出现的序号"：
  "egg"  → 首次出现索引编码：[0, 1, 1]
  "add"  → 首次出现索引编码：[0, 1, 1]
  编码相同 → 同构 ✅

  "foo"  → [0, 1, 1]
  "bar"  → [0, 1, 2]
  编码不同 → 不同构 ❌
```

把每个字符串都转换成"标准化编码"，然后比较两个编码是否相同。

### 解法三：首次出现索引法（最优）

解法二的精简版：**用字符首次出现的位置作为编码值**。

```
对于字符串 s 中的每个字符 c：
  编码值 = c 第一次出现的索引位置

比较 s 和 t 的编码序列是否完全相同。
```

这个方法不需要构建两个哈希表，只需要一个映射表记录首次出现位置。

---

## 代码实现

### LeetCode 205. 同构字符串

#### JavaScript 版本

##### 方法一：双向哈希表

```javascript
/**
 * @param {string} s
 * @param {string} t
 * @return {boolean}
 */
var isIsomorphic = function(s, t) {
    if (s.length !== t.length) return false;

    const mapST = new Map();  // s → t 的映射
    const mapTS = new Map();  // t → s 的映射

    for (let i = 0; i < s.length; i++) {
        const cs = s[i], ct = t[i];

        // 正向检查：s 的字符已经映射过，但映射目标不一致
        if (mapST.has(cs) && mapST.get(cs) !== ct) return false;
        // 反向检查：t 的字符已经映射过，但映射来源不一致
        if (mapTS.has(ct) && mapTS.get(ct) !== cs) return false;

        mapST.set(cs, ct);
        mapTS.set(ct, cs);
    }

    return true;
};
```

##### 方法二：标准化编码

```javascript
/**
 * @param {string} s
 * @param {string} t
 * @return {boolean}
 */
var isIsomorphic = function(s, t) {
    if (s.length !== t.length) return false;

    // 将字符串转为标准化编码
    function normalize(str) {
        const map = new Map();
        let id = 0;
        const codes = [];
        for (const ch of str) {
            if (!map.has(ch)) {
                map.set(ch, id++);
            }
            codes.push(map.get(ch));
        }
        return codes.join(',');
    }

    return normalize(s) === normalize(t);
};
```

##### 方法三：首次出现索引法（最优）

```javascript
/**
 * @param {string} s
 * @param {string} t
 * @return {boolean}
 */
var isIsomorphic = function(s, t) {
    if (s.length !== t.length) return false;

    // 记录每个字符首次出现的索引
    function firstOccurrence(str) {
        const map = new Map();
        const result = [];
        for (let i = 0; i < str.length; i++) {
            if (!map.has(str[i])) {
                map.set(str[i], i);
            }
            result.push(map.get(str[i]));
        }
        return result;
    }

    const codeS = firstOccurrence(s);
    const codeT = firstOccurrence(t);

    for (let i = 0; i < s.length; i++) {
        if (codeS[i] !== codeT[i]) return false;
    }
    return true;
};
```

#### Python 版本

##### 方法一：双向哈希表

```python
def isIsomorphic(s: str, t: str) -> bool:
    """方法一：双向哈希表"""
    if len(s) != len(t):
        return False

    map_st = {}  # s → t 的映射
    map_ts = {}  # t → s 的映射

    for cs, ct in zip(s, t):
        # 正向检查
        if cs in map_st and map_st[cs] != ct:
            return False
        # 反向检查
        if ct in map_ts and map_ts[ct] != cs:
            return False

        map_st[cs] = ct
        map_ts[ct] = cs

    return True
```

##### 方法二：标准化编码

```python
def isIsomorphic(s: str, t: str) -> bool:
    """方法二：标准化编码"""
    if len(s) != len(t):
        return False

    def normalize(string: str) -> str:
        mapping = {}
        counter = 0
        codes = []
        for ch in string:
            if ch not in mapping:
                mapping[ch] = counter
                counter += 1
            codes.append(str(mapping[ch]))
        return ','.join(codes)

    return normalize(s) == normalize(t)
```

##### 方法三：首次出现索引法（最优）

```python
def isIsomorphic(s: str, t: str) -> bool:
    """方法三：首次出现索引法（最优）"""
    if len(s) != len(t):
        return False

    def first_occurrence(string: str) -> list:
        mapping = {}
        result = []
        for i, ch in enumerate(string):
            if ch not in mapping:
                mapping[ch] = i
            result.append(mapping[ch])
        return result

    return first_occurrence(s) == first_occurrence(t)
```

---

### LeetCode 290. 单词规律

#### JavaScript 版本

##### 方法一：双向哈希表

```javascript
/**
 * @param {string} pattern
 * @param {string} s
 * @return {boolean}
 */
var wordPattern = function(pattern, s) {
    const words = s.split(' ');
    if (pattern.length !== words.length) return false;

    const mapPW = new Map();  // pattern 字符 → 单词
    const mapWP = new Map();  // 单词 → pattern 字符

    for (let i = 0; i < pattern.length; i++) {
        const ch = pattern[i], word = words[i];

        if (mapPW.has(ch) && mapPW.get(ch) !== word) return false;
        if (mapWP.has(word) && mapWP.get(word) !== ch) return false;

        mapPW.set(ch, word);
        mapWP.set(word, ch);
    }

    return true;
};
```

##### 方法二：标准化编码

```javascript
/**
 * @param {string} pattern
 * @param {string} s
 * @return {boolean}
 */
var wordPattern = function(pattern, s) {
    const words = s.split(' ');
    if (pattern.length !== words.length) return false;

    function normalize(arr) {
        const map = new Map();
        let id = 0;
        return arr.map(item => {
            if (!map.has(item)) map.set(item, id++);
            return map.get(item);
        }).join(',');
    }

    return normalize([...pattern]) === normalize(words);
};
```

##### 方法三：首次出现索引法（最优）

```javascript
/**
 * @param {string} pattern
 * @param {string} s
 * @return {boolean}
 */
var wordPattern = function(pattern, s) {
    const words = s.split(' ');
    if (pattern.length !== words.length) return false;

    function firstOccurrence(arr) {
        const map = new Map();
        return arr.map((item, i) => {
            if (!map.has(item)) map.set(item, i);
            return map.get(item);
        });
    }

    const codeP = firstOccurrence([...pattern]);
    const codeW = firstOccurrence(words);

    for (let i = 0; i < pattern.length; i++) {
        if (codeP[i] !== codeW[i]) return false;
    }
    return true;
};
```

#### Python 版本

##### 方法一：双向哈希表

```python
def wordPattern(pattern: str, s: str) -> bool:
    """方法一：双向哈希表"""
    words = s.split()
    if len(pattern) != len(words):
        return False

    map_pw = {}  # pattern 字符 → 单词
    map_wp = {}  # 单词 → pattern 字符

    for ch, word in zip(pattern, words):
        if ch in map_pw and map_pw[ch] != word:
            return False
        if word in map_wp and map_wp[word] != ch:
            return False

        map_pw[ch] = word
        map_wp[word] = ch

    return True
```

##### 方法二：标准化编码

```python
def wordPattern(pattern: str, s: str) -> bool:
    """方法二：标准化编码"""
    words = s.split()
    if len(pattern) != len(words):
        return False

    def normalize(items):
        mapping = {}
        counter = 0
        codes = []
        for item in items:
            if item not in mapping:
                mapping[item] = counter
                counter += 1
            codes.append(mapping[item])
        return codes

    return normalize(pattern) == normalize(words)
```

##### 方法三：首次出现索引法（最优）

```python
def wordPattern(pattern: str, s: str) -> bool:
    """方法三：首次出现索引法（最优）"""
    words = s.split()
    if len(pattern) != len(words):
        return False

    def first_occurrence(items):
        mapping = {}
        result = []
        for i, item in enumerate(items):
            if item not in mapping:
                mapping[item] = i
            result.append(mapping[item])
        return result

    return first_occurrence(pattern) == first_occurrence(words)
```

---

## 逐步推演

### 205 题推演

#### 示例 1：`s = "egg"`, `t = "add"`

**方法一（双向哈希表）推演：**

```
s = "egg", t = "add"
mapST = {}, mapTS = {}

i=0: cs='e', ct='a'
  mapST 中无 'e' → 记录 mapST['e']='a'
  mapTS 中无 'a' → 记录 mapTS['a']='e'

i=1: cs='g', ct='d'
  mapST 中无 'g' → 记录 mapST['g']='d'
  mapTS 中无 'd' → 记录 mapTS['d']='g'

i=2: cs='g', ct='d'
  mapST 中有 'g' → mapST['g']='d' == 'd' ✅
  mapTS 中有 'd' → mapTS['d']='g' == 'g' ✅

遍历完成，无冲突 → return true ✅
```

**方法三（首次出现索引）推演：**

```
s = "egg":
  'e' 首次出现 i=0 → 编码 [0]
  'g' 首次出现 i=1 → 编码 [0, 1]
  'g' 已出现 → 编码 [0, 1, 1]

t = "add":
  'a' 首次出现 i=0 → 编码 [0]
  'd' 首次出现 i=1 → 编码 [0, 1]
  'd' 已出现 → 编码 [0, 1, 1]

编码 [0,1,1] == [0,1,1] → return true ✅
```

#### 示例 2：`s = "foo"`, `t = "bar"`

**方法一（双向哈希表）推演：**

```
s = "foo", t = "bar"
mapST = {}, mapTS = {}

i=0: cs='f', ct='b'
  记录 mapST['f']='b', mapTS['b']='f'

i=1: cs='o', ct='a'
  记录 mapST['o']='a', mapTS['a']='o'

i=2: cs='o', ct='r'
  mapST 中有 'o' → mapST['o']='a' != 'r' ❌ 冲突！

return false ✅
```

**方法三（首次出现索引）推演：**

```
s = "foo": 编码 [0, 1, 1]
t = "bar": 编码 [0, 1, 2]

[0,1,1] != [0,1,2] → return false ✅
```

### 290 题推演

#### 示例 1：`pattern = "abba"`, `s = "dog cat cat dog"`

```
words = ["dog", "cat", "cat", "dog"]
pattern = "abba"

双向哈希表：
i=0: ch='a', word='dog' → mapPW['a']='dog', mapWP['dog']='a'
i=1: ch='b', word='cat' → mapPW['b']='cat', mapWP['cat']='b'
i=2: ch='b', word='cat' → mapPW['b']='cat'=='cat' ✅, mapWP['cat']='b'=='b' ✅
i=3: ch='a', word='dog' → mapPW['a']='dog'=='dog' ✅, mapWP['dog']='a'=='a' ✅

return true ✅
```

#### 示例 2：`pattern = "abba"`, `s = "dog cat cat fish"`

```
words = ["dog", "cat", "cat", "fish"]

i=0: ch='a', word='dog' → 记录映射
i=1: ch='b', word='cat' → 记录映射
i=2: ch='b', word='cat' → 匹配 ✅
i=3: ch='a', word='fish' → mapPW['a']='dog' != 'fish' ❌ 冲突！

return false ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **双向哈希表** | O(n) | O(k) | 最直观，逻辑清晰 | 需要两个哈希表 |
| **标准化编码** | O(n) | O(k) | 思路巧妙，代码简洁 | 需要构建编码数组 |
| **首次出现索引** | O(n) | O(k) | 最优解，一次遍历 | 稍抽象 |

> 注：n = 字符串长度，k = 字符集大小（205 题 k ≤ 256，290 题 k ≤ 不同单词数）。三种方法时间和空间复杂度相同，实际性能差异微乎其微。

---

## 优化方向对比

### 1. 为什么需要双向检查？

```
只检查正向映射的问题：

s = "ab", t = "aa"
正向检查：a → a, b → a  ← 两个不同字符映射到同一个 'a'
如果只检查正向，会误判为 true

必须同时检查反向：
  t 中的 'a' 已经被 s 中的 'a' 映射了
  现在 s 中的 'b' 也要映射到 'a'
  反向冲突 → return false ✅

这就是"双射"的核心：映射必须是可逆的。
```

### 2. 标准化编码为什么有效？

```
同构的本质：两个字符串的"结构模式"相同

"egg"  的模式：第1个字符、第2个字符、第2个字符 → [0, 1, 1]
"add"  的模式：第1个字符、第2个字符、第2个字符 → [0, 1, 1]
→ 结构相同 → 同构

"foo"  的模式：第1个字符、第2个字符、第2个字符 → [0, 1, 1]
"bar"  的模式：第1个字符、第2个字符、第3个字符 → [0, 1, 2]
→ 结构不同 → 不同构

标准化编码把"字符是谁"抽象掉了，只保留"结构关系"。
```

### 3. 三种方法的本质联系

```
双向哈希表：显式维护两套映射，逐对检查
标准化编码：隐式通过"首次出现序号"表达映射关系
首次出现索引：用"位置"代替"序号"，更精简

三者殊途同归，都是在验证：两个序列的"结构模式"是否一致。
面试推荐：先讲双向哈希表（最易理解），再优化到首次出现索引（最简洁）。
```

---

## 举一反三

双射映射是字符串/序列匹配中的经典模式，以下题目都用到了类似思路：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 205. 同构字符串** | 字符 → 字符的双射检查 |
| **LeetCode 290. 单词规律** | 字符 → 单词的双射检查 |
| **LeetCode 291. 单词规律 II** | 290 的进阶版，需要回溯找所有可能映射 |
| **LeetCode 890. 查找和替换模式** | 对一组字符串批量做同构检查 |
| **LeetCode 1153. 字符串转换** | 限制字符替换次数的同构变体 |

它们的共同模式：**用哈希表维护映射关系，同时检查双向唯一性，确保映射是合法的双射。**

---

## 总结

同构字符串和单词规律这两道题的核心是**理解双射映射的概念**：

1. **核心洞察**：同构 = 两个序列的"结构模式"相同，与具体字符/单词无关
2. **三种解法**：
   - 双向哈希表：最直观，面试首选开场
   - 标准化编码：巧妙地把问题转化为"比较编码"
   - **首次出现索引（推荐）**：最精简，一次遍历完成
3. **关键细节**：必须双向检查！只检查单向会漏掉"多对一"的非法映射

面试中建议这样展示：先用双向哈希表把"为什么要双向检查"讲清楚，然后用标准化编码说明"同构的本质是结构模式相同"，最后用首次出现索引作为最简实现。如果能主动提到 205 和 290 是同一类题，以及 291 是它们的进阶版，就是完美的面试表现。

最后记住一句话：**同构不是看"字符是什么"，而是看"字符的出现模式是否一致"。**
