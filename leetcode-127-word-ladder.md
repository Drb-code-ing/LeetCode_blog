---
title: LeetCode 127. 单词接龙
tags: [图论, BFS, 最短路径, 双向BFS, 虚拟节点]
difficulty: Hard
category: 图论
date: 2026-06-17
---

# LeetCode 127. 单词接龙 —— 把单词变成图，BFS 找最短路径

## 前言

这道题表面上是字符串变换，本质上是一个**图的最短路径问题**。如果你能看出"每个单词是图中的节点，相差一个字母的单词之间有一条边"，那么问题就转化为：**在一个无权图中，求起点到终点的最短路径**。

而"无权图的最短路径"，正是 BFS 的看家本领。

更妙的是，这道题还可以用**双向 BFS** 来进一步优化——从起点和终点同时向外扩散，谁先撞上对方的"领地"，谁就是最短路径的中点。双向 BFS 能让搜索空间从 `b^d` 降到 `2·b^(d/2)`，在单词长度较长时尤为有效。

本文将覆盖两种解法：**单向 BFS（虚拟节点优化建图）** → **双向 BFS（搜索空间对半砍）**，带你彻底掌握"把问题抽象成图 + BFS 求最短路"的核心套路。

---

## 问题描述

### LeetCode 127. Word Ladder（单词接龙）

> 字典 `wordList` 中从单词 `beginWord` 到 `endWord` 的**转换序列**是一个按下述规格形成的序列 `beginWord -> s1 -> s2 -> ... -> sk`：
>
> - 每一对相邻的单词只差一个字母。
> - 对于 `1 <= i <= k` 时，每个 `si` 都在 `wordList` 中。注意，`beginWord` 不需要在 `wordList` 中。
> - `sk == endWord`
>
> 给你两个单词 `beginWord` 和 `endWord` 和一个字典 `wordList`，返回**从 `beginWord` 到 `endWord` 的最短转换序列中的单词数目**。如果不存在这样的转换序列，返回 `0`。

**示例 1：**

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：5
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。
```

**示例 2：**

```
输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]
输出：0
解释：endWord "cog" 不在 wordList 中，所以无法进行转换。
```

**提示：**
- `1 <= beginWord.length <= 10`
- `endWord.length == beginWord.length`
- `1 <= wordList.length <= 5000`
- `wordList[i].length == beginWord.length`
- `beginWord`、`endWord` 和 `wordList[i]` 由小写英文字母组成
- `beginWord != endWord`
- `wordList` 中的所有单词**互不相同**

---

## 核心思想

### 把单词变换抽象成图

每一个单词是图中的一个**节点**。两个单词之间如果可以只改一个字母互相转换，它们之间就有一条**无向边**。

```
示例 1：
  beginWord = "hit", endWord = "cog"
  wordList = ["hot","dot","dog","lot","log","cog"]

图结构（每条边 = 差一个字母）：

        hit
         |
        hot
       /     \
    dot       lot
     |         |
    dog       log
       \     /
        cog
```

问题转化为：**在无权无向图中，求从 `hit` 到 `cog` 的最短路径长度**。

### 为什么是 BFS？

无权图的最短路径 = **BFS（广度优先搜索）**。BFS 一层一层地向外扩散，第一次遇到终点时就一定是最短路径。

> 💡 DFS 也能找到终点，但不能保证是最短的——它会顺着一条路走到黑。

### 建图的关键优化：虚拟节点（通配符模式）

最朴素的想法：两两比较所有单词，如果差异为 1 就建边 → O(N²·L)，N 可达 5000，会超时。

**优化思路**：不直接比较单词，而是引入"通配符模式"作为虚拟节点。

```
对于单词 "hot"，它可以匹配三种通配符模式：
  *ot   h*t   ho*

将每个通配符模式也作为图中的虚拟节点：

  "hot" ←→ "*ot"
  "hot" ←→ "h*t"
  "hot" ←→ "ho*"

那么 "hot" 和 "dot" 都连接到 "*ot" 这个虚拟节点，
通过虚拟节点间接相连：hot → *ot → dot

路径长度计算：
  真实节点 → 虚拟节点 → 真实节点 = 两步
  所以最终答案需要除以 2，或单独维护真实节点的距离
```

### 图解：虚拟节点建图

```
以 wordList = ["hot","dot","dog"] 为例：

         *ot        h*t        ho*
        /  \         |          |
     hot   dot      hot        hot
             |       |
            do*     *ot
             |        |
            dog      dot
             |
            *og
             |
            dog

