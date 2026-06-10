---
title: LeetCode 92. 翻转链表 II
tags: [链表, 头插法, 穿针引线, 分段反转, 递归]
difficulty: Medium
category: 链表
date: 2026-06-10
---

# LeetCode 92. 翻转链表 II —— 穿针引线：三种方法吃透链表局部反转

## 前言

链表反转是数据结构中的基本功。LeetCode 206 **翻转整个链表**是 Easy 题，而 LeetCode 92 **翻转链表 II** 将它升级为 Medium——只反转链表中 `[left, right]` 这一段。

这道题考察的核心能力是**指针操作**：如何在反转部分链表时，正确连接"前段 → 反转段 → 后段"三个部分。三种主流方法：

1. **头插法（穿针引线）**：最优雅，一趟扫描，逐个把节点"拎"到反转段的最前面
2. **分段反转法**：最直观，截断 → 反转 → 拼接，把问题拆成子任务
3. **递归法**：最简洁的代码，但需要理解递归的"后序遍历"思想

本文附带 JavaScript 和 Python 双语言实现，以及每一步的指针变化推演。

---

## 问题描述

### LeetCode 92. Reverse Linked List II（翻转链表 II）

> 给你单链表的头指针 `head` 和两个整数 `left` 和 `right`，其中 `left <= right`。请你反转从位置 `left` 到位置 `right` 的链表节点，返回**翻转后的链表**。

**示例：**

```
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```

```
输入：head = [5], left = 1, right = 1
输出：[5]
```

**图示：**

```
原始链表:  1 → 2 → 3 → 4 → 5
               ↑         ↑
              left=2   right=4

反转后:    1 → 4 → 3 → 2 → 5
               ↑_________↑
              反转段 [2,4]
```

**约束条件：**
- 链表中节点数目为 `n`
- `1 <= n <= 500`
- `-500 <= Node.val <= 500`
- `1 <= left <= right <= n`

---

## 核心思想

### 链表的"穿针引线"

反转链表的核心操作是**改变节点的 `next` 指向**。对于局部反转，关键挑战在于：

```
原始:   ... → A → [B → C → D] → E → ...
                           ↑
                    反转段 [left, right]

目标:   ... → A → [D → C → B] → E → ...
                  ↑_________↑
                 反转后的段
```

你需要同时维护三个关键位置的指针：

| 位置 | 指针名 | 含义 |
|------|-------|------|
| **前驱** | `prev` / `preLeft` | 反转段的前一个节点（例子中的 A） |
| **反转段起点** | `leftNode` / `cur` | 原反转段的第一个节点（例子中的 B） |
| **后继** | `rightNode.next` | 反转段的后一个节点（例子中的 E） |

反转完成后，需要重新连接：
- A 的 next → 反转后的新头（原来的 D）
- 原来 B 的 next → E

---

## 思路分析

### 解法一：头插法（穿针引线）⭐ 推荐

**核心思路**：找到反转段的前驱 `prev` 之后，对于 `[left+1, right]` 范围内的每个节点，将它从链表中"摘"下来，插入到 `prev` 后面（即反转段的最前面）。

```
以 [1,2,3,4,5], left=2, right=4 为例：

初始:  1 → 2 → 3 → 4 → 5
       ↑   ↑
      prev cur (prev 是 left 的前一个节点)

第一轮（处理节点 3）：
  1. 摘下 cur.next（即 3）:  removed = 3
  2. cur.next = removed.next  → 2 → 4 → 5
  3. removed.next = prev.next → 3 → 2 → 4 → 5
  4. prev.next = removed      → 1 → 3 → 2 → 4 → 5

第二轮（处理节点 4）：
  1. 摘下 cur.next（现在 cur 还是指向 2，cur.next 是 4）:  removed = 4
  2. cur.next = removed.next  → 2 → 5
  3. removed.next = prev.next → 4 → 3 → 2 → 5
  4. prev.next = removed      → 1 → 4 → 3 → 2 → 5

完成！需要操作 right - left = 2 次（即 head-insert 2 次）
```

> 💡 **关键洞察**：头插法全程 `prev` 和 `cur` 不动，只把 `cur.next` 依次"提"到 `prev` 后面。操作次数 = `right - left`。

### 解法二：分段反转法

**核心思路**：把问题拆成三个子任务：

```
1. 定位：找到 left 的前驱 preLeft 和 right 的后继 rightNext
2. 截断+反转：将 [left, right] 段截下来，单独反转
3. 拼接：将前段 → 反转段 → 后段 重新连接

原始: 1 → [2 → 3 → 4] → 5
          ↑          ↑
        preLeft    rightNext

步骤1: 截断 → 反转 [2→3→4] 得到 [4→3→2]
步骤2: preLeft.next = 反转后的头 (4)
步骤3: 原 left 节点(2).next = rightNext (5)
```

