---
title: LeetCode 207/210. 课程表
tags: [图论, 拓扑排序, BFS, DFS, 环检测]
difficulty: Medium
category: 图论
date: 2026-05-26
---

# LeetCode 207/210. 课程表 —— 拓扑排序从入门到精通

## 前言

图的遍历不仅仅只有 DFS 和 BFS，还有一种专门处理"依赖关系"的遍历方式——**拓扑排序**。

日常生活中到处都是"先做 A，才能做 B"的场景：先修完微积分才能选高级课程、先安装依赖才能编译项目、先完成子任务才能合并 PR。课程表问题（LeetCode 207/210）正是这类依赖关系的经典抽象，也是面试中考察图论基础的高频题。

本文将覆盖两种解法：**BFS（Kahn 算法）** 和 **DFS（后序遍历法）**，从原理到代码，让你彻底掌握拓扑排序。

---

## 问题描述

### LeetCode 207. Course Schedule（课程表 I）

> 你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1`。
>
> 在选修某些课程之前需要一些先修课程。先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]`，表示如果要学习课程 `ai` 则**必须**先学习课程 `bi`。
>
> 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0`，你需要先完成课程 `1`。
>
> 请你判断是否可能完成所有课程的学习？如果可以，返回 `true`；否则返回 `false`。

**示例：**

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前你需要先完成课程 0。这是可能的。

输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：总共有 2 门课程。学习课程 1 之前你需要先完成课程 0，
     并且学习课程 0 之前你需要先完成课程 1。这是不可能的（循环依赖）。
```

### LeetCode 210. Course Schedule II（课程表 II）

> 在 207 的基础上，**返回你为了学完所有课程所安排的学习顺序**。如果有多个正确顺序，返回任意一个即可。如果不可能完成所有课程，返回空数组。

**示例：**

```
输入：numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
输出：[0,1,2,3] 或 [0,2,1,3]
解释：一个可能的顺序是：先学 0，然后学 1 和 2（可互换），最后学 3。
```

---

## 核心思想

### 抽象成图

每门课程是一个**节点**，先修关系 `[ai, bi]`（学 ai 前必须先学 bi）是一条**有向边** `bi → ai`。

```
prerequisites = [[1,0],[2,0],[3,1],[3,2]]

图结构：
  0 ──→ 1 ──→ 3
   └─→ 2 ──→ 3
```

问题就转化为：**给定一个有向图，判断是否存在拓扑排序（是否存在环）。**

### 什么是拓扑排序？

**拓扑排序**是有向无环图（DAG）所有顶点的一个线性序列，使得对于每条有向边 `u → v`，`u` 在序列中都出现在 `v` 之前。

拓扑排序存在的条件：**图必须是有向无环图（DAG）**。如果图中有环，环上的课程相互依赖，不可能排出一个合法的学习顺序。

### 入度与出度

在拓扑排序中，"入度"是最关键的概念：

- **入度（in-degree）**：指向该节点的边数，即"有多少门课依赖我"
- **出度（out-degree）**：从该节点出发的边数，即"我依赖多少门课"

入度为 0 的节点意味着**没有前置依赖**，可以立即学习。

---

## 思路分析

### 解法一：BFS（Kahn 算法）

Kahn 算法的核心是**"从入度为 0 的节点开始，逐层剥除"**。

步骤：

1. **建图**：用邻接表 `graph` 存储所有边，同时统计每个节点的入度 `inDegree`
2. **初始化队列**：将所有入度为 0 的节点入队
3. **逐层处理**：
   - 出队一个节点，将其加入结果集
   - 遍历该节点的所有邻接节点，将它们的入度减 1
   - 如果某个邻接节点的入度变为 0，将其入队
4. **判断**：处理完所有节点后，如果结果集中的节点数等于课程总数，说明可以完成；否则说明存在环

```
可视化过程：

numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]

入度: [0:0, 1:1, 2:1, 3:2]
邻接表: {0→[1,2], 1→[3], 2→[3]}

初始队列: [0]  ← 入度为 0 的节点

出队 0 → 结果: [0]
        邻接点 1 入度减 1: 1-1=0 → 入队
        邻接点 2 入度减 1: 1-1=0 → 入队
队列: [1, 2]

出队 1 → 结果: [0, 1]
        邻接点 3 入度减 1: 2-1=1
队列: [2]

出队 2 → 结果: [0, 1, 2]
        邻接点 3 入度减 1: 1-1=0 → 入队
队列: [3]

出队 3 → 结果: [0, 1, 2, 3]
队列: []

结果集大小 = 4 = numCourses → 无环，返回 true ✅
```

