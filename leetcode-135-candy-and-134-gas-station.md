---
title: LeetCode 135 & 134. 分发糖果 & 加油站
tags: [贪心, 双遍历, 环形数组, 局部最优, 全局最优]
difficulty: Hard / Medium
category: 贪心
date: 2026-05-31
---

# LeetCode 135 & 134. 分发糖果 & 加油站 —— 贪心两面镜：从"双向扫描"到"全局盈余"

## 前言

这两道题放在一起写，是因为它们共享同一个贪心核心思想：

> **当局部最优的简单叠加就能保证全局最优时，贪心就是最佳策略。**

- **分发糖果（135）**：需要从左到右、再从右到左，两次遍历才能满足所有约束——"双向贪心"
- **加油站（134）**：只需要一次遍历，但需要一个关键洞察——"如果总油量 ≥ 总消耗，一定有解"

两道题都是贪心经典，但思路截然不同。135 的难点在于**约束条件的拆解**（左邻右邻都要考虑），134 的难点在于**反直觉的证明**（为什么从亏最多的点后面开始一定能行？）。

本文将为每道题提供多种解法，每种方法都附带 JavaScript 和 Python 双语言实现，以及详细的手动推演过程。

---

# 第一部分：分发糖果（LeetCode 135）

## 问题描述

### LeetCode 135. Candy（分发糖果）

> `n` 个孩子站成一排。给你一个整数数组 `ratings`，表示每个孩子的评分。
>
> 你需要给每个孩子分配糖果，遵循以下规则：
> 1. 每个孩子至少分配 **1** 个糖果
> 2. 相邻两个孩子中，评分更高的那个必须获得**更多**的糖果
>
> 返回你需要准备的**最少糖果总数**。

**示例：**

```
输入：ratings = [1,0,2]
输出：5
解释：分别给 [2,1,2] 个糖果

输入：ratings = [1,2,2]
输出：4
解释：分别给 [1,2,1] 个糖果（评分相同不要求更多）
```

---

## 核心思想

### 难点在哪？

规则只有两条，看起来很简单，但**同时满足**很难：

```
孩子:    A  B  C
评分:    1  3  2

如果从左到右:  A=1, B=2, C=1  ← C 评分比 B 低，OK
如果从右到左:  C=1, B=2, A=1  ← A 评分比 B 低，OK
但这不是最少的！正确答案是 [1,3,1] 或 [1,2,1]。

再看:  ratings = [1, 3, 2, 1]
从左到右满足: [1, 2, 1, 1]
但从右到左: 第3个孩子(评分2) > 第4个孩子(评分1)，OK
            第2个孩子(评分3) > 第3个孩子(评分2)，但 2 > 1 ✗ ← 违反了！
```

**关键洞察：** 单次遍历无法同时满足左右两个方向的约束。但我们可以**拆成两次**：

1. **从左到右**：只处理"右比左大"的情况
2. **从右到左**：只处理"左比右大"的情况
3. **取两次结果的最大值**：两个方向的约束同时满足

---

## 思路分析

### 解法一：双遍历贪心（经典解法）

**第一次遍历（左→右）：** 如果右边孩子评分比左边高，右边糖果 = 左边 + 1

```
ratings = [1, 3, 2, 1]

左→右扫描:
  i=0: candy = [1, _, _, _]     （每人至少1个）
  i=1: 3 > 1 → candy[1] = candy[0]+1 = 2  → [1, 2, _, _]
  i=2: 2 > 3? → 否              → [1, 2, 1, _]
  i=3: 1 > 2? → 否              → [1, 2, 1, 1]
```

**第二次遍历（右→左）：** 如果左边孩子评分比右边高，左边糖果 = max(当前值, 右边+1)

```
右→左扫描 (修正左邻约束):
  i=2: 2 > 1 → candy[2] = max(1, candy[3]+1) = max(1,2) = 2  → [1, 2, 2, 1]
  i=1: 3 > 2 → candy[1] = max(2, candy[2]+1) = max(2,3) = 3  → [1, 3, 2, 1]
  i=0: 1 > 3? → 否                                             → [1, 3, 2, 1]

总糖果 = 1+3+2+1 = 7
```

