# 🏆 LeetCode 题解博客

> **多解法 · 手动推演 · 举一反三** — 用最清晰的方式讲透算法题

---

## 📚 专题目录

### 🧱 单调栈系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 84 | [柱状图中最大的矩形](./leetcode-84-largest-rectangle-in-histogram.md) | 🔴 Hard | 单调递增栈、左右边界 | 哨兵法 / 两次遍历 |
| 42 | [接雨水](./leetcode-42-trapping-rain-water.md) | 🔴 Hard | 单调递减栈、双指针 | DP / 单调栈 / 双指针 |

> **关联阅读**：42 和 84 是单调栈的双生子——一个找更高的墙（储水），一个找更矮的柱子（边界）。

### 📈 动态规划系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 1143 | [最长公共子序列](./leetcode-1143-longest-common-subsequence.md) | 🟡 Medium | 二维 DP、序列匹配 | DP / DP+栈回溯 / 滚动数组 |
| 152 | [乘积最大子数组](./leetcode-152-maximum-product-subarray.md) | 🟡 Medium | Kadane 变体、负负得正 | 暴力 / Kadane DP / 空间优化 / 双遍历 |
| 32 | [最长有效括号](./leetcode-32-longest-valid-parentheses.md) | 🔴 Hard | 括号匹配、三种思路 | 栈 / DP / 双指针 |

### 🔙 回溯系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 51/52 | [N 皇后](./leetcode-51-n-queens-backtracking.md) | 🔴 Hard | 回溯模板、对角线剪枝 | 标准回溯 / 位运算优化 |

### 🔗 图论系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 207/210 | [课程表](./leetcode-207-course-schedule.md) | 🟡 Medium | 拓扑排序、环检测 | Kahn(BFS) / DFS |

### 📦 栈的应用

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 394 | [字符串解码](./leetcode-394-decode-string.md) | 🟡 Medium | 嵌套结构、栈保存上下文 | 双栈法 / 递归(DFS) |

### 🔤 字符串系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 6 | [Z 字形变换](./leetcode-6-zigzag-conversion.md) | 🟡 Medium | 周期公式、索引定位 | 暴力模拟 / 逐行追踪 / 数学公式 |
| 12 | [整数转罗马数字](./leetcode-12-integer-to-roman-and-13-roman-to-integer.md) | 🟡 Medium | 贪心、映射表 | 贪心匹配 / 按位处理 |
| 13 | [罗马数字转整数](./leetcode-12-integer-to-roman-and-13-roman-to-integer.md) | 🟢 Easy | 哈希表、相邻比较 | 一次遍历 / 从右到左 |

### 🔑 哈希表系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 49 | [字母异位词分组](./leetcode-49-group-anagrams.md) | 🟡 Medium | 哈希分组、签名函数、排序/计数 | 排序法 / 计数法 / 质数乘积法 |
| 205 | [同构字符串](./leetcode-205-isomorphic-strings-and-290-word-pattern.md) | 🟢 Easy | 双射映射、字符编码 | 双向哈希表 / 标准化编码 / 首次出现索引 |
| 290 | [单词规律](./leetcode-205-isomorphic-strings-and-290-word-pattern.md) | 🟢 Easy | 双射映射、字符→单词 | 双向哈希表 / 标准化编码 / 首次出现索引 |

### 🪟 滑动窗口系列

| # | 题目 | 难度 | 核心考点 | 解法 |
|---|------|------|---------|------|
| 76 | [最小覆盖子串](./leetcode-76-minimum-window-substring.md) | 🔴 Hard | 滑动窗口、哈希表计数、valid 计数器 | 暴力枚举 / 滑动窗口 / 优化滑动窗口 |

---

## 🧩 按算法模式分类

### 「单调栈」—— 找左右边界
- [84. 柱状图中最大的矩形](./leetcode-84-largest-rectangle-in-histogram.md) — 左右第一个更小
- [42. 接雨水](./leetcode-42-trapping-rain-water.md) — 左右第一个更大

### 「双指针」—— 对撞 / 双向扫描
- [42. 接雨水](./leetcode-42-trapping-rain-water.md) — 对撞指针，O(1) 空间
- [32. 最长有效括号](./leetcode-32-longest-valid-parentheses.md) — 双向扫描，O(1) 空间

### 「二维 DP」—— 序列匹配
- [1143. 最长公共子序列](./leetcode-1143-longest-common-subsequence.md) — 二维表格 + 回溯还原序列
- [72. 编辑距离](https://leetcode.com/problems/edit-distance/) (关联)

### 「状态翻转」—— 同时维护最大最小值
- [152. 乘积最大子数组](./leetcode-152-maximum-product-subarray.md) — 乘法的非单调性

### 「回溯模板」—— 做选择 → 递归 → 撤销选择
- [51. N 皇后](./leetcode-51-n-queens-backtracking.md) — 经典回溯模板
- [46. 全排列](https://leetcode.com/problems/permutations/) (关联) | [39. 组合总和](https://leetcode.com/problems/combination-sum/) (关联)

### 「拓扑排序」—— 依赖关系线性化
- [207/210. 课程表](./leetcode-207-course-schedule.md) — BFS(Kahn) / DFS

### 「贪心 + 映射表」—— 固定符号体系转换
- [12/13. 整数与罗马数字互转](./leetcode-12-integer-to-roman-and-13-roman-to-integer.md) — 贪心匹配 + 哈希表映射

### 「滑动窗口」—— 双指针 + 哈希表计数
- [76. 最小覆盖子串](./leetcode-76-minimum-window-substring.md) — right 凑齐覆盖，left 收缩到最短

### 「哈希分组」—— 设计签名 + 分桶
- [49. 字母异位词分组](./leetcode-49-group-anagrams.md) — 排序/计数/质数乘积作为 key

### 「双射映射」—— 双向哈希表检查一一对应
- [205. 同构字符串 & 290. 单词规律](./leetcode-205-isomorphic-strings-and-290-word-pattern.md) — 字符/单词级别的结构模式匹配

### 「周期定位」—— 用公式直接找索引
- [6. Z 字形变换](./leetcode-6-zigzag-conversion.md) — 周期 = 2*numRows-2，逐行按索引取字符

---

## 📈 数据统计

| 指标 | 数值 |
|------|------|
| 总题数 | **14** 道（含变体共 17 题） |
| 🔴 Hard | 5 道 (36%) |
| 🟡 Medium | 6 道 (43%) |
| 🟢 Easy | 3 道 (21%) |
| 覆盖算法专题 | 11 大类 |
| 代码语言 | JavaScript + Python |

---

## 🚀 更新计划

- [x] **Easy 入门题**：罗马数字转整数 ✅ | 二叉树遍历、二分查找、滑动窗口
- [x] **滑动窗口专题**：最小覆盖子串 ✅
- [x] **Python 代码**：每道题都已补充 Python 实现 ✅
- [ ] **专题总结**：单调栈专题、回溯模板专题、区间 DP 专题
- [ ] **图解说明**：为复杂问题添加 ASCII 图解和状态转移图

---

## ✍️ 关于作者

LeetCode 刷题中，致力于用最清晰的方式讲透算法题。

每篇文章都追求：
1. ✅ **多解法覆盖** — 从暴力到最优，思路层层递进
2. ✅ **手动推演** — 每一步的状态变化都写清楚
3. ✅ **举一反三** — 提炼通用思维框架，让你做一道题会一类题

---

> **说明**：本博客源码在 [GitHub](https://github.com/DRB-code-ing) 上，欢迎 Star ⭐ 和 PR！
