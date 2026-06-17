---
title: LeetCode 130. 被围绕的区域
tags: [矩阵, DFS, BFS, 并查集, 逆向思维]
difficulty: Medium
category: 矩阵
date: 2026-06-17
---

# LeetCode 130. 被围绕的区域 —— 逆向思维：从边界反推

## 前言

这道题有一个非常经典的思维陷阱：如果你试图从内部找"被围绕的区域"，你会发现很难判断一个 `O` 是否真的被 `X` 完全包围——因为你不知道它是否通过某条路径延伸到了边界。

一旦你换一个角度，从**边界**出发，问题就豁然开朗：

> **任何没有被 `X` 包围的 `O`，一定直接或间接与边界上的 `O` 相连。反过来，与边界相连的 `O` 一定不会被包围。**

所以，与其从内部找被包围的 `O`，不如：**先从边界出发，标记所有"安全的 `O`"（与边界相连的），再把剩下的 `O` 统统变成 `X`。**

这种"反过来想"的思维在矩阵问题中非常常见——遇到"包围""封闭区域"等关键词时，优先考虑从边界反向推导。

本文将覆盖三种解法：**DFS** → **BFS** → **并查集**，带你彻底掌握"边界逆向思维"。

---

## 问题描述

### LeetCode 130. Surrounded Regions（被围绕的区域）

> 给你一个 `m x n` 的矩阵 `board`，由若干字符 `'X'` 和 `'O'` 组成。
>
> **捕获所有被围绕的区域**：
> - 将与被围绕区域中的 `'O'` 全部替换为 `'X'`
> - 如果一个 `'O'` 在矩阵的边界上，或者与边界上的 `'O'` 相连，那么这个 `'O'` **不会被替换**。
>
> 连接方式：水平或垂直方向相邻（即上下左右，不包括对角线）。

**示例 1：**

```
输入：board = [["X","X","X","X"],
               ["X","O","O","X"],
               ["X","X","O","X"],
               ["X","O","X","X"]]

输出：       [["X","X","X","X"],
               ["X","X","X","X"],
               ["X","X","X","X"],
               ["X","O","X","X"]]

解释：被围绕的区域如上所示。底部的 'O' 在边界上，不会被替换。
      其余三个 'O' 被 'X' 包围，替换为 'X'。
```

**示例 2：**

```
输入：board = [["X"]]
输出：[["X"]]
```

**提示：**
- `m == board.length`
- `n == board[i].length`
- `1 <= m, n <= 200`
- `board[i][j]` 为 `'X'` 或 `'O'`

---

## 核心思想

### 问题的本质：从"找被包围的"变成"找不被包围的"

如果正向思考——遍历每个 `O` 判断它是否被包围——你需要检查它是否被 `X` 完全封住。但"完全封住"意味着什么？意味着从这个 `O` 出发，**无论怎么走都走不到边界**。

这等价于：**如果一个 `O` 能走到边界，那它就不被包围。**

于是思路反转：

```
原问题：  找出所有被 X 包围的 O → 替换为 X
反转后：  找出所有不被 X 包围的 O（与边界相连的 O）→ 标记为安全
         剩余的 O 就是被包围的 → 替换为 X
         恢复安全标记 → 变回 O
```

### 图解

```
初始矩阵：

    X   X   X   X
    X   O   O   X
    X   X   O   X
    X   O   X   X

第一步：从四条边界出发，找到所有与边界相连的 O（标记为 #）

    边界上的 O：board[3][1] = O → 标记为 #

    从 (3,1) 向四周扩散：
      (2,1) = X → 不能走
      (3,0) = X → 不能走
      (3,2) = X → 不能走
    扩散结束。只有 board[3][1] 被标记为 #。

第二步：遍历整个矩阵
    中间的 O 没有被标记 → 被包围 → 替换为 X
    # → 恢复为 O

    X   X   X   X          X   X   X   X
    X   O   O   X    →     X   X   X   X
    X   X   O   X          X   X   X   X
    X   #   X   X          X   O   X   X
```

关键判决：

```
从边界 O 出发，DFS/BFS 能到达的所有 O = 安全 O（不被包围）
其余 O = 被包围 O → 替换为 X
```

### 为什么这个思路正确？

- **充分性**：如果一个 `O` 与边界相连，那么存在一条从边界到这个 `O` 的路径，这条路径上的 `X` 无法完全包围它 → 它不会被替换。
- **必要性**：如果一个 `O` 不被包围，那么一定存在一条从它到边界的路径（否则所有方向都被 `X` 封死，就是被包围）→ 它与边界相连。

