---
title: LeetCode 152. 乘积最大子数组
tags: [动态规划, Kadane, 数组]
difficulty: Medium
category: 动态规划
date: 2026-05-27
---

# LeetCode 152. 乘积最大子数组 —— 从最大子数组和到乘积，一个 Kadane 的变体

## 前言

在算法面试中，"最大子数组和"（LeetCode 53）几乎是必考题，它的 Kadane 算法简洁优美——`dp[i] = max(nums[i], dp[i-1] + nums[i])`。但面试官常常会在此基础上升级：**"如果是乘积呢？"**

这便引出了 LeetCode 152 —— **乘积最大子数组**（Maximum Product Subarray）。

这道题和最大子数组和看似只是把"加法"换成了"乘法"，但背后的思维跳跃却大得多：加法是单调的（加负数一定变小），而乘法中**负数遇到负数会翻转成正数**——这个差异带来了全新的挑战，也造就了一道经典的面试题。

本文将覆盖三种解法：**暴力法** → **动态规划（Kadane 变体）** → **优化空间版**，从易到难，彻底吃透乘积最大子数组。

---

## 问题描述

### LeetCode 152. Maximum Product Subarray（乘积最大子数组）

> 给你一个整数数组 `nums`，请你找出数组中乘积最大的**非空连续子数组**（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

**示例：**

```
输入：nums = [2,3,-2,4]
输出：6
解释：子数组 [2,3] 有最大乘积 6。
```

```
输入：nums = [-2,0,-1]
输出：0
解释：结果不能为 2，因为 [-2,-1] 不是连续子数组，中间被 0 隔开了。
```

```
输入：nums = [-2,3,-4]
输出：24
解释：子数组 [-2,3,-4] 的乘积 = (-2) × 3 × (-4) = 24。
                     注意：两个负数相乘变成了正数！
```

---

## 核心思想

### 为什么 Kadane 不能直接搬过来？

回顾最大子数组和的 Kadane 算法：

```
dp_max[i] = max(nums[i], dp_max[i-1] + nums[i])
```

这个公式成立的原因是：**加法是"单调"的**——`dp_max[i-1]` 要么对当前有帮助（是正数），要么没帮助（是负数，不如从当前重新开始）。

但对于乘法，情况完全不同：

```
nums = [-2, 3, -4]
```

如果我们只记录以每个位置结尾的最大乘积：
- `max[0] = -2`
- `max[1] = max(3, -2×3) = 3`  ← 我们"放弃"了 -6
- `max[2] = max(-4, 3×(-4)) = -4`  ← 得到 -4，错了！

但实际上 `(-2) × 3 × (-4) = 24` 才是正确答案。问题出在哪？

**问题在于：一个很小的负数乘积，再乘以一个负数，可能变成很大的正数。**

### 核心公式：同时维护最大值和最小值

解决方案是：**同时记录以当前位置结尾的「最大乘积」和「最小乘积」**。

```
当前位置为 i，值为 nums[i]：

以 i 结尾的最大乘积 = max(nums[i], 上个位置的最大乘积 × nums[i], 上个位置的最小乘积 × nums[i])
以 i 结尾的最小乘积 = min(nums[i], 上个位置的最大乘积 × nums[i], 上个位置的最小乘积 × nums[i])
```

为什么这样有效？

| nums[i] 的符号 | 最大乘积来源 | 最小乘积来源 |
|---------------|------------|------------|
| 正数 | `最大 × 正数` | `最小 × 正数` |
| 负数 | `最小 × 负数`（最小可能是负得最多的） | `最大 × 负数` |
| 0 | 重新开始（乘积归零） | 重新开始 |

关键洞察：**一个负的 nums[i] 会把"最大"变成"最小"，把"最小"变成"最大"——所以我们需要同时知道两者。**

---

## 思路分析

### 解法一：暴力法（O(n²)）

最直接的思路：枚举所有可能的子数组，计算每个子数组的乘积，取最大值。

```
遍历每个起点 i：
    遍历每个终点 j >= i：
        计算 nums[i] 到 nums[j] 的乘积
        更新全局最大值
```

虽然简单，但 O(n²) 在大数据量下不可接受，仅作为理解题意的起点。

### 解法二：动态规划（Kadane 变体）