### 解法二：DFS（后序遍历法）

DFS 判断环的思路：对每个节点进行深度优先遍历，如果从当前节点出发遇到了**正在当前路径中**的节点，说明存在环。

使用三种状态标记节点：
- `0`（未访问）：从未被 DFS 访问过
- `1`（访问中）：在当前的 DFS 路径上（还在递归栈中）
- `2`（已访问）：DFS 已经完成，该节点之后的子图无环

```
可视化 DFS 过程：

numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]

邻接表: {0→[1,2], 1→[3], 2→[3]}

从 0 开始 DFS:
  0 → [1, 2]
  ├── 0 → 1 → [3]
  │       └── 1 → 3（无邻接点）→ 标记 3 为 2
  │   标记 1 为 2
  └── 0 → 2 → [3]（3 已标记 2，跳过）
  标记 0 为 2

所有节点已访问，无环 → 返回 true ✅
```

关键区别：
- **Kahn 算法**：BFS 思路，从入度为 0 的节点开始"剥离"，是**广度的、主动的**
- **DFS 法**：深度遍历，检测后向边（回边），是**深度的、被动的**

---

## 代码实现

### JavaScript 版本（BFS Kahn 算法）

```javascript
/**
 * @param {number} numCourses
 * @param {number[][]} prerequisites
 * @return {boolean}
 */
var canFinish = function(numCourses, prerequisites) {
    // 1. 建图 + 统计入度
    const graph = Array.from({ length: numCourses }, () => []);
    const inDegree = new Array(numCourses).fill(0);

    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);  // prereq → course
        inDegree[course]++;           // course 的入度 +1
    }

    // 2. 初始化队列：所有入度为 0 的节点
    const queue = [];
    for (let i = 0; i < numCourses; i++) {
        if (inDegree[i] === 0) queue.push(i);
    }

    // 3. BFS 拓扑排序
    let visited = 0;
    while (queue.length > 0) {
        const node = queue.shift();
        visited++;

        for (const neighbor of graph[node]) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] === 0) {
                queue.push(neighbor);
            }
        }
    }

    // 4. 如果访问了所有节点 → 无环
    return visited === numCourses;
};
```

```python
def can_finish(num_courses: int, prerequisites: list[list[int]]) -> bool:
    """BFS（Kahn 算法）判断是否有环"""
    # 1. 建图 + 统计入度
    graph = [[] for _ in range(num_courses)]
    in_degree = [0] * num_courses

    for course, prereq in prerequisites:
        graph[prereq].append(course)  # prereq → course
        in_degree[course] += 1        # course 的入度 +1

    # 2. 初始化队列：所有入度为 0 的节点
    queue = [i for i in range(num_courses) if in_degree[i] == 0]

    # 3. BFS 拓扑排序
    visited = 0
    while queue:
        node = queue.pop(0)
        visited += 1

        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    # 4. 如果访问了所有节点 → 无环
    return visited == num_courses
```

### 返回学习顺序（LeetCode 210）

只需要在 207 的基础上，把访问顺序记录下来：

```javascript
/**
 * @param {number} numCourses
 * @param {number[][]} prerequisites
 * @return {number[]}
 */
var findOrder = function(numCourses, prerequisites) {
    const graph = Array.from({ length: numCourses }, () => []);
    const inDegree = new Array(numCourses).fill(0);

    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
        inDegree[course]++;
    }

    const queue = [];
    for (let i = 0; i < numCourses; i++) {
        if (inDegree[i] === 0) queue.push(i);
    }

    const order = [];
    while (queue.length > 0) {
        const node = queue.shift();
        order.push(node);

        for (const neighbor of graph[node]) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] === 0) {
                queue.push(neighbor);
            }
        }
    }

    return order.length === numCourses ? order : [];
};
```

```python
def find_order(num_courses: int, prerequisites: list[list[int]]) -> list[int]:
    """BFS（Kahn 算法）返回学习顺序"""
    graph = [[] for _ in range(num_courses)]
    in_degree = [0] * num_courses

    for course, prereq in prerequisites:
        graph[prereq].append(course)
        in_degree[course] += 1

    queue = [i for i in range(num_courses) if in_degree[i] == 0]
    order = []

    while queue:
        node = queue.pop(0)
        order.append(node)

        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return order if len(order) == num_courses else []
```

### DFS 解法（判断环）