真实节点：hot, dot, dog（白底）
虚拟节点：*ot, h*t, ho*, do*, *og（灰底）

hot 通过 *ot 连接到 dot → hot 和 dot 相差一个字母 ✅
```

> 这种技巧将建图复杂度从 O(N²·L) 优化到了 **O(N·L²)**。每个单词只需要枚举它所有的通配符模式（L 种），每种通配符模式生成一个虚拟节点 ID。

---

## 思路分析

### 解法一：单向 BFS（虚拟节点建图）⭐ 推荐

**核心思路**：为每个单词生成所有通配符模式（如 `*ot`、`h*t`、`ho*`），将单词和模式都作为图中的节点。然后在图上做标准 BFS 求最短路径。

```
算法框架（单向 BFS）：

1. 预处理：
   - 将 wordList 转为 Set，便于 O(1) 查找
   - 如果 endWord 不在 Set 中 → 直接返回 0

2. 初始化 BFS：
   - 队列 queue = [beginWord]
   - 距离映射 dist = { beginWord: 1 }（题目要求的是序列中的单词数，起点本身算 1）
   - 如果 beginWord 不在 wordList 中，将其加入 Set

3. BFS 过程：
   while queue 非空：
     取出队首 word
     如果 word === endWord → 返回 dist[word]
     
     枚举 word 的每个通配符模式：
       生成虚拟节点 key = word 将第 i 位替换为 '*'
       
       遍历所有能与该模式匹配的真实单词（即 wordList 中所有替换第 i 位为 * 后等于 key 的单词）：
         如果 nextWord 未被访问过：
           dist[nextWord] = dist[word] + 1
           queue.push(nextWord)

4. BFS 结束未找到 → 返回 0
```

**优化说明**：实际上有两种常见的 BFS 实现方式：

**方式 A：虚拟节点法（本解法的核心）**

```javascript
// 建图：为每种通配符模式建立一个"邻居列表"
// patternMap: { "*ot": ["hot", "dot", "lot"], "h*t": ["hot"], ... }

// BFS 时，从当前单词出发，访问所有通配符模式对应的邻居列表
for (每个通配符模式 pattern of word) {
    for (每个邻居 nextWord of patternMap[pattern]) {
        if (nextWord 未访问) {
            // 直接到达下一个真实单词，距离 +1
        }
    }
}
```

**方式 B：显式建虚拟节点图**

```javascript
// 为每个单词编号，为每个通配符模式编号
// graph[wordId].push(patternId), graph[patternId].push(wordId)
// BFS 走两步（word → pattern → word）才算真正跨过一个单词
```

> ⭐ 推荐方式 A（模式映射法），代码更简洁，免去距离 ÷2 的麻烦。

### 解法二：双向 BFS（搜索空间减半）⭐ 进阶

**核心思路**：同时从起点和终点出发，每次选择较小的那一端扩展。两端在某处"相遇"时，路径长度 = 起点距离 + 终点距离 - 1（中间节点被算两次）。

```
算法框架（双向 BFS）：

1. 初始化：
   - beginSet = { beginWord }   // 从起点出发的"当前层"
   - endSet = { endWord }       // 从终点出发的"当前层"
   - visited = Set([beginWord, endWord])  // 全局访问标记
   - 如果 endWord 不在 wordList 中 → 返回 0
   - distance = 1  // 当前步数

2. 双向扩散：
   while beginSet 非空 and endSet 非空：
     
     // 优化：始终扩展较小的集合
     if beginSet.size > endSet.size:
       swap(beginSet, endSet)
     
     nextSet = new Set()
     
     for 每个 word in beginSet:
       枚举 word 的每个通配符模式：
         遍历该模式下的所有邻居单词 nextWord：
           if nextWord in endSet:
             return distance + 1  // 相遇！
           if nextWord 未访问:
             visited.add(nextWord)
             nextSet.add(nextWord)
     
     beginSet = nextSet
     distance++
     
     // 注意：当交换了 begin/end 时，distance 的含义也交换了
     // 实际上更推荐分别维护两个距离
```

**更推荐的实现方式**：分别维护 beginVisited/endVisited 和它们各自的距离。

```
真正推荐的双向 BFS 写法：