这种方法的好处是**思路清晰**——"截断→反转→拼接"，每个子任务独立可测。而且反转子链表可以直接复用 LeetCode 206 反转链表的代码。

### 解法三：递归法

**核心思路**：利用递归的**后序遍历**——当你递归到最深处时，`right` 对应的节点就是递归到底的位置，然后逐层返回时完成反转。

```
递归函数: reverseBetween(node, m, n)

当 m == 1 时：问题变成"翻转前 n 个节点" → reverseN(node, n)
当 m > 1 时：node.next = reverseBetween(node.next, m-1, n-1)

reverseN 的实现：
  递归到 n == 1 时，记录后继 successor
  回溯时，让 node.next.next = node（反转指针）
  最后 node.next = successor（连接后继）
```

递归法的代码非常精炼，但需要较强的递归思维。理解的关键在于：**递归回溯时，每一层都拥有"后面的链表已经反转好了"的保证**。

---

## 代码实现

### JavaScript 版本

#### 方法一：头插法（穿针引线）⭐

```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} left
 * @param {number} right
 * @return {ListNode}
 */
var reverseBetween = function(head, left, right) {
    // 哑节点，处理 left=1 的情况
    const dummy = new ListNode(0, head);

    // 1. 找到 left 的前驱节点 prev
    let prev = dummy;
    for (let i = 1; i < left; i++) {
        prev = prev.next;
    }

    // 2. cur 指向反转段的第一个节点（也是反转后的最后一个节点）
    let cur = prev.next;

    // 3. 头插法：将 cur.next 逐个插入到 prev 后面
    //    需要操作 right - left 次
    for (let i = 0; i < right - left; i++) {
        const removed = cur.next;       // 要"拎"出来的节点
        cur.next = removed.next;        // cur 跳过 removed，连接下一个
        removed.next = prev.next;       // removed 插入到反转段最前面
        prev.next = removed;            // prev 指向新的反转段头
    }

    return dummy.next;
};
```

#### 方法二：分段反转法

```javascript
/**
 * @param {ListNode} head
 * @param {number} left
 * @param {number} right
 * @return {ListNode}
 */
var reverseBetween = function(head, left, right) {
    const dummy = new ListNode(0, head);

    // 1. 找到 left 的前驱 preLeft
    let preLeft = dummy;
    for (let i = 1; i < left; i++) {
        preLeft = preLeft.next;
    }

    // 2. 找到 right 节点
    let rightNode = preLeft;
    for (let i = 0; i <= right - left; i++) {
        rightNode = rightNode.next;
    }

    // 3. 截断：记录 left 节点和 right 的后继
    const leftNode = preLeft.next;
    const rightNext = rightNode.next;

    // 4. 反转 [leftNode, rightNode] 这一段
    function reverseList(node, endNext) {
        let prev = endNext;   // 反转后，原第一个节点的 next 应指向 rightNext
        let cur = node;
        while (cur !== endNext) {
            const next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        return prev;  // 新的头（原 rightNode）
    }

    // 5. 拼接
    preLeft.next = reverseList(leftNode, rightNext);

    return dummy.next;
};
```

#### 方法三：递归法

```javascript
/**
 * @param {ListNode} head
 * @param {number} left
 * @param {number} right
 * @return {ListNode}
 */
var reverseBetween = function(head, left, right) {
    let successor = null; // 记录反转段后面的第一个节点

    // 反转前 n 个节点
    function reverseN(node, n) {
        if (n === 1) {
            successor = node.next;
            return node;
        }
        const last = reverseN(node.next, n - 1);
        node.next.next = node;
        node.next = successor;
        return last;
    }

    // 主递归
    if (left === 1) {
        return reverseN(head, right);
    }
    // 前进到 left 位置，同时缩小问题规模
    head.next = reverseBetween(head.next, left - 1, right - 1);
    return head;
};
```

### Python 版本

#### 方法一：头插法（穿针引线）⭐

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

def reverseBetween(head: ListNode, left: int, right: int) -> ListNode:
    """方法一：头插法（穿针引线）"""
    # 哑节点，处理 left=1 的情况
    dummy = ListNode(0, head)

    # 1. 找到 left 的前驱节点 prev
    prev = dummy
    for _ in range(1, left):
        prev = prev.next

    # 2. cur 指向反转段的第一个节点
    cur = prev.next

    # 3. 头插法：将 cur.next 逐个插入到 prev 后面
    for _ in range(right - left):
        removed = cur.next         # 要"拎"出来的节点
        cur.next = removed.next    # cur 跳过 removed
        removed.next = prev.next   # removed 插到反转段最前面
        prev.next = removed        # prev 指向新反转段头

    return dummy.next