```javascript
/**
 * @param {number} numCourses
 * @param {number[][]} prerequisites
 * @return {boolean}
 */
var canFinish = function(numCourses, prerequisites) {
    const graph = Array.from({ length: numCourses }, () => []);
    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
    }

    // 0 = 未访问, 1 = 访问中（当前路径上）, 2 = 已访问（安全）
    const visited = new Array(numCourses).fill(0);

    function dfs(node) {
        if (visited[node] === 1) return false;  // 发现环
        if (visited[node] === 2) return true;   // 已经检查过

        visited[node] = 1;  // 标记为当前路径中
        for (const neighbor of graph[node]) {
            if (!dfs(neighbor)) return false;
        }
        visited[node] = 2;  // 标记为已处理完毕
        return true;
    }

    for (let i = 0; i < numCourses; i++) {
        if (visited[i] === 0 && !dfs(i)) return false;
    }

    return true;
};
```

```python
def can_finish_dfs(num_courses: int, prerequisites: list[list[int]]) -> bool:
    """DFS 判断是否有环"""
    graph = [[] for _ in range(num_courses)]
    for course, prereq in prerequisites:
        graph[prereq].append(course)

    # 0 = 未访问, 1 = 访问中, 2 = 已访问
    visited = [0] * num_courses

    def dfs(node: int) -> bool:
        if visited[node] == 1:
            return False  # 发现环
        if visited[node] == 2:
            return True   # 已经检查过

        visited[node] = 1  # 标记为当前路径中
        for neighbor in graph[node]:
            if not dfs(neighbor):
                return False
        visited[node] = 2  # 标记为已处理完毕
        return True

    for i in range(num_courses):
        if visited[i] == 0 and not dfs(i):
            return False

    return True
```

### DFS 解法（返回学习顺序）

```javascript
/**
 * @param {number} numCourses
 * @param {number[][]} prerequisites
 * @return {number[]}
 */
var findOrder = function(numCourses, prerequisites) {
    const graph = Array.from({ length: numCourses }, () => []);
    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
    }

    const visited = new Array(numCourses).fill(0);  // 0:未访问 1:访问中 2:已访问
    const order = [];

    function dfs(node) {
        if (visited[node] === 1) return false;
        if (visited[node] === 2) return true;

        visited[node] = 1;
        for (const neighbor of graph[node]) {
            if (!dfs(neighbor)) return false;
        }
        visited[node] = 2;
        order.push(node);  // 后序加入：先完成依赖，再加入自己
        return true;
    }

    for (let i = 0; i < numCourses; i++) {
        if (visited[i] === 0 && !dfs(i)) return [];
    }

    return order.reverse();  // 需要反转：先修课在前面
};
```

```python
def find_order_dfs(num_courses: int, prerequisites: list[list[int]]) -> list[int]:
    """DFS 返回学习顺序"""
    graph = [[] for _ in range(num_courses)]
    for course, prereq in prerequisites:
        graph[prereq].append(course)

    visited = [0] * num_courses  # 0:未访问 1:访问中 2:已访问
    order = []

    def dfs(node: int) -> bool:
        if visited[node] == 1:
            return False
        if visited[node] == 2:
            return True

        visited[node] = 1
        for neighbor in graph[node]:
            if not dfs(neighbor):
                return False
        visited[node] = 2
        order.append(node)  # 后序加入：先完成依赖，再加入自己
        return True

    for i in range(num_courses):
        if visited[i] == 0 and not dfs(i):
            return []

    return order[::-1]  # 需要反转：先修课在前面
```

> **注意**：DFS 版本在返回学习顺序时，使用的是**后序遍历**——先递归处理所有邻接节点，再将当前节点加入结果。这样最先加入的是最深层的课程（依赖链末端的），所以最后需要将结果反转。

---

## 复杂度分析

### Kahn 算法（BFS）

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | O(V + E) | V = 课程数，E = 先修关系数。建图 O(E)，每个节点入出队一次 O(V)，每条边处理一次 O(E) |
| 空间复杂度 | O(V + E) | 邻接表 O(E)，入度数组 O(V)，队列 O(V) |

### DFS 法

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 | O(V + E) | 每条边和每个节点都只访问一次 |
| 空间复杂度 | O(V + E) | 邻接表 O(E)，visited 数组 O(V)，递归栈最深 O(V) |

---

## 优化方向

### 1. 用 Set/Map 代替数组（当课程数很大但先修关系稀疏时）

如果课程数量巨大（如 10^5 门）但先修关系很少，用数组初始化 `graph` 可能浪费空间。可以用 Map 只存储有出边的节点：