while beginQueue 非空 and endQueue 非空：
  
  if beginQueue.length > endQueue.length:
    交换 beginQueue 和 endQueue
    交换 beginVisited 和 endVisited
  
  levelSize = beginQueue.length
  重复 levelSize 次：
    取出队首 word
    
    如果 word 在 endVisited 中：
      返回 beginVisited[word] + endVisited[word] - 1
    
    枚举所有通配符模式的邻居：
      如果邻居未被 beginVisited 访问：
        beginVisited[邻居] = beginVisited[word] + 1
        beginQueue.push(邻居)
```

> 💡 双向 BFS 的关键优化点：**始终扩展较小的那一端（beginQueue/endQueue）**。这样能最大限度地减少搜索空间。

### 为什么双向 BFS 更快？

```
假设每个单词平均有 k 个邻居，最短路径长度为 d：

单向 BFS 搜索的节点数 ≈ k^d
双向 BFS 搜索的节点数 ≈ 2 × k^(d/2)

当 d = 5, k = 3 时：
  单向 ≈ 3^5 = 243
  双向 ≈ 2 × 3^2.5 ≈ 2 × 15.6 = 31
  
差距显著！
```

---

## 代码实现

### JavaScript 版本

#### 方法一：单向 BFS（模式映射法）

```javascript
/**
 * @param {string} beginWord
 * @param {string} endWord
 * @param {string[]} wordList
 * @return {number}
 */
var ladderLength = function(beginWord, endWord, wordList) {
    const wordSet = new Set(wordList);
    if (!wordSet.has(endWord)) return 0;

    // 构建通配符模式 → 匹配单词列表 的映射
    const patternMap = new Map();
    const L = beginWord.length;

    for (const word of wordSet) {
        for (let i = 0; i < L; i++) {
            const key = word.slice(0, i) + '*' + word.slice(i + 1);
            if (!patternMap.has(key)) {
                patternMap.set(key, []);
            }
            patternMap.get(key).push(word);
        }
    }

    // BFS
    const queue = [beginWord];
    const visited = new Set([beginWord]);
    let steps = 1;  // 序列中的单词数量，起点算 1

    while (queue.length > 0) {
        const levelSize = queue.length;
        for (let k = 0; k < levelSize; k++) {
            const word = queue.shift();

            if (word === endWord) {
                return steps;
            }

            // 枚举当前单词的所有通配符模式
            for (let i = 0; i < L; i++) {
                const key = word.slice(0, i) + '*' + word.slice(i + 1);
                const neighbors = patternMap.get(key) || [];

                for (const nextWord of neighbors) {
                    if (!visited.has(nextWord)) {
                        visited.add(nextWord);
                        queue.push(nextWord);
                    }
                }
            }
        }
        steps++;
    }

    return 0;
};
```

#### 方法二：双向 BFS

```javascript
/**
 * @param {string} beginWord
 * @param {string} endWord
 * @param {string[]} wordList
 * @return {number}
 */
var ladderLength = function(beginWord, endWord, wordList) {
    const wordSet = new Set(wordList);
    if (!wordSet.has(endWord)) return 0;

    const L = beginWord.length;

    // 构建通配符模式映射
    const patternMap = new Map();
    for (const word of wordSet) {
        for (let i = 0; i < L; i++) {
            const key = word.slice(0, i) + '*' + word.slice(i + 1);
            if (!patternMap.has(key)) {
                patternMap.set(key, []);
            }
            patternMap.get(key).push(word);
        }
    }

    // 双向 BFS：两套队列和距离
    let beginQueue = [beginWord];
    let endQueue = [endWord];
    const beginDist = new Map([[beginWord, 1]]);
    const endDist = new Map([[endWord, 1]]);

    while (beginQueue.length > 0 && endQueue.length > 0) {
        // 始终扩展较小的那一端
        if (beginQueue.length > endQueue.length) {
            [beginQueue, endQueue] = [endQueue, beginQueue];
        }
        // 注意：交换 queue 时必须同时交换距离映射
        // 更安全的写法是封装成函数，这里为了清晰直接判断
        // 实际上我们通过下面的方式避免交换：
        // 把扩展逻辑抽成函数，根据当前是 begin 还是 end 来查不同的 dist

        // 简化版：始终扩展 beginQueue，只在它比 endQueue 大时交换
        // （这里 beginQueue 和 beginDist 配对，end 同理）
        let t = beginQueue.length > endQueue.length
            ? [beginQueue, endQueue, beginDist, endDist]
            : [endQueue, beginQueue, endDist, beginDist];
        // 不，这样太绕了。用更清晰的写法：

        // 重新来：最简单的实现方式
    }

    return 0;
};
```

> ⚠️ 双向 BFS 的交换逻辑容易写错。下面给一个更清晰的 Python 实现，再给一个更干净的 JS 版本。

#### 方法二：双向 BFS（清晰版）

```javascript
/**
 * @param {string} beginWord
 * @param {string} endWord
 * @param {string[]} wordList
 * @return {number}
 */