**为什么要取 max？** 因为第二次遍历时，左边孩子可能已经从第一次遍历获得了一个较大的值。必须保留更大的那个才能同时满足两个方向的约束。

### 解法二：一次遍历（峰谷法）

观察到糖果分配的形状像一连串的"山坡"：

```
ratings = [1, 3, 5, 3, 2, 1]
candy   = [1, 2, 3, 2, 1, 1]

         3
        / \
  2   2   1
 / \ / \ / \
1   1   1   1
```

- **上坡**：每次比前一个评分高，糖果递增
- **下坡**：每次比前一个评分低，糖果递减
- **峰顶**：需要被上下坡共享，取 max(上坡高度, 下坡高度)

关键：记录当前上坡长度 `up` 和下坡长度 `down`，遇到平地或峰顶时结算。

### 解法三：排序法（O(n log n)）

按评分从低到高排序，每次处理评分最低的孩子（他只需要 1 个），然后检查邻居是否需要更新。时间复杂度 O(n log n)，但思路非常直观。

---

## 代码实现

### JavaScript 版本

#### 方法一：双遍历贪心

```javascript
/**
 * @param {number[]} ratings
 * @return {number}
 */
var candy = function(ratings) {
    const n = ratings.length;
    const candy = new Array(n).fill(1);

    // 从左到右：右边评分高 → 右边糖果 = 左边 + 1
    for (let i = 1; i < n; i++) {
        if (ratings[i] > ratings[i - 1]) {
            candy[i] = candy[i - 1] + 1;
        }
    }

    // 从右到左：左边评分高 → 左边糖果 = max(当前, 右边 + 1)
    for (let i = n - 2; i >= 0; i--) {
        if (ratings[i] > ratings[i + 1]) {
            candy[i] = Math.max(candy[i], candy[i + 1] + 1);
        }
    }

    return candy.reduce((sum, c) => sum + c, 0);
};
```

#### 方法二：峰谷法（一次遍历）

```javascript
var candy = function(ratings) {
    const n = ratings.length;
    if (n <= 1) return n;

    let total = 1;  // 第一个孩子至少1个
    let up = 0, down = 0, peak = 0;

    for (let i = 1; i < n; i++) {
        if (ratings[i] > ratings[i - 1]) {
            // 上坡
            up++;
            down = 0;
            peak = up;         // 记录峰顶高度
            total += up + 1;
        } else if (ratings[i] === ratings[i - 1]) {
            // 平地：重置所有计数器
            up = down = peak = 0;
            total += 1;
        } else {
            // 下坡
            down++;
            up = 0;
            total += down + 1;
            // 如果下坡长度 ≥ 峰顶高度，峰顶需要额外 +1
            if (down >= peak) {
                total++;
            }
        }
    }

    return total;
};
```

#### 方法三：排序法

```javascript
var candy = function(ratings) {
    const n = ratings.length;
    const sorted = ratings.map((r, i) => [r, i]).sort((a, b) => a[0] - b[0]);
    const candy = new Array(n).fill(1);

    for (const [rate, idx] of sorted) {
        // 检查左邻居
        if (idx > 0 && ratings[idx] > ratings[idx - 1]) {
            candy[idx] = Math.max(candy[idx], candy[idx - 1] + 1);
        }
        // 检查右邻居
        if (idx < n - 1 && ratings[idx] > ratings[idx + 1]) {
            candy[idx] = Math.max(candy[idx], candy[idx + 1] + 1);
        }
    }

    return candy.reduce((sum, c) => sum + c, 0);
};
```

### Python 版本

#### 方法一：双遍历贪心