> 💡 所以"不被包围 ⇔ 与边界相连"，两者等价。从边界 BFS/DFS 找到所有相连的 `O`，剩下的就是被包围的。

---

## 思路分析

### 解法一：DFS（深度优先搜索）⭐ 推荐

**核心思路**：从四条边界的每个 `O` 出发，DFS 标记所有能到达的 `O` 为临时标记（如 `#`），第二遍遍历时：`O` → `X`，`#` → `O`。

```
算法框架（DFS）：

1. 特殊情况：矩阵为空或大小 ≤ 2×2 → 不可能有内部区域，直接返回

2. 第一遍：从边界出发 DFS 标记安全 O
   for 每条边界（上、下、左、右）：
     for 边界上的每个格子：
       if 格子 == 'O'：
         dfs(格子)  // 标记为 '#'

   dfs(r, c):
     if 越界 or board[r][c] != 'O': return
     board[r][c] = '#'
     dfs(r-1, c)  // 上
     dfs(r+1, c)  // 下
     dfs(r, c-1)  // 左
     dfs(r, c+1)  // 右

3. 第二遍：遍历整个矩阵做替换
   for 每个格子：
     if 'O' → 'X'   （被包围的）
     if '#' → 'O'   （恢复安全的）
```

### 解法二：BFS（广度优先搜索）

**核心思路**：和 DFS 一样，只是把递归换成队列。BFS 的优势是不会有递归栈溢出的风险（虽然 n ≤ 200 时 DFS 也完全安全）。

```
算法框架（BFS）：

1. 初始化队列 queue = []

2. 遍历四条边界，把边界上的 'O' 入队并标记为 '#'

3. while queue 非空：
    取出队首 (r, c)
    遍历四个方向 (nr, nc)：
      if 未越界 and board[nr][nc] == 'O'：
        board[nr][nc] = '#'
        queue.push((nr, nc))

4. 第二遍遍历，做替换：'O' → 'X', '#' → 'O'
```

### 解法三：并查集（Union-Find）

**核心思路**：将矩阵中所有格子映射到并查集中。引入一个**虚拟节点 `dummy`**，将所有边界上的 `O` 和与它们相连的 `O` 都与 `dummy` 连通。最后遍历所有 `O`：与 `dummy` 不连通的 → 被包围 → 替换为 `X`。

```
算法框架（并查集）：

1. 创建并查集，大小为 m×n + 1（多一个虚拟节点 dummy = m×n）

2. 遍历矩阵：
   - 如果当前是 'O'：
     - 如果在边界上 → union(当前, dummy)
     - 如果在内部 → 检查四个邻居，邻居也是 'O' 则 union(当前, 邻居)

3. 再次遍历矩阵：
   - 如果是 'O' 且 find(当前) != find(dummy) → 替换为 'X'
```

> ⚠️ 并查集在本问题上不是最优解（代码更长，常数更大），但它提供了一个**通用的连通性判断框架**，适合触类旁通。

### 为什么 DFS/BFS 是最优方案？

| 方法 | 时间 | 空间 | 推荐度 |
|------|------|------|--------|
| **DFS** | O(m×n) | O(m×n)（递归栈） | ⭐⭐⭐ 最简洁 |
| **BFS** | O(m×n) | O(m×n)（队列） | ⭐⭐⭐ 无栈溢出风险 |
| **并查集** | O(m×n × α(n)) | O(m×n) | ⭐⭐ 通用性强但代码长 |

DFS 代码最短，BFS 更安全（无递归栈），两种都值得掌握。面试中先讲 DFS/BFS，最后提一句并查集的思路，就是完美的表现了。

---

## 代码实现

### JavaScript 版本

#### 方法一：DFS