var ladderLength = function(beginWord, endWord, wordList) {
    const wordSet = new Set(wordList);
    if (!wordSet.has(endWord)) return 0;

    const L = beginWord.length;

    // 构建通配符模式映射
    const patternMap = new Map();
    for (const word of wordSet) {
        for (let i = 0; i < L; i++) {
            const key = word.slice(0, i) + '*' + word.slice(i + 1);
            if (!patternMap.has(key)) patternMap.set(key, []);
            patternMap.get(key).push(word);
        }
    }

    // 辅助函数：扩展一层
    function expandLevel(queue, currentDist, otherDist, patternMap, L) {
        const levelSize = queue.length;
        for (let k = 0; k < levelSize; k++) {
            const word = queue.shift();

            for (let i = 0; i < L; i++) {
                const key = word.slice(0, i) + '*' + word.slice(i + 1);
                const neighbors = patternMap.get(key) || [];

                for (const nextWord of neighbors) {
                    if (otherDist.has(nextWord)) {
                        // 相遇！
                        return currentDist.get(word) + otherDist.get(nextWord);
                    }
                    if (!currentDist.has(nextWord)) {
                        currentDist.set(nextWord, currentDist.get(word) + 1);
                        queue.push(nextWord);
                    }
                }
            }
        }
        return -1;  // 本层未相遇
    }

    let beginQueue = [beginWord];
    let endQueue = [endWord];
    const beginDist = new Map([[beginWord, 1]]);
    const endDist = new Map([[endWord, 1]]);

    while (beginQueue.length > 0 && endQueue.length > 0) {
        // 始终扩展较小的那一端
        let result;
        if (beginQueue.length <= endQueue.length) {
            result = expandLevel(beginQueue, beginDist, endDist, patternMap, L);
        } else {
            result = expandLevel(endQueue, endDist, beginDist, patternMap, L);
        }

        if (result !== -1) return result;
    }

    return 0;
};
```

### Python 版本

#### 方法一：单向 BFS（模式映射法）

```python
from collections import defaultdict, deque
from typing import List


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        word_set = set(wordList)
        if endWord not in word_set:
            return 0

        L = len(beginWord)

        # 构建通配符模式 → 匹配单词列表 的映射
        pattern_map = defaultdict(list)
        for word in word_set:
            for i in range(L):
                key = word[:i] + '*' + word[i + 1:]
                pattern_map[key].append(word)

        # BFS
        queue = deque([beginWord])
        visited = {beginWord}
        steps = 1  # 序列长度，起点算 1

        while queue:
            for _ in range(len(queue)):
                word = queue.popleft()

                if word == endWord:
                    return steps

                # 枚举当前单词的所有通配符模式
                for i in range(L):
                    key = word[:i] + '*' + word[i + 1:]
                    for next_word in pattern_map.get(key, []):
                        if next_word not in visited:
                            visited.add(next_word)
                            queue.append(next_word)

            steps += 1

        return 0
```

#### 方法二：双向 BFS

```python
from collections import defaultdict, deque
from typing import List


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        word_set = set(wordList)
        if endWord not in word_set:
            return 0

        L = len(beginWord)

        # 构建通配符模式映射
        pattern_map = defaultdict(list)
        for word in word_set:
            for i in range(L):
                key = word[:i] + '*' + word[i + 1:]
                pattern_map[key].append(word)

        def expand_level(queue: deque, cur_dist: dict, other_dist: dict) -> int:
            """扩展一层，返回相遇时的总距离，-1 表示未相遇"""
            for _ in range(len(queue)):
                word = queue.popleft()

                for i in range(L):
                    key = word[:i] + '*' + word[i + 1:]
                    for next_word in pattern_map.get(key, []):
                        if next_word in other_dist:
                            # 相遇！
                            return cur_dist[word] + other_dist[next_word]
                        if next_word not in cur_dist:
                            cur_dist[next_word] = cur_dist[word] + 1
                            queue.append(next_word)
            return -1

        begin_queue = deque([beginWord])
        end_queue = deque([endWord])
        begin_dist = {beginWord: 1}
        end_dist = {endWord: 1}

        while begin_queue and end_queue:
            # 始终扩展较小的那一端
            if len(begin_queue) <= len(end_queue):
                result = expand_level(begin_queue, begin_dist, end_dist)
            else:
                result = expand_level(end_queue, end_dist, begin_dist)

            if result != -1:
                return result

        return 0