```javascript
var canFinish = function(numCourses, prerequisites) {
    const graph = new Map();
    const inDegree = new Array(numCourses).fill(0);

    for (const [course, prereq] of prerequisites) {
        if (!graph.has(prereq)) graph.set(prereq, []);
        graph.get(prereq).push(course);
        inDegree[course]++;
    }

    // ... 后续 BFS 同，但访问邻接时注意判空
};
```

### 2. 用真正的队列（而非 shift()）

JavaScript 的 `shift()` 是 O(n) 操作。在面试中可以指出这一点，并说明可以用**头尾指针模拟队列**来优化到 O(1) 出队：

```javascript
// 用头尾指针模拟 O(1) 出队
const queue = new Array(numCourses);
let head = 0, tail = 0;

// 入队: queue[tail++] = node
// 出队: const node = queue[head++]

// 队列非空: head < tail
```

但实际面试中直接用 `shift()` 也完全可接受，因为拓扑排序的 BFS 队列长度最大为 V，V 通常不会特别大。主动指出这个优化点反而是加分项。

### 3. 检测环的其他方式

对于 207（只判断是否有环），还可以用**并查集**：

```javascript
// 思路：如果一条边的两个端点已经在同一集合中，说明形成了环
// 但并查集处理有向图需要特殊处理，不如 Kahn 直观
```

不过并查集更适合**无向图**的环检测，有向图还是推荐 Kahn 或 DFS。

---

## 举一反三

拓扑排序的思维框架是**"依赖关系 + 线性化"**。以下题目都可以用相同的思路解决：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 207. 课程表 I** | 判断有向图是否有环（拓扑排序是否存在） |
| **LeetCode 210. 课程表 II** | 返回任意一个拓扑排序序列 |
| **LeetCode 269. 火星词典** | 根据单词顺序推断字符的拓扑排序 |
| **LeetCode 329. 矩阵中的最长递增路径** | 拓扑排序 + DP，从值小的指向值大的 |
| **LeetCode 444. 序列重建** | 判断拓扑排序是否唯一 |
| **LeetCode 1203. 项目管理** | 两级拓扑排序（项目内 + 项目间） |
| **LeetCode 802. 找到最终的安全状态** | 反向图 + 拓扑排序（从出度为 0 开始） |

它们的共同模式：

```
function topologicalSort(n, edges) {
    // 1. 建图 + 统计入度
    // 2. 将入度为 0 的节点入队
    // 3. BFS 剥洋葱：出队 → 减少邻接入度 → 入度为 0 入队
    // 4. 判断访问节点数是否等于总节点数
}
```

### 进阶思考

LeetCode 1203（项目管理）是一个很好的进阶练习：它引入了**两级拓扑排序**——先对项目组排序，再对组内的课程排序。这种"先分组再排序"的思想在实际工程中也很常见，比如构建系统中的模块依赖管理。

---

## 总结

课程表系列题目的精髓在于三点：

1. **模型转换**：识别出"先修依赖"就是有向图，问题等价于判断是否有环 / 求拓扑排序
2. **两种解法**：
   - **Kahn 算法（BFS）**：从入度为 0 的节点开始剥离，直观易懂，适合面试推荐
   - **DFS 法**：三种状态标记（0/1/2），后序遍历检测后向边，空间上可能更优
3. **拓扑排序模板**：建图 → 统计入度 → 队列处理 → 判断数量——一套组合拳打通所有依赖排序问题

面试中遇到"先做 A 才能做 B"的题目，第一反应就应该是拓扑排序。建议按以下策略展示：

- 先和白板说清楚"把课程和先修关系建模成图"
- 用 Kahn 算法写出 BFS 版本，逻辑清晰不易出错
- 有时间再补充 DFS 版本，展示你对两种思路的掌握
- 如果能说出"课程表 II 只需要在 I 的基础上记录出队顺序"和"用头尾指针代替 shift 优化队列"，就是完美的面试表现

建议在纸上手动模拟一遍 `4` 门课程 `[[1,0],[2,0],[3,1],[3,2]]` 的 BFS 过程——把每一步的 `inDegree`、`queue`、`order` 写下来。纸上跑完一遍，远比看十遍代码更管用。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 394. 字符串解码](./leetcode-394-decode-string.md) | [LeetCode 84. 柱状图中最大的矩形](./leetcode-84-largest-rectangle-in-histogram.md) | 后续会继续更新图论系列，关注不迷路。