```

#### 方法二：分段反转法

```python
def reverseBetween(head: ListNode, left: int, right: int) -> ListNode:
    """方法二：分段反转法"""
    dummy = ListNode(0, head)

    # 1. 找到 left 的前驱 preLeft
    pre_left = dummy
    for _ in range(1, left):
        pre_left = pre_left.next

    # 2. 找到 right 节点
    right_node = pre_left
    for _ in range(right - left + 1):
        right_node = right_node.next

    # 3. 截断
    left_node = pre_left.next
    right_next = right_node.next

    # 4. 反转 [left_node, right_node]
    def reverse_list(node: ListNode, end_next: ListNode) -> ListNode:
        prev = end_next
        cur = node
        while cur is not end_next:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        return prev

    # 5. 拼接
    pre_left.next = reverse_list(left_node, right_next)

    return dummy.next
```

#### 方法三：递归法

```python
def reverseBetween(head: ListNode, left: int, right: int) -> ListNode:
    """方法三：递归法"""
    successor = None

    def reverseN(node: ListNode, n: int) -> ListNode:
        """反转前 n 个节点"""
        nonlocal successor
        if n == 1:
            successor = node.next
            return node
        last = reverseN(node.next, n - 1)
        node.next.next = node
        node.next = successor
        return last

    if left == 1:
        return reverseN(head, right)

    head.next = reverseBetween(head.next, left - 1, right - 1)
    return head
```

---

## 逐步推演

### 头插法完整推演

以 `head = [1,2,3,4,5], left = 2, right = 4` 为例：

```
初始状态:
dummy → 1 → 2 → 3 → 4 → 5
         ↑   ↑
        prev cur
操作次数 = right - left = 2

═══════════════ 第 1 轮（i=0） ═══════════════

  当前链表:
   dummy → 1 → 2 → 3 → 4 → 5
           ↑   ↑   ↑
          prev cur removed(cur.next=3)

  步骤① cur.next = removed.next (2→4):
   dummy → 1 → 2 ───→ 4 → 5
           ↑   ↑      ↑
          prev cur    (3 被摘出来了)
                └→ 3 ↗ removed

  步骤② removed.next = prev.next (3→2):
   dummy → 1    2 ───→ 4 → 5
           ↑    ↑      ↑
          prev  cur
           ↓
           3 ─┘
          removed → 2

  步骤③ prev.next = removed (1→3):
   dummy → 1 → 3 → 2 → 4 → 5
           ↑       ↑
          prev    cur (cur 仍然指向 2！)

  链表变为: 1 → 3 → 2 → 4 → 5


═══════════════ 第 2 轮（i=1） ═══════════════

  当前链表:
   dummy → 1 → 3 → 2 → 4 → 5
           ↑       ↑   ↑
          prev    cur removed(cur.next=4)

  步骤① cur.next = removed.next (2→5):
   dummy → 1 → 3 → 2 ───→ 5
           ↑       ↑      ↑
          prev    cur    (4 被摘出来了)
                   └→ 4 ↗ removed

  步骤② removed.next = prev.next (4→3):
   dummy → 1    3 → 2 ───→ 5
           ↑    ↑   ↑
          prev      cur
           ↓
           4 ─┘
          removed → 3

  步骤③ prev.next = removed (1→4):
   dummy → 1 → 4 → 3 → 2 → 5
           ↑           ↑
          prev        cur

  最终链表: 1 → 4 → 3 → 2 → 5 ✅
```

> 💡 **核心观察**：整个过程中 `prev`（节点 1）和 `cur`（节点 2）的引用始终不变！变的只是 `prev.next` 和 `cur.next` 的指向。

### 分段反转法推演同一示例

```
初始: dummy → 1 → 2 → 3 → 4 → 5

步骤1: 定位
  pre_left = 1 (left=2 的前驱)
  left_node = 2
  right_node = 4
  right_next = 5

步骤2: 截断
  right_node.next = None  (暂时断开)
  反转段: [2 → 3 → 4] → None
  前段:   1
  后段:   5

步骤3: 反转 [2→3→4]
  遍历过程:
    初始: prev=None, cur=2
    → next=3, cur.next=None, prev=2, cur=3
    → next=4, cur.next=2,   prev=3, cur=4
    → next=None, cur.next=3, prev=4, cur=None
  结果: 4 → 3 → 2 → None

步骤4: 拼接
  pre_left.next = 4  →  1 → 4 → 3 → 2
  left_node.next = 5  →  1 → 4 → 3 → 2 → 5 ✅
```

### 递归法推演同一示例

```
reverseBetween([1,2,3,4,5], left=2, right=4)