```

---

## 逐步推演

以示例 1 为例，演示单向 BFS 的完整过程：

```
beginWord = "hit", endWord = "cog"
wordList = ["hot","dot","dog","lot","log","cog"]
```

### 第一步：构建通配符模式映射

```
扫描 wordList 中每个单词，生成通配符 → 单词映射：

"hot":
  *ot → 加入 "hot"
  h*t → 加入 "hot"
  ho* → 加入 "hot"

"dot":
  *ot → 加入 "dot"
  d*t → 加入 "dot"
  do* → 加入 "dot"

"dog":
  d*g → 加入 "dog"
  *og → 加入 "dog"
  do* → 加入 "dog"  （do* 现在有 ["dot", "dog"]）

...以此类推

最终 patternMap：
  *ot → ["hot","dot","lot"]
  h*t → ["hot","hit"]  ★ "hit" 虽然不在 wordList 中，但也要加入映射
  ho* → ["hot"]
  d*t → ["dot"]
  do* → ["dot","dog"]
  *og → ["dog","log","cog"]
  d*g → ["dog"]
  l*t → ["lot"]
  lo* → ["lot","log"]
  l*g → ["log"]
  c*g → ["cog"]
  co* → ["cog"]

注意：beginWord "hit" 也需要加入映射，否则无法从 "hit" 出发找到邻居。
实际上我们只在 BFS 时对"当前访问的单词"做通配符展开，不需要预先加入。
```

### 第二步：BFS 逐层扩展

```
第 1 层 (steps = 1)：
  queue = ["hit"]
  
  取出 "hit":
    检查是否等于 "cog" → 否
    枚举通配符模式：
      *it → patternMap 中没有（只有 h*t 有 "hit"）
      h*t → 邻居 = ["hot", "hit"]
        "hot" 未访问 → visited.add("hot"), queue.push("hot")
        "hit" 是自己，跳过
      hi* → patternMap 中没有
  
  steps++ → steps = 2

  当前状态：
    visited = {"hit", "hot"}
    queue = ["hot"]
```

```
第 2 层 (steps = 2)：
  queue = ["hot"]
  
  取出 "hot":
    检查是否等于 "cog" → 否
    枚举通配符模式：
      *ot → 邻居 = ["hot","dot","lot"]
        "hot" 已访问
        "dot" 未访问 → visited.add("dot"), queue.push("dot")
        "lot" 未访问 → visited.add("lot"), queue.push("lot")
      h*t → 邻居 = ["hot","hit"]
        都已访问
      ho* → 邻居 = ["hot"]
        已访问
  
  steps++ → steps = 3

  当前状态：
    visited = {"hit", "hot", "dot", "lot"}
    queue = ["dot", "lot"]
```

```
第 3 层 (steps = 3)：
  queue = ["dot", "lot"]
  
  取出 "dot":
    检查是否等于 "cog" → 否
    枚举通配符模式：
      *ot → 邻居 = ["hot","dot","lot"]
        都已访问
      d*t → 邻居 = ["dot"]
        已访问
      do* → 邻居 = ["dot","dog"]
        "dog" 未访问 → visited.add("dog"), queue.push("dog")
  
  取出 "lot":
    检查是否等于 "cog" → 否
    枚举通配符模式：
      *ot → 邻居 = ["hot","dot","lot"]
        都已访问
      l*t → 邻居 = ["lot"]
        已访问
      lo* → 邻居 = ["lot","log"]
        "log" 未访问 → visited.add("log"), queue.push("log")
  
  steps++ → steps = 4

  当前状态：
    visited = {"hit", "hot", "dot", "lot", "dog", "log"}
    queue = ["dog", "log"]