```javascript
/**
 * @param {character[][]} board
 * @return {void} Do not return anything, modify board in-place instead.
 */
var solve = function(board) {
    const m = board.length;
    const n = board[0].length;
    if (m <= 2 || n <= 2) return;  // 边界矩阵没有内部格子

    // DFS：从 (r, c) 出发，标记所有相连的 'O' 为 '#'
    function dfs(r, c) {
        if (r < 0 || r >= m || c < 0 || c >= n || board[r][c] !== 'O') {
            return;
        }
        board[r][c] = '#';
        dfs(r - 1, c);  // 上
        dfs(r + 1, c);  // 下
        dfs(r, c - 1);  // 左
        dfs(r, c + 1);  // 右
    }

    // 第一遍：从四条边界出发 DFS
    for (let i = 0; i < m; i++) {
        if (board[i][0] === 'O')     dfs(i, 0);       // 左边界
        if (board[i][n - 1] === 'O') dfs(i, n - 1);   // 右边界
    }
    for (let j = 0; j < n; j++) {
        if (board[0][j] === 'O')     dfs(0, j);       // 上边界
        if (board[m - 1][j] === 'O') dfs(m - 1, j);   // 下边界
    }

    // 第二遍：'O' → 'X', '#' → 'O'
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (board[i][j] === 'O') {
                board[i][j] = 'X';
            } else if (board[i][j] === '#') {
                board[i][j] = 'O';
            }
        }
    }
};
```

#### 方法二：BFS

```javascript
/**
 * @param {character[][]} board
 * @return {void}
 */
var solve = function(board) {
    const m = board.length;
    const n = board[0].length;
    if (m <= 2 || n <= 2) return;

    const dirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];
    const queue = [];

    // 第一遍：将边界上的 'O' 入队并标记为 '#'
    for (let i = 0; i < m; i++) {
        if (board[i][0] === 'O') {
            board[i][0] = '#';
            queue.push([i, 0]);
        }
        if (board[i][n - 1] === 'O') {
            board[i][n - 1] = '#';
            queue.push([i, n - 1]);
        }
    }
    for (let j = 0; j < n; j++) {
        if (board[0][j] === 'O') {
            board[0][j] = '#';
            queue.push([0, j]);
        }
        if (board[m - 1][j] === 'O') {
            board[m - 1][j] = '#';
            queue.push([m - 1, j]);
        }
    }

    // BFS 扩散
    while (queue.length > 0) {
        const [r, c] = queue.shift();
        for (const [dr, dc] of dirs) {
            const nr = r + dr;
            const nc = c + dc;
            if (nr >= 0 && nr < m && nc >= 0 && nc < n && board[nr][nc] === 'O') {
                board[nr][nc] = '#';
                queue.push([nr, nc]);
            }
        }
    }

    // 第二遍：替换
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (board[i][j] === 'O') {
                board[i][j] = 'X';
            } else if (board[i][j] === '#') {
                board[i][j] = 'O';
            }
        }
    }
};
```

#### 方法三：并查集

```javascript
/**
 * @param {character[][]} board
 * @return {void}
 */
var solve = function(board) {
    const m = board.length;
    const n = board[0].length;
    if (m <= 2 || n <= 2) return;

    const dummy = m * n;  // 虚拟节点代表"边界连通分量"
    const parent = Array.from({ length: m * n + 1 }, (_, i) => i);

    function find(x) {
        if (parent[x] !== x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    function union(x, y) {
        parent[find(x)] = find(y);
    }

    function isConnected(x, y) {
        return find(x) === find(y);
    }

    const dirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];

    // 遍历所有格子，建立连通关系
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (board[i][j] !== 'O') continue;

            const idx = i * n + j;

            // 边界上的 O 与 dummy 连通
            if (i === 0 || i === m - 1 || j === 0 || j === n - 1) {
                union(idx, dummy);
            }

            // 与相邻的 O 连通
            for (const [dr, dc] of dirs) {
                const ni = i + dr;
                const nj = j + dc;
                if (ni >= 0 && ni < m && nj >= 0 && nj < n && board[ni][nj] === 'O') {
                    union(idx, ni * n + nj);
                }
            }
        }
    }

    // 第二遍：与 dummy 不连通的 O → X
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (board[i][j] === 'O' && !isConnected(i * n + j, dummy)) {
                board[i][j] = 'X';
            }
        }
    }
};
```

### Python 版本

#### 方法一：DFS

```python
class Solution:
    def solve(self, board: list[list[str]]) -> None:
        """
        Do not return anything, modify board in-place instead.
        """
        m, n = len(board), len(board[0])
        if m <= 2 or n <= 2:
            return

        def dfs(r: int, c: int) -> None:
            if r < 0 or r >= m or c < 0 or c >= n or board[r][c] != 'O':
                return
            board[r][c] = '#'
            dfs(r - 1, c)  # 上
            dfs(r + 1, c)  # 下
            dfs(r, c - 1)  # 左
            dfs(r, c + 1)  # 右

        # 第一遍：从四条边界出发 DFS
        for i in range(m):
            if board[i][0] == 'O':
                dfs(i, 0)
            if board[i][n - 1] == 'O':
                dfs(i, n - 1)
        for j in range(n):
            if board[0][j] == 'O':
                dfs(0, j)
            if board[m - 1][j] == 'O':
                dfs(m - 1, j)

        # 第二遍：'O' → 'X', '#' → 'O'
        for i in range(m):
            for j in range(n):
                if board[i][j] == 'O':
                    board[i][j] = 'X'
                elif board[i][j] == '#':
                    board[i][j] = 'O'
```