维护两个 DP 数组：
- `maxDP[i]`：以 `nums[i]` 结尾的子数组的最大乘积
- `minDP[i]`：以 `nums[i]` 结尾的子数组的最小乘积

转移方程：

```
maxDP[i] = max(nums[i], maxDP[i-1] * nums[i], minDP[i-1] * nums[i])
minDP[i] = min(nums[i], maxDP[i-1] * nums[i], minDP[i-1] * nums[i])
```

全局答案 = `max(maxDP[0], maxDP[1], ..., maxDP[n-1])`

```
手动推演：nums = [2, 3, -2, 4]

i=0: nums[0]=2
  maxDP[0] = max(2, -, -) = 2
  minDP[0] = min(2, -, -) = 2
  全局 max = 2

i=1: nums[1]=3
  maxDP[1] = max(3, 2×3=6, 2×3=6) = 6
  minDP[1] = min(3, 2×3=6, 2×3=6) = 3
  全局 max = 6

i=2: nums[2]=-2
  maxDP[2] = max(-2, 6×(-2)=-12, 3×(-2)=-6) = -2
  minDP[2] = min(-2, 6×(-2)=-12, 3×(-2)=-6) = -12
  全局 max = 6

i=3: nums[3]=4
  maxDP[3] = max(4, -2×4=-8, -12×4=-48) = 4
  minDP[3] = min(4, -2×4=-8, -12×4=-48) = -48
  全局 max = 6 ✅
```

### 解法三：优化空间版

观察转移方程，**当前位置的值只依赖上一个位置的值**，所以不需要数组，用两个变量滚动更新即可：

```javascript
prevMax = maxDP[i-1]  →  currMax
prevMin = minDP[i-1]  →  currMin
```

**⚠️ 重要细节**：更新 `currMax` 时用的 `prevMax` 和 `prevMin` 是**上一轮**的值。如果先更新了 `currMax`，再更新 `currMin` 时如果用新的 `currMax` 就错了。所以要么用临时变量保存旧值，要么在遇到负数时交换 `prevMax` 和 `prevMin`。

### 解法四：双遍历法（一种有趣的思路）

另一种巧妙的思路：从左到右累乘，再从右到左累乘，遇到 0 重置。取两趟遍历的最大值。

原理：乘积最大的子数组要么从左端开始延伸，要么从右端开始延伸（因为 0 会断开数组，而负数最多只需要考虑一个）。

代码非常简短，思路也很有趣，但**不如 DP 解法通用**（修改一下就能解决很多变体），本文以 DP 为主线。

---

## 代码实现

### JavaScript 版本

#### 方法一：暴力法（O(n²)）

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var maxProduct = function(nums) {
    let globalMax = -Infinity;

    for (let i = 0; i < nums.length; i++) {
        let product = 1;
        for (let j = i; j < nums.length; j++) {
            product *= nums[j];
            globalMax = Math.max(globalMax, product);
        }
    }

    return globalMax;
};
```

#### 方法二：动态规划（Kadane 变体，推荐）

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var maxProduct = function(nums) {
    const n = nums.length;
    const maxDP = new Array(n);
    const minDP = new Array(n);

    maxDP[0] = nums[0];
    minDP[0] = nums[0];
    let result = nums[0];

    for (let i = 1; i < n; i++) {
        // 当前位置有三种可能：
        // 1. 从当前重新开始：nums[i]
        // 2. 接在上个最大乘积后面：maxDP[i-1] * nums[i]
        // 3. 接在上个最小乘积后面：minDP[i-1] * nums[i]
        maxDP[i] = Math.max(nums[i], maxDP[i-1] * nums[i], minDP[i-1] * nums[i]);
        minDP[i] = Math.min(nums[i], maxDP[i-1] * nums[i], minDP[i-1] * nums[i]);

        result = Math.max(result, maxDP[i]);
    }

    return result;
};
```

```python
def max_product(nums: list[int]) -> int:
    """方法二：动态规划（Kadane 变体）"""
    n = len(nums)
    max_dp = [0] * n
    min_dp = [0] * n

    max_dp[0] = min_dp[0] = result = nums[0]

    for i in range(1, n):
        # 当前位置有三种可能：
        # 1. 从当前重新开始：nums[i]
        # 2. 接在上个最大乘积后面：max_dp[i-1] * nums[i]
        # 3. 接在上个最小乘积后面：min_dp[i-1] * nums[i]
        max_dp[i] = max(nums[i], max_dp[i-1] * nums[i], min_dp[i-1] * nums[i])
        min_dp[i] = min(nums[i], max_dp[i-1] * nums[i], min_dp[i-1] * nums[i])
        result = max(result, max_dp[i])

    return result
```