```

```
第 4 层 (steps = 4)：
  queue = ["dog", "log"]
  
  取出 "dog":
    检查是否等于 "cog" → 否
    枚举通配符模式：
      d*g → 邻居 = ["dog"]
        已访问
      *og → 邻居 = ["dog","log","cog"]
        "cog" 未访问 → visited.add("cog"), queue.push("cog")
      do* → 邻居 = ["dot","dog"]
        都已访问
  
  取出 "log":
    检查是否等于 "cog" → 否
    枚举通配符模式：
      l*g → 邻居 = ["log"]
        已访问
      *og → 邻居 = ["dog","log","cog"]
        都已访问或已在 queue 中
      lo* → 邻居 = ["lot","log"]
        都已访问
  
  steps++ → steps = 5

  当前状态：
    queue = ["cog"]
```

```
第 5 层 (steps = 5)：
  queue = ["cog"]
  
  取出 "cog":
    检查是否等于 "cog" → 是！
    返回 steps = 5  ✅
```

### BFS 层级可视化

```
        第1层          第2层          第3层          第4层          第5层

        hit ──────→ hot ──────→ dot ──────→ dog ──────→ cog
                              → lot ──────→ log
                                          
        steps=1       steps=2       steps=3       steps=4       steps=5
```

### 双向 BFS 推演（同样示例）

```
起点层：beginQueue = ["hit"], beginDist = {hit: 1}
终点层：endQueue = ["cog"], endDist = {cog: 1}

────────────────────────────────────────────────────────────
第 1 轮：
  beginQueue(1) ≤ endQueue(1) → 扩展 beginQueue

  从 "hit" 扩展（枚举 *ot, h*t, hi*）：
    h*t → 邻居 "hot"：
      "hot" 不在 endDist 中
      beginDist["hot"] = 2, beginQueue = ["hot"]
    （*ot 中只有 "hit" 自己，hi* 为空）
  
  未相遇。beginQueue = ["hot"], beginDist = {hit:1, hot:2}

────────────────────────────────────────────────────────────
第 2 轮：
  beginQueue(1) ≤ endQueue(1) → 扩展 beginQueue

  从 "hot" 扩展（枚举 *ot, h*t, ho*）：
    *ot → 邻居 "dot", "lot"：
      "dot" 不在 endDist → beginDist["dot"] = 3
      "lot" 不在 endDist → beginDist["lot"] = 3
      beginQueue = ["dot", "lot"]
  
  未相遇。beginQueue = ["dot", "lot"]

────────────────────────────────────────────────────────────
第 3 轮：
  beginQueue(2) > endQueue(1) → 扩展 endQueue（较小的）

  从 "cog" 扩展（枚举 c*g, *og, co*）：
    *og → 邻居 "dog", "log"：
      "dog" 不在 beginDist → endDist["dog"] = 2
      "log" 不在 beginDist → endDist["log"] = 2
      endQueue = ["dog", "log"]
  
  未相遇。endQueue = ["dog", "log"]

────────────────────────────────────────────────────────────
第 4 轮：
  beginQueue(2) ≤ endQueue(2) → 扩展 beginQueue

  从 "dot" 扩展：
    do* → 邻居 "dog"：
      "dog" 在 endDist 中！ ← 相遇！
      返回 beginDist["dot"] + endDist["dog"] = 3 + 2 = 5 ✅
```

> 对比单向 BFS 的 4 轮扩展到第 5 层，双向 BFS 只用了 4 轮（且每轮搜索的节点数更少）就找到了最短路径。

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 说明 |
|------|-----------|-----------|------|
| **单向 BFS** | O(N × L²) | O(N × L) | N 个单词，每个 L 长度，建图 O(NL²)，BFS O(NL) |
| **双向 BFS** | O(N × L²) | O(N × L) | 同上，但实际搜索节点数远小于单向 |

### 详细分析

**时间复杂度：**

- **建图**：对每个单词（N 个），枚举所有通配符模式（L 种），生成 key 需要 O(L) 时间。总计 O(N × L²)。
- **BFS**：最坏情况下访问所有 N 个单词，每个单词枚举 L 个 key，每个 key 可能对应多个邻居。但由于每个单词被访问一次，总遍历邻居次数的上限是 O(N × L)。所以 BFS 部分 O(N × L)。
- **总计**：O(N × L²)。当 N=5000, L=10 时，NL² = 500,000，完全可行。

**空间复杂度：**

- patternMap 存储 N 个单词 × L 个模式，每个 key 长度为 L。总计 O(N × L)。
- visited 集合和 queue 最坏 O(N)。

**双向 BFS 为什么实际更快？**

最坏时间复杂度相同，但搜索空间从 `b^d` 降到 `2 × b^(d/2)`，实际访问的节点数大幅减少。当最短路径长度 d 较大时，双向 BFS 的优势极为显著。

---

## 举一反三

### 本题在"图论最短路径"体系中的位置

```
LeetCode 126. 单词接龙 II（Hard）
  → 本题的升级版：不仅要长度，还要返回所有最短路径
  → 需要 BFS 找到最短距离 + DFS 回溯所有最短路径
  ↑