```python
def candy(ratings: list[int]) -> int:
    """方法一：双遍历贪心"""
    n = len(ratings)
    candies = [1] * n

    # 从左到右：右边评分高 → 右边糖果 = 左边 + 1
    for i in range(1, n):
        if ratings[i] > ratings[i - 1]:
            candies[i] = candies[i - 1] + 1

    # 从右到左：左边评分高 → 左边糖果 = max(当前, 右边 + 1)
    for i in range(n - 2, -1, -1):
        if ratings[i] > ratings[i + 1]:
            candies[i] = max(candies[i], candies[i + 1] + 1)

    return sum(candies)
```

#### 方法二：峰谷法（一次遍历）

```python
def candy(ratings: list[int]) -> int:
    """方法二：峰谷法（一次遍历）"""
    n = len(ratings)
    if n <= 1:
        return n

    total = 1  # 第一个孩子至少1个
    up = down = peak = 0

    for i in range(1, n):
        if ratings[i] > ratings[i - 1]:
            # 上坡
            up += 1
            down = 0
            peak = up         # 记录峰顶高度
            total += up + 1
        elif ratings[i] == ratings[i - 1]:
            # 平地：重置
            up = down = peak = 0
            total += 1
        else:
            # 下坡
            down += 1
            up = 0
            total += down + 1
            # 下坡长度 ≥ 峰顶高度时，峰顶需要额外 +1
            if down >= peak:
                total += 1

    return total
```

#### 方法三：排序法

```python
def candy(ratings: list[int]) -> int:
    """方法三：排序法"""
    n = len(ratings)
    candies = [1] * n
    # 按评分从低到高排序，处理索引
    sorted_indices = sorted(range(n), key=lambda i: ratings[i])

    for idx in sorted_indices:
        # 检查左邻居
        if idx > 0 and ratings[idx] > ratings[idx - 1]:
            candies[idx] = max(candies[idx], candies[idx - 1] + 1)
        # 检查右邻居
        if idx < n - 1 and ratings[idx] > ratings[idx + 1]:
            candies[idx] = max(candies[idx], candies[idx + 1] + 1)

    return sum(candies)
```

---

### 逐步推演

以 `ratings = [1, 3, 2, 1]` 为例。

#### 双遍历贪心推演

```
初始: candy = [1, 1, 1, 1]

=== 第一次遍历：左→右 ===
i=1: ratings[1]=3 > ratings[0]=1 → candy[1] = candy[0]+1 = 2
  candy = [1, 2, 1, 1]

i=2: ratings[2]=2 > ratings[1]=3? → 否
  candy = [1, 2, 1, 1]

i=3: ratings[3]=1 > ratings[2]=2? → 否
  candy = [1, 2, 1, 1]

=== 第二次遍历：右→左 ===
i=2: ratings[2]=2 > ratings[3]=1 → candy[2] = max(1, candy[3]+1) = max(1,2) = 2
  candy = [1, 2, 2, 1]

i=1: ratings[1]=3 > ratings[2]=2 → candy[1] = max(2, candy[2]+1) = max(2,3) = 3
  candy = [1, 3, 2, 1]

i=0: ratings[0]=1 > ratings[1]=3? → 否
  candy = [1, 3, 2, 1]

总糖果 = 1 + 3 + 2 + 1 = 7 ✅
```

#### 峰谷法推演

以 `ratings = [1, 3, 2, 1]` 为例，峰谷法把糖果分配看作一连串上坡和下坡：

```
评分:  1  3  2  1
糖果:  1  2  1  1   ← 初始分配（上坡递增，下坡递减）
         ↑
        峰顶

实际上峰顶应该是 max(上坡长度, 下坡长度) + 1 = max(1, 1) + 1 = 3

修正后: [1, 3, 2, 1]，总 = 7
```

峰谷法的核心逻辑：遍历时维护 `up`（连续上坡长度）和 `down`（连续下坡长度）。每到一个下坡，检查 `down >= peak`——如果下坡长度超过了峰顶之前的上坡长度，说明峰顶的糖果数不够，需要额外 +1 来补偿。