#### 方法二：BFS

```python
from collections import deque

class Solution:
    def solve(self, board: list[list[str]]) -> None:
        m, n = len(board), len(board[0])
        if m <= 2 or n <= 2:
            return

        dirs = [(-1, 0), (1, 0), (0, -1), (0, 1)]
        queue = deque()

        # 第一遍：将边界上的 'O' 入队并标记为 '#'
        for i in range(m):
            if board[i][0] == 'O':
                board[i][0] = '#'
                queue.append((i, 0))
            if board[i][n - 1] == 'O':
                board[i][n - 1] = '#'
                queue.append((i, n - 1))
        for j in range(n):
            if board[0][j] == 'O':
                board[0][j] = '#'
                queue.append((0, j))
            if board[m - 1][j] == 'O':
                board[m - 1][j] = '#'
                queue.append((m - 1, j))

        # BFS 扩散
        while queue:
            r, c = queue.popleft()
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if 0 <= nr < m and 0 <= nc < n and board[nr][nc] == 'O':
                    board[nr][nc] = '#'
                    queue.append((nr, nc))

        # 第二遍：替换
        for i in range(m):
            for j in range(n):
                if board[i][j] == 'O':
                    board[i][j] = 'X'
                elif board[i][j] == '#':
                    board[i][j] = 'O'
```

#### 方法三：并查集

```python
class Solution:
    def solve(self, board: list[list[str]]) -> None:
        m, n = len(board), len(board[0])
        if m <= 2 or n <= 2:
            return

        dummy = m * n
        parent = list(range(m * n + 1))

        def find(x: int) -> int:
            if parent[x] != x:
                parent[x] = find(parent[x])
            return parent[x]

        def union(x: int, y: int) -> None:
            parent[find(x)] = find(y)

        def is_connected(x: int, y: int) -> bool:
            return find(x) == find(y)

        dirs = [(-1, 0), (1, 0), (0, -1), (0, 1)]

        for i in range(m):
            for j in range(n):
                if board[i][j] != 'O':
                    continue

                idx = i * n + j

                # 边界上的 O 与 dummy 连通
                if i == 0 or i == m - 1 or j == 0 or j == n - 1:
                    union(idx, dummy)

                # 与相邻的 O 连通
                for dr, dc in dirs:
                    ni, nj = i + dr, j + dc
                    if 0 <= ni < m and 0 <= nj < n and board[ni][nj] == 'O':
                        union(idx, ni * n + nj)

        for i in range(m):
            for j in range(n):
                if board[i][j] == 'O' and not is_connected(i * n + j, dummy):
                    board[i][j] = 'X'
```

---

## 逐步推演

以示例 1 为例，演示 DFS 方法的完整过程：

```
初始 board：
    ┌───┬───┬───┬───┐
    │ X │ X │ X │ X │
    ├───┼───┼───┼───┤
    │ X │ O │ O │ X │
    ├───┼───┼───┼───┤
    │ X │ X │ O │ X │
    ├───┼───┼───┼───┤
    │ X │ O │ X │ X │
    └───┴───┴───┴───┘
```

### 第一遍：从边界出发 DFS 标记安全 O

```
检查四条边界：

左边界（j=0）：[0,0]=X, [1,0]=X, [2,0]=X, [3,0]=X → 没有 O

右边界（j=3）：[0,3]=X, [1,3]=X, [2,3]=X, [3,3]=X → 没有 O

上边界（i=0）：[0,0]=X, [0,1]=X, [0,2]=X, [0,3]=X → 没有 O

下边界（i=3）：[3,0]=X, [3,1]=O!, [3,2]=X, [3,3]=X → 发现 O！

从 (3,1) 出发 DFS：
  board[3][1] = '#'（标记为安全）

  检查四个方向：
    (2,1) = X → 不扩散
    (4,1) → 越界
    (3,0) = X → 不扩散
    (3,2) = X → 不扩散

  DFS 结束。只有 (3,1) 被标记。

标记后的矩阵：
    ┌───┬───┬───┬───┐
    │ X │ X │ X │ X │
    ├───┼───┼───┼───┤
    │ X │ O │ O │ X │
    ├───┼───┼───┼───┤
    │ X │ X │ O │ X │
    ├───┼───┼───┼───┤
    │ X │ # │ X │ X │
    └───┴───┴───┴───┘
```

