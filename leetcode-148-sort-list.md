---
title: LeetCode 148. 排序链表
tags: [链表, 归并排序, 分治, 双指针, 自底向上]
difficulty: Medium
category: 链表
date: 2026-06-21
---

# LeetCode 148. 排序链表 —— 链表上的归并排序

## 前言

数组排序有快排、归并、堆排三大经典算法，但链表的世界里，**归并排序**是唯一真神。原因很简单：

- **快排**依赖随机访问（`arr[i]`），链表做不到
- **堆排**需要下标计算父子节点，链表不支持
- **归并排序**只需要"断开"和"合并"两个链表基本操作，天然适配

更妙的是，归并排序在链表上可以做到 **O(1) 空间**（自底向上迭代版），这是数组归并做不到的（数组需要辅助空间，链表只需要改指针）。

本文从**自顶向下递归**到**自底向上迭代**两种实现层层拆解，重点讲解链表归并的核心操作——**找中点**和**合并两个有序链表**，附 JavaScript/Python 双版本代码 + 完整指针推演。

> **前置阅读**：建议先掌握 [LeetCode 21. 合并两个有序链表](https://leetcode.com/problems/merge-two-sorted-lists/)（合并操作）和 [LeetCode 876. 链表的中间结点](https://leetcode.com/problems/middle-of-the-linked-list/)（快慢指针找中点），本题是它们的组合应用。

---

## 问题描述

### LeetCode 148. Sort List（排序链表）

> 给你链表的头结点 `head`，请将其按 **升序** 排列并返回排序后的链表。
>
> 要求：时间复杂度 `O(n log n)`，空间复杂度 `O(1)`（递归栈除外）。

**示例 1：**

```
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```

**示例 2：**

```
输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]
```

**示例 3：**

```
输入：head = []
输出：[]
```

**图示（示例 1）：**

```
原始链表:  4 → 2 → 1 → 3

排序后:    1 → 2 → 3 → 4
```

**约束条件：**
- 链表中节点的数目在范围 `[0, 5 * 10^4]` 内
- `-10^5 <= Node.val <= 10^5`

**进阶**：你可以在 `O(n log n)` 时间复杂度和常数级空间复杂度下，对链表进行排序吗？

---

## 核心思想

### 为什么选归并排序？

链表排序的核心矛盾是：**O(n log n) 的时间复杂度 + O(1) 的空间复杂度**。

| 排序算法 | 时间 | 空间 | 链表可行性 | 原因 |
|---------|------|------|-----------|------|
| 快速排序 | O(n log n) 平均 | O(log n) | ⚠️ 勉强 | 需要随机访问 partition，链表上退化严重 |
| 堆排序 | O(n log n) | O(1) | ❌ 不可行 | 需要数组下标计算父子关系 |
| **归并排序** | O(n log n) | O(1)* | ✅ 最优 | 只需断开和合并，天然适配链表 |

> *链表归并的 O(1) 空间指的是自底向上迭代版，不计递归栈。

### 归并排序的两个核心操作

```
归并排序 = 拆分（找中点） + 合并（合并两个有序链表）

┌─────────────────────────────────────────────────────────┐
│  4 → 2 → 1 → 3                                         │
│  └──找中点──┘                                           │
│  mid = 节点 2                                           │
│                                                         │
│  左半: 4 → 2        右半: 1 → 3                         │
│  递归排序 ↓          递归排序 ↓                           │
│  2 → 4              1 → 3                               │
│       └───── 合并 ─────┘                                │
│       1 → 2 → 3 → 4                                    │
└─────────────────────────────────────────────────────────┘
```

| 操作 | 方法 | 时间 | 关键技巧 |
|------|------|------|---------|
| **找中点** | 快慢指针（slow 走 1 步，fast 走 2 步） | O(n) | `slow` 停在中点的**前一个**节点，方便断开 |
| **合并** | 哑节点 + 双指针逐一比较 | O(n) | 哑节点统一头节点变化的边界 |

### 数据结构设计

```
找中点（快慢指针）：
  head → ... → slow → ... → fast → ...
                  ↑           ↑
              每次走1步    每次走2步
              fast到末尾时，slow恰好在中点

  断开方式：记录 slow.next 作为右半起点，slow.next = null 断开
  ├── 左半: [head, slow]
  └── 右半: [slow.next, 末尾]

合并两个有序链表：
  哑节点 dummy → ...
                  ↑
               cur（当前合并位置）

  l1 → [l1剩余]     l2 → [l2剩余]
   ↑                  ↑
  比较 l1.val 和 l2.val，小的接到 cur 后面
```

---

## 思路分析

### 解法一：自顶向下归并排序（递归）⭐ 经典

**核心思路**：经典的分治递归——找中点断开 → 递归排序左半 → 递归排序右半 → 合并两个有序链表。

```
sortList(head):
  1. 递归终止条件：head 为空 或 head.next 为空（只有一个节点，天然有序）
  2. 快慢指针找中点，断开链表为左右两半
  3. 左半 = sortList(原 head)
  4. 右半 = sortList(原 mid)
  5. return merge(左半, 右半)
```

> 💡 **关键细节**：找中点时，`slow` 要停在中点的**前一个**节点，这样 `slow.next` 就是右半的起点，`slow.next = null` 就能断开链表。为此需要 `prev` 指针记录 `slow` 的前驱。

### 解法二：自底向上归并排序（迭代）⭐⭐ 进阶

**核心思路**：不用递归，直接从底向上合并。先合并相邻的 1 个节点为一组，再合并相邻的 2 个节点为一组，再 4 个、8 个……直到整条链表有序。

```
自底向上过程（以 [4,2,1,3] 为例）：

step=1（每组1个）:  [4] [2] [1] [3]
  两两合并:         [2,4] [1,3]

step=2（每组2个）:  [2,4] [1,3]
  两两合并:         [1,2,3,4]

step=4（每组4个）:  [1,2,3,4]
  只有一组，结束
```

> 💡 **为什么是 O(1) 空间**：迭代版不需要递归栈，每轮合并只需要几个指针变量（`dummy`、`cur`、`l1`、`l2` 等），不随链表长度增长。

### 两种方法对比

```
自顶向下（递归）：
  [4,2,1,3]
  ├── [4,2]          右: [1,3]
  │   ├── [4]  [2]   ├── [1]  [3]
  │   └── [2,4]      └── [1,3]
  └── [1,2,3,4]
  空间: O(log n) 递归栈

自底向上（迭代）：
  [4] [2] [1] [3]     ← 初始，每组 1 个
  [2,4] [1,3]         ← 合并后，每组 2 个
  [1,2,3,4]           ← 合并后，每组 4 个
  空间: O(1)
```

---

## 代码实现

### JavaScript 版本

#### 方法一：自顶向下归并排序（递归）⭐

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
 * @return {ListNode}
 */
var sortList = function(head) {
    // 递归终止：空链表或只有一个节点
    if (!head || !head.next) return head;

    // 1. 快慢指针找中点
    //    slow 最终停在中点的前一个节点，方便断开
    let slow = head, fast = head;
    let prev = null;  // 记录 slow 的前驱
    while (fast && fast.next) {
        prev = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    // 断开链表：左半 [head, prev]，右半 [slow, 末尾]
    prev.next = null;

    // 2. 递归排序左右两半
    const left = sortList(head);
    const right = sortList(slow);

    // 3. 合并两个有序链表
    return merge(left, right);
};

/**
 * 合并两个有序链表（LeetCode 21）
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
function merge(l1, l2) {
    const dummy = new ListNode(0);
    let cur = dummy;

    while (l1 && l2) {
        if (l1.val <= l2.val) {
            cur.next = l1;
            l1 = l1.next;
        } else {
            cur.next = l2;
            l2 = l2.next;
        }
        cur = cur.next;
    }

    // 接上剩余部分
    cur.next = l1 || l2;

    return dummy.next;
}
```

#### 方法二：自底向上归并排序（迭代）⭐⭐

```javascript
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var sortList = function(head) {
    if (!head || !head.next) return head;

    // 1. 计算链表长度
    let n = 0;
    let node = head;
    while (node) {
        n++;
        node = node.next;
    }

    // 2. 自底向上迭代合并
    const dummy = new ListNode(0, head);

    // step: 每组的大小，从 1 开始倍增
    for (let step = 1; step < n; step *= 2) {
        let prev = dummy;   // prev：已合并部分的尾节点
        let cur = dummy.next;  // cur：当前待合并的起点

        while (cur) {
            // 2a. 取出左半部分（[cur, cur + step)）
            const left = cur;
            for (let i = 1; i < step && cur.next; i++) {
                cur = cur.next;
            }

            // 2b. 取出右半部分（[cur.next, cur.next + step)）
            const right = cur.next;
            cur.next = null;  // 断开左半
            cur = right;
            for (let i = 1; i < step && cur && cur.next; i++) {
                cur = cur.next;
            }

            // 2c. 记录下一轮的起点，并断开右半
            let next = null;
            if (cur) {
                next = cur.next;
                cur.next = null;  // 断开右半
            }

            // 2d. 合并左右两半，接到 prev 后面
            const merged = merge(left, right);
            prev.next = merged;

            // 2e. prev 移到合并后的尾部
            while (prev.next) {
                prev = prev.next;
            }

            // 2f. cur 移到下一轮起点
            cur = next;
        }
    }

    return dummy.next;
};

function merge(l1, l2) {
    const dummy = new ListNode(0);
    let cur = dummy;
    while (l1 && l2) {
        if (l1.val <= l2.val) {
            cur.next = l1;
            l1 = l1.next;
        } else {
            cur.next = l2;
            l2 = l2.next;
        }
        cur = cur.next;
    }
    cur.next = l1 || l2;
    return dummy.next;
}
```

### Python 版本

#### 方法一：自顶向下归并排序（递归）⭐

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

def sortList(head: ListNode) -> ListNode:
    """方法一：自顶向下归并排序（递归）"""
    # 递归终止：空链表或只有一个节点
    if not head or not head.next:
        return head

    # 1. 快慢指针找中点
    slow, fast = head, head
    prev = None
    while fast and fast.next:
        prev = slow
        slow = slow.next
        fast = fast.next.next
    # 断开链表
    prev.next = None

    # 2. 递归排序左右两半
    left = sortList(head)
    right = sortList(slow)

    # 3. 合并
    return merge(left, right)


def merge(l1: ListNode, l2: ListNode) -> ListNode:
    """合并两个有序链表"""
    dummy = ListNode(0)
    cur = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            cur.next = l1
            l1 = l1.next
        else:
            cur.next = l2
            l2 = l2.next
        cur = cur.next
    cur.next = l1 or l2
    return dummy.next
```

#### 方法二：自底向上归并排序（迭代）⭐⭐

```python
def sortList(head: ListNode) -> ListNode:
    """方法二：自底向上归并排序（迭代）"""
    if not head or not head.next:
        return head

    # 1. 计算链表长度
    n = 0
    node = head
    while node:
        n += 1
        node = node.next

    # 2. 自底向上迭代合并
    dummy = ListNode(0, head)

    step = 1
    while step < n:
        prev = dummy
        cur = dummy.next

        while cur:
            # 取出左半部分
            left = cur
            for _ in range(1, step):
                if cur.next:
                    cur = cur.next
            # 断开左半
            right = cur.next
            cur.next = None
            cur = right
            # 取出右半部分
            for _ in range(1, step):
                if cur and cur.next:
                    cur = cur.next
            # 记录下一轮起点，断开右半
            nxt = None
            if cur:
                nxt = cur.next
                cur.next = None
            # 合并并接到 prev 后面
            prev.next = merge(left, right)
            # prev 移到合并后的尾部
            while prev.next:
                prev = prev.next
            # 移到下一轮起点
            cur = nxt

        step *= 2

    return dummy.next
```

---

## 逐步推演

### 递归法完整推演

以 `head = [4,2,1,3]` 为例，逐步展示递归过程：

```
sortList([4,2,1,3])

═══════════════ 第 1 层递归 ═══════════════

步骤 1：找中点
  4 → 2 → 1 → 3
  ↑s ↑f         初始: slow=4, fast=4
  ↑      ↑s ↑f  第 1 轮: slow=2, fast=1 (fast.next=3)
                    第 2 轮: slow=1, fast=null (fast.next.next=null, 停)

  但 fast.next=3 存在，所以还需要一轮：
  实际过程:
    初始: slow=4, fast=4, prev=null
    第 1 轮: prev=4, slow=2, fast=1
    第 2 轮: fast.next=3, fast.next.next=null → 停！
    prev=4, slow=2

  断开: prev.next = null → 4→null, 2→1→3
  左半: [4], 右半: [2,1,3]

步骤 2：递归排序左半 → sortList([4])
  只有 1 个节点 → 返回 [4]

步骤 3：递归排序右半 → sortList([2,1,3])

═══════════════ 第 2 层递归（右半） ═══════════════

  步骤 1：找中点
    2 → 1 → 3
    ↑s ↑f        初始: slow=2, fast=2
    ↑      ↑s ↑f 第 1 轮: slow=1, fast=3
                  第 2 轮: fast.next=null → 停！

    prev=2, slow=1
    断开: 2→null, 1→3
    左半: [2], 右半: [1,3]

  步骤 2：递归排序左半 → sortList([2]) → 返回 [2]

  步骤 3：递归排序右半 → sortList([1,3])

═══════════════ 第 3 层递归（右半的右半） ═══════════════

    步骤 1：找中点
      1 → 3
      ↑s ↑f        初始: slow=1, fast=1
                    第 1 轮: fast.next=null → 停！
      prev=1, slow=3

      断开: 1→null, 3
      左半: [1], 右半: [3]

    步骤 2：sortList([1]) → [1]
    步骤 3：sortList([3]) → [3]
    步骤 4：merge([1], [3])
      dummy → 1 → 3
      返回 [1,3]

═══════════════ 回溯第 2 层 ═══════════════

  左半 = [2], 右半 = [1,3]
  步骤 4：merge([2], [1,3])

    初始: dummy→null, cur=dummy
    比较: 2 vs 1 → 选 1 → dummy→1
    比较: 2 vs 3 → 选 2 → 1→2
    剩余: 3 → 2→3
    结果: 1→2→3

  返回 [1,2,3]

═══════════════ 回溯第 1 层 ═══════════════

左半 = [4], 右半 = [1,2,3]
步骤 4：merge([4], [1,2,3])

  初始: dummy→null, cur=dummy
  比较: 4 vs 1 → 选 1 → dummy→1
  比较: 4 vs 2 → 选 2 → 1→2
  比较: 4 vs 3 → 选 3 → 2→3
  剩余: 4 → 3→4
  结果: 1→2→3→4

返回 [1,2,3,4] ✅
```

### 自底向上迭代法完整推演

以 `head = [4,2,1,3]` 为例，逐步展示每轮合并：

```
初始链表: 4 → 2 → 1 → 3
链表长度 n = 4


═══════════════ step = 1（每组 1 个节点） ═══════════════

第 1 轮合并:
  left = [4], right = [2]
  merge([4], [2]) → 2 → 4
  链表: 2 → 4 → 1 → 3
  prev 移到 4

第 2 轮合并:
  left = [1], right = [3]
  merge([1], [3]) → 1 → 3
  链表: 2 → 4 → 1 → 3
  prev 移到 3

本轮结束，链表: 2 → 4 → 1 → 3


═══════════════ step = 2（每组 2 个节点） ═══════════════

第 1 轮合并:
  left = [2,4], right = [1,3]
  merge([2,4], [1,3]):
    比较: 2 vs 1 → 选 1
    比较: 2 vs 3 → 选 2
    比较: 4 vs 3 → 选 3
    剩余: 4
    结果: 1 → 2 → 3 → 4
  链表: 1 → 2 → 3 → 4
  prev 移到 4

本轮结束，链表: 1 → 2 → 3 → 4


═══════════════ step = 4（每组 4 个节点） ═══════════════

step=4 >= n=4，循环结束

最终链表: 1 → 2 → 3 → 4 ✅
```

### 找中点推演

以 `[4,2,1,3]` 为例，展示快慢指针找中点的过程：

```
链表: 4 → 2 → 1 → 3

初始: slow=4, fast=4, prev=null

第 1 轮:
  fast 有 next（2）, fast.next 有 next（3）→ 继续
  prev = slow = 4
  slow = slow.next = 2
  fast = fast.next.next = 1

第 2 轮:
  fast=1, fast.next=3, fast.next.next=null → 停止！
  prev = slow = 2
  slow = slow.next = 1
  fast = fast.next.next = null

结果: prev=2, slow=1
断开: prev.next = null → 左半 [4,2], 右半 [1,3]

注意：这里找的是"中点偏左"的断开方式
  4 → 2 | 1 → 3
  ←左半→   ←右半→
```

> 💡 **找中点的两种风格**：
> - **停在中点前一个**（本题用法）：需要 `prev` 指针，断开时 `prev.next = null`
> - **停在中点本身**：不需要 `prev`，但断开时要用 `mid.next` 作为右半起点，`mid.next = null` 断开
>
> 两种都可以，关键是断开方式要一致。

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **递归归并** ⭐ | O(n log n) | O(log n) | 代码直观，易于理解和实现 | 递归栈占用 O(log n) 空间 |
| **迭代归并** ⭐⭐ | O(n log n) | O(1) | 空间最优，无栈溢出风险 | 代码较复杂，指针操作多 |

> - **时间复杂度**：两种方法都是 O(n log n)。归并排序每层需要 O(n) 的合并操作，共 log n 层
> - **空间复杂度**：递归版需要 O(log n) 的递归栈深度；迭代版只用常数个指针变量，O(1)
> - **稳定性**：归并排序是稳定排序（相等元素保持原始顺序），这在链表场景中很重要

### 各操作的时间分解

```
每层归并的总工作量：

  找中点:  O(n/2^k) × 2^k = O(n)  （每组找一次，共 2^k 组）
  合并:    O(n)                   （每组合并两个子链表，总元素数 = n）
  ─────────────────────────────
  每层总计: O(n)

  层数: log n（每次对半分）
  总计: O(n log n) ✅
```

---

## 举一反三

### 本题在链表排序体系中的位置

```
链表排序核心题：

LeetCode 148 排序链表（Medium）
  ├── 核心操作 1: 快慢指针找中点 → LeetCode 876
  ├── 核心操作 2: 合并有序链表 → LeetCode 21
  ├── 变体: 合并 K 个有序链表 → LeetCode 23
  └── 进阶: O(1) 空间迭代归并 → 本题重点
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [21. 合并两个有序链表](https://leetcode.com/problems/merge-two-sorted-lists/) | 哑节点 + 双指针合并 | 本题的核心子操作 |
| [876. 链表的中间结点](https://leetcode.com/problems/middle-of-the-linked-list/) | 快慢指针找中点 | 本题找中点的简化版 |
| [23. 合并 K 个升序链表](https://leetcode.com/problems/merge-k-sorted-lists/) | 分治合并 / 最小堆 | 本题的推广——从 2 路到 K 路 |
| [147. 对链表进行插入排序](https://leetcode.com/problems/insertion-sort-list/) | 插入排序、哑节点 | 链表排序的另一种思路，O(n²) |
| [912. 排序数组](https://leetcode.com/problems/sort-an-array/) | 数组归并排序 | 数组版归并，需要辅助空间 |

### 链表归并排序通用框架

反复出现的模式提炼：

```
链表归并排序 = 拆分 + 递归/迭代 + 合并

1. 拆分（找中点）：
   ├── 快慢指针：slow 走 1 步，fast 走 2 步
   ├── fast 到末尾时，slow 在中点附近
   └── 断开：记录断开点，设置 .next = null

2. 排序：
   ├── 递归版：sortList(left) + sortList(right)
   └── 迭代版：step 从 1 倍增到 n，每轮合并相邻 step 大小的组

3. 合并（两个有序链表）：
   ├── 哑节点 dummy 统一边界
   ├── 双指针逐一比较，小的接到结果链表
   └── 剩余部分直接接上
```

> **一句话总结**：链表排序 = 快慢指针找中点（断开） + 合并两个有序链表。递归版代码最简洁，迭代版空间最优。掌握这两个子操作，链表排序就是它们的组合。

### 延伸思考

在实际面试中，面试官可能追问：

1. **"为什么不用快速排序？"** → 快排需要随机访问 `arr[i]`，链表不支持；即使强行实现 partition，最坏情况 O(n²) 且常数因子大
2. **"递归版的空间复杂度？"** → O(log n) 递归栈深度。如果面试官要求严格 O(1)，需要写迭代版
3. **"如何保证排序的稳定性？"** → 归并排序天然稳定，合并时 `l1.val <= l2.val`（注意 `<=`）保证相等元素不交换顺序
4. **"能不能在原链表上直接操作，不创建新节点？"** → 本题解法就是原地操作——只改指针，不创建新节点

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **148 排序链表** | 🟡 Medium | 归并排序、快慢指针、分治 | 递归归并 / 迭代归并 |

这道题的核心是**链表归并排序**的两种实现。抓住两个子操作，问题就迎刃而解：

| 子操作 | 方法 | 关键点 |
|--------|------|--------|
| **找中点** | 快慢指针 | slow 走 1 步，fast 走 2 步，fast 到末尾时 slow 在中点 |
| **合并** | 哑节点 + 双指针 | 逐一比较取小的，剩余直接接上 |

**两组解法各有千秋：**

1. **递归归并（推荐）**：代码最直观。核心口诀：**找中点断开，递归排左右，合并两半。** 面试中最容易写对的解法，O(log n) 栈空间在大多数场景下可以接受。

2. **迭代归并（进阶）**：O(1) 空间，无栈溢出风险。核心口诀：**step 从 1 倍增到 n，每轮合并相邻的两个 step 大小组。** 如果面试官追问"能不能做到严格 O(1) 空间"，这就是答案。

> **记住一句话：链表排序只能用归并。找中点用快慢指针，合并用哑节点+双指针。递归版好写，迭代版省空间——两个子操作组合起来，这道 Medium 题就是送分题。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 21. 合并两个有序链表](https://leetcode.com/problems/merge-two-sorted-lists/)（合并操作） | [LeetCode 876. 链表的中间结点](https://leetcode.com/problems/middle-of-the-linked-list/)（找中点） | [LeetCode 23. 合并 K 个升序链表](https://leetcode.com/problems/merge-k-sorted-lists/)（K 路合并） | 链表系列持续更新中，关注不迷路。