这种方法只需一次遍历、O(1) 空间，但实现细节比双遍历法复杂得多。面试中推荐写双遍历法，峰谷法适合进阶理解。

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **双遍历贪心** | O(n) | O(n) | 直观、不易出错 | 需要额外数组 |
| **峰谷法** | O(n) | O(1) | 空间最优 | 实现细节复杂 |
| **排序法** | O(n log n) | O(n) | 思路最自然 | 时间不是最优 |

---

# 第二部分：加油站（LeetCode 134）

## 问题描述

### LeetCode 134. Gas Station（加油站）

> 在一条环形路线上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。
>
> 你有一辆油箱容量无限的汽车，从第 `i` 个加油站开往第 `(i+1)` 个加油站需要消耗 `cost[i]` 升汽油。你从某个加油站出发，油箱初始为空。
>
> 如果你能按顺时针方向绕环路一周，返回**出发加油站的编号**；否则返回 `-1`。
>
> 答案保证**唯一**（如果存在的话）。

**示例：**

```
输入：gas = [1,2,3,4,5], cost = [3,4,5,1,2]
输出：3
解释：
  从3号站出发：
  3→4: 油=4, 消耗=1, 剩余=3
  4→0: 油=3+5=8, 消耗=2, 剩余=6
  0→1: 油=6+1=7, 消耗=3, 剩余=4
  1→2: 油=4+2=6, 消耗=4, 剩余=2
  2→3: 油=2+3=5, 消耗=5, 剩余=0 ✓ 完成一圈！
```

```
输入：gas = [2,3,4], cost = [3,4,3]
输出：-1
解释：无论从哪个站出发，都无法绕一圈。
```

---

## 核心思想

### 关键洞察

**两个事实：**

1. **如果总油量 < 总消耗，一定无解。** 油不够，怎么都走不完一圈。
2. **如果总油量 ≥ 总消耗，一定有解（且唯一）。** 这就是本题最反直觉的地方。

### 为什么"有盈余必有解"？

假设从站点 `i` 出发，走到站点 `j` 时油量变为负数（走不动了）。那么：

> **从 `i` 到 `j` 之间的任何一个站点 `k`（i ≤ k ≤ j）出发，也不可能走到 `j`。**

为什么？因为从 `i` 出发走到 `k` 时油量 ≥ 0（否则走不到 `k`），但从 `k` 到 `j` 还是不够。如果从 `k` 直接出发（油箱从 0 开始），那就更不可能走到 `j` 了。

所以，如果从 `i` 到 `j` 走不通，下次可以直接从 `j+1` 开始尝试——跳过了中间所有站点。

---

## 思路分析

### 解法一：一次遍历贪心（最优解）

维护两个变量：
- `total`：全局油量盈余（总油量 - 总消耗）
- `tank`：从当前候选起点出发到当前站点的油量

```
遍历每个站点 i:
  total += gas[i] - cost[i]    // 更新全局盈余
  tank  += gas[i] - cost[i]    // 更新当前油量

  if tank < 0:                 // 走不动了
    start = i + 1              // 下一个站点作为新候选起点
    tank = 0                   // 重置油量

最终: total >= 0 ? start : -1
```

### 解法二：前缀和 + 最小值

`diff[i] = gas[i] - cost[i]` 表示经过站点 `i` 后的净油量变化。

如果从站点 `s` 出发能走一圈，等价于：所有从 `s` 开始的"环形前缀和"都 ≥ 0。

这等价于：从 `s` 开始的环形前缀和的最小值 ≥ 0。

用前缀和数组 + 环形遍历可以求出，但空间 O(n)。

### 解法三：暴力法（O(n²)）

对每个站点，模拟一圈，检查油量是否始终 ≥ 0。时间 O(n²)，但能帮助理解题意。

---

## 代码实现

### JavaScript 版本

#### 方法一：一次遍历贪心（最优解）