### 第二遍：替换

```
遍历每个格子：
  'O' → 被包围 → 替换为 'X'
  '#' → 安全 → 恢复为 'O'
  'X' → 不变

[0,0]=X → X    [0,1]=X → X    [0,2]=X → X    [0,3]=X → X
[1,0]=X → X    [1,1]=O → X!   [1,2]=O → X!   [1,3]=X → X
[2,0]=X → X    [2,1]=X → X    [2,2]=O → X!   [2,3]=X → X
[3,0]=X → X    [3,1]=# → O!   [3,2]=X → X    [3,3]=X → X

最终结果：
    ┌───┬───┬───┬───┐
    │ X │ X │ X │ X │
    ├───┼───┼───┼───┤
    │ X │ X │ X │ X │
    ├───┼───┼───┼───┤
    │ X │ X │ X │ X │
    ├───┼───┼───┼───┤
    │ X │ O │ X │ X │
    └───┴───┴───┴───┘

与预期输出一致 ✅
```

### 补充示例：多个边界 O 的情况

```
输入：
    ┌───┬───┬───┬───┐
    │ O │ X │ X │ O │
    ├───┼───┼───┼───┤
    │ X │ O │ O │ X │
    ├───┼───┼───┼───┤
    │ X │ X │ O │ X │
    ├───┼───┼───┼───┤
    │ O │ X │ X │ O │
    └───┴───┴───┴───┘

第一遍：从边界出发 DFS

  (0,0)=O → DFS 标记:
    (0,0)→#、(1,1)→#、(1,2)→#、(2,2)→#

  (0,3)=O → DFS 标记:
    (0,3)→# （邻居都是 X，无法扩散）

  (3,0)=O → DFS 标记:
    (3,0)→# （邻居都是 X，无法扩散）

  (3,3)=O → DFS 标记:
    (3,3)→# （邻居都是 X，无法扩散）

标记后：
    ┌───┬───┬───┬───┐
    │ # │ X │ X │ # │
    ├───┼───┼───┼───┤
    │ X │ # │ # │ X │
    ├───┼───┼───┼───┤
    │ X │ X │ # │ X │
    ├───┼───┼───┼───┤
    │ # │ X │ X │ # │
    └───┴───┴───┴───┘

注意：(1,1) 和 (1,2) 没有在边界上，但通过 (0,0) 与边界相连，
因此也被标记为安全 ✅

第二遍：O（没有！）→ X, # → O
结果不变，因为所有 O 都与边界相连！
```

> 💡 这个补充例子展示了关键洞察：(1,1) 和 (1,2) 虽然被 X 包围着，但因为有一条通向边界的路径，它们不会被替换。这就是为什么必须从边界出发做连通性搜索。

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 说明 |
|------|-----------|-----------|------|
| **DFS** | O(m × n) | O(m × n) | 最坏情况递归栈深度 = 矩阵大小（全部是 O） |
| **BFS** | O(m × n) | O(m × n) | 最坏情况队列大小 = 矩阵大小 |
| **并查集** | O(m × n × α(n)) | O(m × n) | α 是阿克曼函数的反函数，≈ 常数 |

三种方法的分析：

- **时间复杂度**：每个格子最多被访问 2 次（DFS/BFS 一次 + 最终遍历一次），边界遍历是 O(m+n)，主遍历是 O(m×n)，总复杂度 O(m×n)。
- **空间复杂度**：DFS 递归栈最深 O(m×n)（全部是 O 且全部相连），BFS 队列也是 O(m×n)，并查集需要 O(m×n) 的 parent 数组。
- **DFS vs BFS**：DFS 代码更短，BFS 更安全（没有递归栈溢出风险）。本题 m,n ≤ 200，DFS 完全够用，推荐 DFS 写法。

---

## 举一反三

### 本题在"矩阵连通性"体系中的位置