LeetCode 127. 单词接龙（Hard）⭐ 本题
  → 无权图最短路径 = BFS
  → 优化建图：通配符虚拟节点
  ↓
LeetCode 433. 最小基因变化（Medium）
  → 几乎一模一样的题！只是把字母换成了基因字符
  ↓
LeetCode 752. 打开转盘锁（Medium）
  → 同样是状态转移 + BFS 最短路径，多了"死锁"约束
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| **126. 单词接龙 II** | BFS 分层 + DFS 回溯 | 本题的升级版，需要所有最短路径 |
| **433. 最小基因变化** | BFS + 状态转移 | 几乎完全一样的题目，换了个壳 |
| **752. 打开转盘锁** | BFS + 禁止状态 | 同样是状态转移 BFS，多了 deadends |
| **279. 完全平方数** | BFS 在数论上的应用 | 隐式图 BFS 最短路，不显式建图 |
| **207/210. 课程表** | 拓扑排序 | 同样是图论基础问题，不同的问题类型 |

### 通用模板：隐式图 BFS 最短路径

```javascript
// 隐式图 BFS 最短路径 通用模板
function shortestPath(start, target, getNeighbors) {
    const queue = [start];
    const visited = new Set([start]);
    let steps = 1;

    while (queue.length > 0) {
        const levelSize = queue.length;
        for (let i = 0; i < levelSize; i++) {
            const current = queue.shift();

            if (current === target) {
                return steps;
            }

            for (const next of getNeighbors(current)) {
                if (!visited.has(next)) {
                    visited.add(next);
                    queue.push(next);
                }
            }
        }
        steps++;
    }

    return -1;  // 不可达
}
```

这套模板稍作修改就能应对 127、433、752、279 等一系列"隐式图最短路径"题目。

---

## 总结

单词接龙这道题有四个层次的收获：

1. **问题抽象**：把"字符串变换"抽象成"图的最短路径问题"——这是解决本题最关键的一步。

2. **建图优化**：直接 O(N²) 两两比较会超时，引入**通配符虚拟节点**将建图复杂度降到 O(NL²)。

3. **BFS 求最短路**：无权图的最短路径，BFS 是标准解法。关键代码只有 20 行。

4. **双向 BFS 进阶**：从两端同时搜索，始终扩展较小的那一端，将搜索空间从 `b^d` 降到 `2·b^(d/2)`。

| 步骤 | 操作 |
|------|------|
| ① | 检查 endWord 是否在 wordList 中 |
| ② | 构建通配符模式映射表（patternMap） |
| ③ | BFS：从 beginWord 出发，逐层扩展 |
| ④ | 遇到 endWord 时返回当前层数（steps） |
| ⑤ | BFS 结束未遇到 → 返回 0 |

**三步记忆法**：
1. **建图**：对每个单词，枚举 L 个通配符模式（`word[:i] + '*' + word[i+1:]`），建立模式 → 单词列表的映射
2. **BFS**：从 beginWord 出发，逐层扩展，遇到 endWord 就返回当前步数
3. **进阶**：双向 BFS — 从两端同时搜索，每次扩展较小的队列

> **记住一句话：单词接龙 = 隐式图最短路径 = BFS。核心优化是用通配符模式做"虚拟节点"，避免 O(N²) 两两比较。**

在纸上画出"hit → hot → dot → dog → cog"这个链条，手动模拟一遍 BFS 的逐层扩展——你会发现这道题其实非常直观。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 126. 单词接龙 II](https://leetcode.com/problems/word-ladder-ii/)（本题升级版，所有最短路径） | [LeetCode 433. 最小基因变化](https://leetcode.com/problems/minimum-genetic-mutation/)（换壳题） | [LeetCode 752. 打开转盘锁](https://leetcode.com/problems/open-the-lock/)（状态转移 BFS） | 图论系列持续更新中，关注不迷路。