#### 方法三：优化空间版（最优）

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var maxProduct = function(nums) {
    let prevMax = nums[0];
    let prevMin = nums[0];
    let result = nums[0];

    for (let i = 1; i < nums.length; i++) {
        // 遇到负数时，最大和最小会互换
        if (nums[i] < 0) {
            [prevMax, prevMin] = [prevMin, prevMax];
        }

        // 更新当前的最大和最小
        prevMax = Math.max(nums[i], prevMax * nums[i]);
        prevMin = Math.min(nums[i], prevMin * nums[i]);

        result = Math.max(result, prevMax);
    }

    return result;
};
```

```python
def max_product(nums: list[int]) -> int:
    """方法三：优化空间版（O(1) 空间）"""
    prev_max = prev_min = result = nums[0]

    for i in range(1, len(nums)):
        if nums[i] < 0:
            # 遇到负数时，最大和最小互换
            prev_max, prev_min = prev_min, prev_max

        prev_max = max(nums[i], prev_max * nums[i])
        prev_min = min(nums[i], prev_min * nums[i])
        result = max(result, prev_max)

    return result
```

#### 方法四：双遍历法（巧妙思路）

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var maxProduct = function(nums) {
    let result = -Infinity;
    let product = 1;

    // 从左到右
    for (let i = 0; i < nums.length; i++) {
        product *= nums[i];
        result = Math.max(result, product);
        if (product === 0) product = 1;  // 遇到 0 重置
    }

    // 从右到左
    product = 1;
    for (let i = nums.length - 1; i >= 0; i--) {
        product *= nums[i];
        result = Math.max(result, product);
        if (product === 0) product = 1;
    }

    return result;
};
```

```python
def max_product(nums: list[int]) -> int:
    """方法四：双遍历法"""
    result = float('-inf')
    product = 1

    # 从左到右
    for num in nums:
        product *= num
        result = max(result, product)
        if product == 0:
            product = 1  # 遇到 0 重置

    # 从右到左
    product = 1
    for num in reversed(nums):
        product *= num
        result = max(result, product)
        if product == 0:
            product = 1

    return result
```

### 逐步推演

以 `nums = [2, 3, -2, 4]` 为例，用优化空间版推演：

```
初始：prevMax = 2, prevMin = 2, result = 2

i=1: nums[1]=3 (正数)
  prevMax = max(3, 2×3=6) = 6
  prevMin = min(3, 2×3=6) = 3
  result = max(2, 6) = 6

i=2: nums[2]=-2 (负数 → 交换 prevMax 和 prevMin)
  prevMax = 3, prevMin = 6  ← 交换！
  prevMax = max(-2, 3×(-2)=-6) = -2
  prevMin = min(-2, 6×(-2)=-12) = -12
  result = max(6, -2) = 6

i=3: nums[3]=4 (正数)
  prevMax = max(4, -2×4=-8) = 4
  prevMin = min(4, -12×4=-48) = -48
  result = max(6, 4) = 6 ✅
```

再看一个**全是负数**的例子来说明负负得正：