```
LeetCode 200. 岛屿数量（Medium）
  → DFS/BFS 统计连通分量数量
  ↓
LeetCode 130. 被围绕的区域（Medium）⭐ 本题
  → 从边界出发找不被包围的区域（逆向思维）
  ↓
LeetCode 417. 太平洋大西洋水流问题（Medium）
  → 从两个边界同时出发，找交集
  ↓
LeetCode 695. 岛屿的最大面积（Medium）
  → DFS/BFS 计算每个连通分量的大小
  ↓
LeetCode 1254. 统计封闭岛屿的数目（Medium）
  → 本题的变种——同样是"边界不计数"
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| **200. 岛屿数量** | DFS/BFS 统计连通分量 | 本题是岛屿问题的变种——先标记边界岛屿 |
| **417. 太平洋大西洋水流问题** | 从两个边界分别 DFS，取交集 | 同一条"从边界出发"思路，扩展到两个边界 |
| **695. 岛屿的最大面积** | DFS/BFS + 计数 | 同样的连通分量搜索，只是搜索过程中多一个计数器 |
| **1254. 统计封闭岛屿的数目** | 边界不算 → 统计内部岛屿 | 和本题几乎一样，只是不翻转而统计数量 |
| **1020. 飞地的数量** | 边界连通的不算 → 统计内部 | 同样是边界逆向思维 + 计数 |
| **463. 岛屿的周长** | 遍历每个格子查边界 | 矩阵遍历 + 边界条件判断 |

### 通用模板：矩阵边界逆向搜索

```javascript
// 矩阵边界逆向搜索 通用模板
function boundaryDFS(board) {
    const m = board.length, n = board[0].length;

    function dfs(r, c) {
        if (r < 0 || r >= m || c < 0 || c >= n) return;
        if (board[r][c] !== TARGET) return;     // TARGET 要搜索的字符

        board[r][c] = MARKER;                    // MARKER 临时标记

        dfs(r - 1, c);  // 上
        dfs(r + 1, c);  // 下
        dfs(r, c - 1);  // 左
        dfs(r, c + 1);  // 右
    }

    // 从四条边界出发
    for (let i = 0; i < m; i++) {
        if (board[i][0] === TARGET)     dfs(i, 0);
        if (board[i][n - 1] === TARGET) dfs(i, n - 1);
    }
    for (let j = 0; j < n; j++) {
        if (board[0][j] === TARGET)     dfs(0, j);
        if (board[m - 1][j] === TARGET) dfs(m - 1, j);
    }

    // 后处理：根据标记做不同操作
}
```

这套模板稍作修改就能应对 130、417、1254、1020 等一系列题目。

---

## 总结

被围绕的区域这道题有三个层次的收获：

1. **核心思维**：正向找"被包围的"很难，反过来找"不被包围的"就很简单。**边界逆向思维**是矩阵连通性问题的核心套路。

2. **三种解法**：
   - **DFS（推荐）**：代码最短，逻辑最直观
   - **BFS**：同样优秀，无递归栈溢出风险
   - **并查集**：通用连通性框架，适合触类旁通

3. **思维框架**：遇到"封闭""包围""边界"等关键词，第一时间问自己——**能从边界反推吗？**

| 步骤 | 操作 |
|------|------|
| ① | 从四条边界的 `O` 出发，DFS/BFS 标记所有相连的 `O` 为临时标记 `#` |
| ② | 遍历矩阵：剩余的 `O` → `X`（被包围）、`#` → `O`（恢复安全） |
| ③ | 完工 ✅ |

**三步记忆法**：
1. **边界 `O`** → 标记为安全（`#`），并 DFS/BFS 扩散到所有相连的 `O`
2. **内部 `O`** → 没有被标记到 → 被围绕 → 替换为 `X`
3. **恢复 `#`** → 变回 `O`

> **记住一句话：判断一个 `O` 是否被包围，就看它是否能走到边界。从边界出发做连通性搜索，标记所有"能走到边界的 `O`"，剩下的就是被包围的。**

在纸上手动模拟一遍——从边界出发标记，第二遍替换——你就会发现这道题其实非常直观。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 200. 岛屿数量](https://leetcode.com/problems/number-of-islands/)（DFS/BFS 基础） | [LeetCode 417. 太平洋大西洋水流问题](https://leetcode.com/problems/pacific-atlantic-water-flow/)（双边界逆向搜索） | [LeetCode 1254. 统计封闭岛屿的数目](https://leetcode.com/problems/number-of-closed-islands/)（本题统计版） | 矩阵系列持续更新中，关注不迷路。