```javascript
/**
 * @param {number[]} gas
 * @param {number[]} cost
 * @return {number}
 */
var canCompleteCircuit = function(gas, cost) {
    let total = 0;   // 全局油量盈余
    let tank = 0;    // 当前候选起点出发的油量
    let start = 0;   // 候选起点

    for (let i = 0; i < gas.length; i++) {
        const diff = gas[i] - cost[i];
        total += diff;
        tank += diff;

        if (tank < 0) {
            // 从 start 到 i 都走不通，i+1 是新的候选起点
            start = i + 1;
            tank = 0;
        }
    }

    return total >= 0 ? start : -1;
};
```

#### 方法二：前缀和法

```javascript
var canCompleteCircuit = function(gas, cost) {
    const n = gas.length;
    const diff = gas.map((g, i) => g - cost[i]);

    // 检查总盈余
    const total = diff.reduce((s, d) => s + d, 0);
    if (total < 0) return -1;

    // 找环形前缀和的最小值出现的位置
    let minPrefix = Infinity;
    let minIdx = 0;
    let prefix = 0;

    for (let i = 0; i < n; i++) {
        prefix += diff[i];
        if (prefix < minPrefix) {
            minPrefix = prefix;
            minIdx = i;
        }
    }

    // 最小前缀和之后的下一个位置就是起点
    // （从那里开始，环形前缀和不会再跌破0）
    return (minIdx + 1) % n;
};
```

#### 方法三：暴力法

```javascript
var canCompleteCircuit = function(gas, cost) {
    const n = gas.length;

    for (let start = 0; start < n; start++) {
        let tank = 0;
        let valid = true;

        for (let step = 0; step < n; step++) {
            const i = (start + step) % n;
            tank += gas[i] - cost[i];
            if (tank < 0) {
                valid = false;
                break;
            }
        }

        if (valid) return start;
    }

    return -1;
};
```

### Python 版本

#### 方法一：一次遍历贪心（最优解）

```python
def can_complete_circuit(gas: list[int], cost: list[int]) -> int:
    """方法一：一次遍历贪心"""
    total = 0   # 全局油量盈余
    tank = 0    # 当前候选起点出发的油量
    start = 0   # 候选起点

    for i in range(len(gas)):
        diff = gas[i] - cost[i]
        total += diff
        tank += diff

        if tank < 0:
            # 从 start 到 i 都走不通，i+1 是新的候选起点
            start = i + 1
            tank = 0

    return start if total >= 0 else -1
```

#### 方法二：前缀和法

```python
def can_complete_circuit(gas: list[int], cost: list[int]) -> int:
    """方法二：前缀和法"""
    n = len(gas)
    diff = [g - c for g, c in zip(gas, cost)]

    # 检查总盈余
    if sum(diff) < 0:
        return -1

    # 找环形前缀和的最小值出现的位置
    min_prefix = float('inf')
    min_idx = 0
    prefix = 0

    for i in range(n):
        prefix += diff[i]
        if prefix < min_prefix:
            min_prefix = prefix
            min_idx = i

    # 最小前缀和之后的下一个位置就是起点
    return (min_idx + 1) % n
```

#### 方法三：暴力法

```python
def can_complete_circuit(gas: list[int], cost: list[int]) -> int:
    """方法三：暴力法"""
    n = len(gas)

    for start in range(n):
        tank = 0
        valid = True

        for step in range(n):
            i = (start + step) % n
            tank += gas[i] - cost[i]
            if tank < 0:
                valid = False
                break

        if valid:
            return start

    return -1
```

---

### 逐步推演

以 `gas = [1,2,3,4,5], cost = [3,4,5,1,2]` 为例。

#### 一次遍历贪心推演

```
diff = [1-3, 2-4, 3-5, 4-1, 5-2] = [-2, -2, -2, 3, 3]

total=0, tank=0, start=0

i=0: diff=-2
  total = -2, tank = -2
  tank < 0 → start = 1, tank = 0

i=1: diff=-2
  total = -4, tank = -2
  tank < 0 → start = 2, tank = 0

i=2: diff=-2
  total = -6, tank = -2
  tank < 0 → start = 3, tank = 0

i=3: diff=3
  total = -3, tank = 3
  tank ≥ 0 → 保持

i=4: diff=3
  total = 0, tank = 6
  tank ≥ 0 → 保持

total = 0 ≥ 0 → 返回 start = 3 ✅
```