```
nums = [-2, 3, -4]

初始：prevMax = -2, prevMin = -2, result = -2

i=1: nums[1]=3 (正数)
  prevMax = max(3, -2×3=-6) = 3
  prevMin = min(3, -2×3=-6) = -6
  result = max(-2, 3) = 3

i=2: nums[2]=-4 (负数 → 交换)
  prevMax = -6, prevMin = 3  ← 交换！
  prevMax = max(-4, -6×(-4)=24) = 24   ← 负负得正！
  prevMin = min(-4, 3×(-4)=-12) = -12
  result = max(3, 24) = 24 ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|-----------|-----------|---------|
| 暴力法 | O(n²) | O(1) | 仅用于理解题意 |
| DP（数组版） | O(n) | O(n) | 便于理解和调试 |
| **优化空间版** | **O(n)** | **O(1)** | **最优，推荐面试使用** |
| 双遍历法 | O(n) | O(1) | 思路巧妙，但不够通用 |

---

## 优化方向对比

### 1. 为什么需要同时维护最大和最小？

这是本题和最大子数组和**最核心的区别**：

| 场景 | 加法 | 乘法 |
|------|------|------|
| 正 + 正 = ? | 更大 | 更大 |
| 正 + 负 = ? | 更小（可舍弃） | 更小（但可能被负数翻盘） |
| 负 + 负 = ? | 更小 | 更大（**负负得正**） |
| 遇到负数 | 一定变小 | 可能翻转大小 |

乘法的**非单调性**决定了：我们不能只记录最大值，必须同时记录最小值——因为当前的最小值（负得最多）乘以一个负数可能会变成最大值。

### 2. 空间优化：从数组到滚动变量

```
DP 数组版：       维护 maxDP 和 minDP 两个数组 → O(n) 空间
滚动变量版：      维护 prevMax 和 prevMin 两个变量 → O(1) 空间
遇到负数交换版：  进一步利用乘法性质，代码更简洁 → O(1) 空间
```

滚动变量版的实现需要特别注意更新顺序——这是面试中容易踩的坑。

### 3. 和最大子数组和（LeetCode 53）的对比

| | 最大子数组和（53） | 乘积最大子数组（152） |
|------|-----------------|-------------------|
| 核心操作 | 加法 | 乘法 |
| DP 状态 | 一个变量（最大和） | 两个变量（最大/最小乘积） |
| 负数的效果 | 让和变小 | 可能翻转大小 |
| 遇到 0 | 重置为 0 | 重置为 0 |
| 数据结构 | O(1) 空间 | O(1) 空间 |

**记忆口诀**：加法单调，一个够用；乘法翻转，两个才行。

---

## 举一反三

理解了"同时维护最大最小值"的思路，以下题目都可以用类似的思维解决：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 53. 最大子数组和** | 基础 Kadane，只需一个变量 |
| **LeetCode 918. 环形子数组的最大和** | 环形数组，分两种情况讨论 |
| **LeetCode 1567. 乘积为正数的最长子数组长度** | 同时维护正负长度 |
| **LeetCode 713. 乘积小于 K 的子数组** | 滑动窗口，不是 DP |
| **LeetCode 152. 乘积最大子数组** | 本题 |

它们的共同模式：**当状态转移需要依赖上一步的多个"极端值"时，同时维护最大和最小两个状态。**

```
// 处理"有正有负的乘法/加法"的通用框架
function solve(nums) {
    let prevMax = nums[0];  // 上一步的最大值
    let prevMin = nums[0];  // 上一步的最小值
    let result = nums[0];

    for (let i = 1; i < nums.length; i++) {
        // 状态转移：当前值 = f(上步最大, 上步最小, 当前元素)
        const currMax = Math.max(nums[i], prevMax * nums[i], prevMin * nums[i]);
        const currMin = Math.min(nums[i], prevMax * nums[i], prevMin * nums[i]);

        [prevMax, prevMin] = [currMax, currMin];
        result = Math.max(result, currMax);
    }

    return result;
}
```

---

## 总结

乘积最大子数组这道题的精髓在于**从加法的单调思维跃迁到乘法的翻转思维**：

1. **核心洞察**：乘法不具备单调性，负数能翻转大小，所以必须同时维护「最大乘积」和「最小乘积」
2. **三种解法**：
   - 暴力法 O(n²)：枚举所有子数组
   - **Kadane 变体 DP**：同时维护 max 和 min，O(n) 时间
   - **优化空间版（推荐）**：滚动变量，O(1) 空间
3. **思维升级**：遇到负数时交换最大和最小——这个技巧简洁又优雅

面试中建议这样展示：先用暴力法讲清楚问题，再用 Kadane 变体写出 DP 版本，最后优化到 O(1) 空间的滚动变量版。如果能说出"遇到负数交换最大最小"这个技巧，就是完美的面试表现。

最后记住一句话：**加法和乘法最大的区别不是运算符号，而是负数能不能带来翻盘的机会。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 42. 接雨水](./leetcode-42-trapping-rain-water.md) | [LeetCode 84. 柱状图中最大的矩形](./leetcode-84-largest-rectangle-in-histogram.md) | 后续会继续更新动态规划系列，关注不迷路。