left=2 ≠ 1，所以:
  head.next = reverseBetween([2,3,4,5], left=1, right=3)

  left=1，调用 reverseN([2,3,4,5], n=3)

  reverseN(node=2, n=3):
    reverseN(node=3, n=2):
      reverseN(node=4, n=1):
        n==1 → successor = 5, return 4

      回溯: node=3, last=4
        node.next.next = node  →  3.next.next = 3  →  4.next = 3
        node.next = successor  →  3.next = 5
        链表: 4 → 3 → 5
        return 4

    回溯: node=2, last=4
      node.next.next = node  →  2.next.next = 2  →  3.next = 2
      node.next = successor  →  2.next = 5
      链表: 4 → 3 → 2 → 5
      return 4

  返回: 4 → 3 → 2 → 5

回溯到第一层: head=1
  head.next = 4→3→2→5
  链表: 1 → 4 → 3 → 2 → 5 ✅
```

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **头插法** ⭐ | O(n) | O(1) | 一趟扫描，代码最紧凑，指针操作精妙 | 需要理解"拎节点"的过程 |
| **分段反转法** | O(n) | O(1) | 思路最直观，拆分→反转→拼接，可复用 | 需要两次遍历（找 right + 反转） |
| **递归法** | O(n) | O(n) | 代码最简洁优雅 | 递归栈 O(n) 空间，不易调试 |

> 注：n = 链表长度。三种方法都只访问反转段内的节点一次或常数次。头插法严格一趟扫描，分段反转法需要先定位再反转。空间上，前两种方法 O(1)，递归法由于调用栈深度为 O(right) = O(n)。

---

## 三种方法的本质联系

```
头插法 ──── 逐个把 cur.next "插"到 prev 后面 ────┐
                                                   │
分段反转法 ──── 截断→单独反转→拼接回去 ────────┤ → 本质相同
                                                   │    都是"保持前驱不变，
递归法 ──── 利用回溯逐层反转指针 ────────────┘    反转段内部旋转"

头插法 = 分段反转的"在线"版本（边遍历边反转，无需截断）
递归法 = 头插法的"函数式"表达（用递归代替显式循环）
```

### 一句话理解

> **头插法就是把反转段想象成一个"台球桌"，每次从桌子尾部拿一个球，塞到桌子最前面。做 `right - left` 次，反转段就翻转了。**

---

## 举一反三

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [206. 反转链表](https://leetcode.com/problems/reverse-linked-list/) | 基础链表反转（迭代/递归） | 本题的前置基础，分段反转法直接复用 |
| [25. K 个一组翻转链表](https://leetcode.com/problems/reverse-nodes-in-k-group/) | 分组 + 局部反转 | 本题的进阶——多段反转，每段用头插法或分段法 |
| [24. 两两交换链表中的节点](https://leetcode.com/problems/swap-nodes-in-pairs/) | 相邻节点交换 | 本题的特例（left, right 间距为 1）+ 多段 |
| [143. 重排链表](https://leetcode.com/problems/reorder-list/) | 找中点 + 反转后半段 + 合并 | 分段反转法的典型应用场景 |
| [234. 回文链表](https://leetcode.com/problems/palindrome-linked-list/) | 找中点 + 反转后半段 + 比较 | 分段反转 + 双指针 |

### 链表局部反转通用框架

```
链表局部反转三步走：

1. 用 dummy 节点统一处理 left=1 的边界情况

2. 找到反转段的前驱 prev（走 left-1 步）

3. 选择策略执行反转：
   - 头插法：right-left 次循环，每次 prev.next↔cur.next 交换
   - 分段法：截断 → 标准反转 → 拼接
   - 递归法：left==1 时转成"反转前 n 个"子问题

模式识别：凡是"反转链表的一部分"，先想到头插法。
凡是"多段分别反转"，先想到分段法（每段复用同一逻辑）。
```

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **92 翻转链表 II** | 🟡 Medium | 穿针引线、分段反转、递归回溯 | 头插法 |

三种方法层层递进：

1. **头插法（推荐）**：一趟扫描，O(1) 空间。核心口诀：**prev 不动，cur 不动，每次把 cur.next 插到 prev 后面，做 right-left 次。** 这是面试中最加分的方法。

2. **分段反转法**：最符合直觉——找到前后边界，截断，反转，拼回去。好处是可以直接复用 LeetCode 206 反转链表的代码，适合"稳扎稳打"的面试节奏。

3. **递归法**：代码最短但思维负担最重。理解的关键在于：**递归到底后，node.next.next = node 是在回溯时完成的**。如果面试官追问"不用迭代怎么写"，这就是答案。

> **记住一句话：链表的局部反转 = 固定前驱 + 旋转段内节点。头插法是最优解，分段法是最稳解，递归法是最美解。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 206. 反转链表](https://leetcode.com/problems/reverse-linked-list/)（本题前置基础） | [LeetCode 25. K 个一组翻转链表](https://leetcode.com/problems/reverse-nodes-in-k-group/)（本题进阶） | 后续会继续更新链表系列，关注不迷路。