#### 前缀和法推演

```
diff = [-2, -2, -2, 3, 3]

计算前缀和:
  prefix[0] = -2
  prefix[1] = -4
  prefix[2] = -6  ← 最小值！
  prefix[3] = -3
  prefix[4] = 0

最小前缀和在 index=2，起点 = (2+1) % 5 = 3 ✅
```

**为什么前缀和最小值的下一个位置是起点？**

前缀和表示从 0 号站出发走到当前站的累计油量。如果 `prefix[k]` 最小，说明从 0 号站走到 `k` 号站消耗最多。那么从 `k+1` 号站出发，绕过 `k` 号站（最困难的一段），就能保证剩余的路程油量都够用。

#### 验证从 3 号站出发

```
gas = [1,2,3,4,5], cost = [3,4,5,1,2]

从3号站出发:
  3→4: 加油4, 消耗1, tank=3
  4→0: 加油5, 消耗2, tank=6
  0→1: 加油1, 消耗3, tank=4
  1→2: 加油2, 消耗4, tank=2
  2→3: 加油3, 消耗5, tank=0

tank 始终 ≥ 0，成功回到起点 ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **一次遍历贪心** | O(n) | O(1) | 最优解，简洁 | 需要理解"跳过"的证明 |
| **前缀和法** | O(n) | O(n) | 直观的数学解释 | 多了 O(n) 空间 |
| **暴力法** | O(n²) | O(1) | 最易理解 | 效率低 |

---

# 第三部分：两题对比

## 为什么放在一起？

| 维度 | 分发糖果（135） | 加油站（134） |
|------|---------------|-------------|
| **贪心类型** | 双向扫描，逐步修正 | 一次遍历，全局盈余判断 |
| **局部→全局** | 左→右 + 右→左 = 全局满足 | 从候选起点出发的局部油量 + 全局盈余 |
| **核心技巧** | 取 max 保证双向约束 | tank < 0 时跳到下一个 |
| **证明关键** | 两次扫描不冲突（取max保留更大值） | 跳过的区间不可能有起点 |
| **共同点** | 都是贪心，都是 O(n)，都需要先理解"为什么这样做是对的" |

### 贪心的两种模式

```
分发糖果：[约束拆解模式]
  → 复杂约束拆成简单子约束，分别贪心，取max合并
  → 适用场景：多方向约束

加油站：[全局盈余模式]
  → 维护全局变量，遇到局部失败就跳过
  → 适用场景：环形/循环问题，存在"有解判定条件"
```

---

## 优化方向

### 1. 分发糖果的空间优化

双遍历法用 O(n) 空间存糖果数组。峰谷法可以做到 O(1)，但实现更复杂。在面试中，**双遍历法是推荐写法**——简单、正确、好解释。

### 2. 加油站的变体

如果答案不唯一（去掉"答案唯一"的限制），前缀和法可以找出**所有**可行起点。贪心法只能找到一个。

### 3. 从贪心到 DP

这两道题之所以用贪心而不是 DP，是因为它们的局部最优决策具有**无后效性**——做出当前最优选择后，不会影响后续的可行性。如果给加油站加上"某些路段限行"之类的额外约束，贪心就不一定成立了，可能需要 DP 或 BFS。

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **135 分发糖果** | Hard | 双遍历 + 取max | 双遍历贪心 |
| **134 加油站** | Medium | 全局盈余 + 局部跳过 | 一次遍历贪心 |

两道题告诉我们贪心的两种经典套路：

1. **当约束有两个方向时** → 拆成两次遍历，分别满足
2. **当问题有环形结构时** → 维护全局盈余，局部失败就跳过

贪心不是"随便选最大的"，而是**证明了局部最优能推出全局最优之后的高效实现**。
