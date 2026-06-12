---
title: LeetCode 25. K 个一组翻转链表
tags: [链表, 分组反转, 穿针引线, 递归, 哑节点]
difficulty: Hard
category: 链表
date: 2026-06-12
---

# LeetCode 25. K 个一组翻转链表 —— 从局部反转到分组反转的跃迁

## 前言

如果说 LeetCode 206 **反转整个链表**是基本功，LeetCode 92 **翻转链表 II** 是局部反转的"穿针引线"技巧，那么 LeetCode 25 **K 个一组翻转链表** 就是将这个模式放大到**多段分组**——每 K 个节点为一组，组内反转，组间保持原序，最后不足 K 个的尾巴原样保留。

这道 Hard 题是链表操作的集大成者。它要求你同时掌握：

1. **哑节点（Dummy Node）** —— 统一头节点变化的边界
2. **标准链表反转** —— 每 K 个节点执行一次
3. **组间连接** —— 前一组反转后的尾 → 当前组反转后的头 → 下一组

本文用**迭代法（穿针引线）**和**递归法**两种思路层层拆解，附 JavaScript/Python 双版本代码 + 完整指针推演。

> **前置阅读**：建议先掌握 [LeetCode 206. 反转链表](https://leetcode.com/problems/reverse-linked-list/)（基础反转）和 [LeetCode 92. 翻转链表 II](./leetcode-92-reverse-linked-list-ii.md)（局部反转），本题是它们的自然延伸。

---

## 问题描述

### LeetCode 25. Reverse Nodes in k-Group（K 个一组翻转链表）

> 给你链表的头节点 `head`，每 `k` 个节点一组进行翻转，请你返回修改后的链表。
>
> `k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。
>
> **你不能只是单纯地改变节点内部的值**，而是需要实际进行节点交换。

**示例 1：**

```
输入：head = [1,2,3,4,5], k = 2
输出：[2,1,4,3,5]
```

**示例 2：**

```
输入：head = [1,2,3,4,5], k = 3
输出：[3,2,1,4,5]
解释：前 3 个节点 [1,2,3] 翻转成 [3,2,1]，后 2 个节点 [4,5] 不足 3 个，保持原样
```

**图示（示例 1，k=2）：**

```
原始链表:  1 → 2 → 3 → 4 → 5
           └─组1─┘ └─组2─┘ └尾┘

翻转后:    2 → 1 → 4 → 3 → 5
           └─组1─┘ └─组2─┘ └尾┘
           (组内翻转)  (组内翻转)  (不足k,不变)
```

**图示（示例 2，k=3）：**

```
原始链表:  1 → 2 → 3 → 4 → 5
           └───组1───┘ └尾巴┘

翻转后:    3 → 2 → 1 → 4 → 5
           └───组1───┘ └尾巴┘
           (组内翻转)   (不足k,不变)
```

**约束条件：**
- 链表中的节点数目为 `n`
- `1 <= k <= n <= 5000`
- `0 <= Node.val <= 1000`

---

## 核心思想

### 问题的本质：两件事交替做

把 K 个一组翻转链表拆开来看，其实是不停重复两件事：

```
重复执行，直到剩余节点不足 k 个：
  ┌──────────────────────────────────────────┐
  │ 1. 检查：当前位置往后是否还有 k 个节点？    │
  │    ├── 有 → 执行步骤 2                    │
  │    └── 没有 → 结束，剩余部分原样保留        │
  │                                          │
  │ 2. 翻转这 k 个节点，并正确连接：            │
  │    前一组尾 → [翻转后的组] → 剩余链表       │
  └──────────────────────────────────────────┘
```

所以这个问题的核心挑战就两点：

| 挑战 | 解法 |
|------|------|
| **怎么知道还剩几个节点？** | 从当前位置往后数 k 步，看是否会碰到 null |
| **翻转 k 个节点后怎么接回去？** | 记录"前一组尾"和"后一组头"，翻转完后做指针重连 |

### 数据结构设计

```
├── dummy（哨兵）→ 永远指向链表头，统一边界处理
├── prevGroup：指向「当前待反转组」的前一个节点
│               初始 = dummy，每反转完一组就前进 k 步
├── kth：从 prevGroup 往后走 k 步到达的节点
│        如果走不到（碰到 null），说明不够 k 个，停止
├── groupHead = prevGroup.next：当前组的第一个节点（反转前）
│                               反转后变成当前组的最后一个节点
├── nextGroup = kth.next：下一组的第一个节点
│                         反转时作为"终止边界"
```

**整体结构示意：**

```
dummy → [已翻转的组...] → prevGroup → [groupHead → ... → kth] → nextGroup → ...
                           ↑                        ↑            ↑
                      前一组尾              当前要翻转的组    下一组头
                                      （共 k 个节点）

翻转操作完成后：
dummy → [已翻转的组...] → prevGroup → [kth → ... → groupHead] → nextGroup → ...
                           ↑              ↑                     ↑
                      前一组尾    当前组的新头（原 kth）   下一组头
                           └── 连接 ──→┘←── 连接 ──┘
                           
然后 prevGroup 前进到 groupHead（即当前组的新尾），继续处理下一组
```

---

## 思路分析

### 解法一：迭代法（穿针引线）⭐ 推荐

**核心思路**：一趟扫描，每次处理一组。对于每组 k 个节点，执行标准链表反转（以 `nextGroup` 为终止边界），然后做组间连接。

```
算法框架：

1. 建 dummy 节点，prevGroup = dummy
2. 循环：
   a. 从 prevGroup 往后走 k 步，找到 kth
      ├── 走不到（碰到 null） → 结束，返回 dummy.next
      └── 走到了 → 继续
   b. 记录边界：
      groupHead = prevGroup.next   （反转前的头，反转后的尾）
      nextGroup = kth.next         （下一组的头，反转时的终止边界）
   c. 反转 [groupHead, kth] 这一段：
      以 nextGroup 为 prev 的初始值，标准反转直到 cur === nextGroup
   d. 连接：
      prevGroup.next = kth        （前一组尾 → 新头）
      prevGroup = groupHead       （更新 prevGroup 为当前组的尾，准备处理下一组）
3. 返回 dummy.next
```

> 💡 **关键洞察**：这个算法的本质是"每次处理一个 k 长度的窗口"。窗口内做标准反转，窗口前后用指针记录下来做连接。`prevGroup` 像是一个"锚点"，始终指向下一组的前一个节点。

### 解法二：递归法

**核心思路**：反转前 k 个节点 → 递归处理剩余链表 → 将当前组的尾（原 head）指向递归结果。

```
reverseKGroup(head, k):
  1. 从 head 往后找第 k 个节点 kth
     ├── 找不到 → 不足 k 个，直接返回 head（原样保留）
     └── 找到了 → 继续
  2. 记录 nextGroup = kth.next，作为反转终止边界
  3. 反转 [head, kth]（以 nextGroup 为终止边界）
  4. head.next = reverseKGroup(nextGroup, k)  ← 递归连接
  5. 返回 kth（新头）
```

> 💡 **递归法的优雅之处**：不需要显式维护 `prevGroup` 指针。每次递归返回的是"当前组的新头"，通过 `head.next = 递归结果` 自然完成组间连接。代码量少一半，但需要理解递归的"先反转当前组，再递归处理后面"的执行顺序。

### 两种方法对比

```
迭代法：自顶向下，逐组处理
  dummy → [处理组1] → [处理组2] → ... → 剩余节点

递归法：自底向上，从最后一组开始连接
  [处理组N] ← [处理组N-1] ← ... ← [处理组1] ← head

本质相同：都是"找到k个 → 反转 → 连接"，只是控制流不同
```

---

## 代码实现

### JavaScript 版本

#### 方法一：迭代法（穿针引线）⭐

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
 * @param {number} k
 * @return {ListNode}
 */
var reverseKGroup = function(head, k) {
    // 哑节点：统一处理头节点可能变化的情况
    const dummy = new ListNode(0, head);

    // prevGroup：始终指向「下一组的前一个节点」
    let prevGroup = dummy;

    while (true) {
        // 1. 检查剩余节点是否够 k 个：从 prevGroup 往后走 k 步
        let kth = prevGroup;
        for (let i = 0; i < k; i++) {
            kth = kth.next;
            if (!kth) {
                // 不足 k 个节点，直接返回
                return dummy.next;
            }
        }

        // 2. 记录当前组的边界
        const groupHead = prevGroup.next;   // 当前组的第一个节点（反转后变成尾）
        const nextGroup = kth.next;         // 下一组的第一个节点（反转的终止边界）

        // 3. 反转 [groupHead, kth] 这一段
        //    标准链表反转，以 nextGroup 为 prev 的初始值
        let prev = nextGroup;
        let cur = groupHead;
        while (cur !== nextGroup) {
            const next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        // 反转完成后，prev 就是反转后的新头（即原来的 kth）

        // 4. 连接：前一组尾 → 新头
        prevGroup.next = kth;  // kth 是反转后的新头

        // 5. 更新 prevGroup：移到当前组的尾（即原 groupHead），准备处理下一组
        prevGroup = groupHead;
        // groupHead.next 已经在反转时被设置为 nextGroup，连接已自动完成
    }
};
```

#### 方法二：递归法

```javascript
/**
 * @param {ListNode} head
 * @param {number} k
 * @return {ListNode}
 */
var reverseKGroup = function(head, k) {
    // 1. 找到第 k 个节点（从 head 算起）
    let kth = head;
    for (let i = 1; i < k; i++) {
        kth = kth?.next;
        if (!kth) {
            // 不足 k 个节点，原样返回
            return head;
        }
    }

    // 2. 记录下一组的起点，并作为反转的终止边界
    const nextGroup = kth.next;

    // 3. 反转 [head, kth] 这一段
    let prev = nextGroup;
    let cur = head;
    while (cur !== nextGroup) {
        const next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    // 反转后，prev = kth（新头），head 变成了当前组的尾

    // 4. 递归处理剩余链表，接到当前组的尾部
    head.next = reverseKGroup(nextGroup, k);

    // 5. 返回当前组的新头
    return kth;
};
```

### Python 版本

#### 方法一：迭代法（穿针引线）⭐

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

def reverseKGroup(head: ListNode, k: int) -> ListNode:
    """方法一：迭代法（穿针引线）"""
    # 哑节点：统一处理头节点可能变化的情况
    dummy = ListNode(0, head)

    # prev_group：始终指向「下一组的前一个节点」
    prev_group = dummy

    while True:
        # 1. 检查剩余节点是否够 k 个：从 prev_group 往后走 k 步
        kth = prev_group
        for _ in range(k):
            kth = kth.next
            if not kth:
                # 不足 k 个节点，直接返回
                return dummy.next

        # 2. 记录当前组的边界
        group_head = prev_group.next   # 当前组的第一个节点（反转后变成尾）
        next_group = kth.next          # 下一组的第一个节点（反转的终止边界）

        # 3. 反转 [group_head, kth] 这一段
        prev = next_group
        cur = group_head
        while cur is not next_group:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        # 反转完成后，prev 就是反转后的新头（即原来的 kth）

        # 4. 连接：前一组尾 → 新头
        prev_group.next = kth  # kth 是反转后的新头

        # 5. 更新 prev_group：移到当前组的尾，准备处理下一组
        prev_group = group_head
```

#### 方法二：递归法

```python
def reverseKGroup(head: ListNode, k: int) -> ListNode:
    """方法二：递归法"""
    # 1. 找到第 k 个节点（从 head 算起）
    kth = head
    for _ in range(1, k):
        kth = kth.next if kth else None
        if not kth:
            # 不足 k 个节点，原样返回
            return head

    # 2. 记录下一组的起点，并作为反转的终止边界
    next_group = kth.next

    # 3. 反转 [head, kth] 这一段
    prev = next_group
    cur = head
    while cur is not next_group:
        nxt = cur.next
        cur.next = prev
        prev = cur
        cur = nxt
    # 反转后，prev = kth（新头），head 变成了当前组的尾

    # 4. 递归处理剩余链表，接到当前组的尾部
    head.next = reverseKGroup(next_group, k)

    # 5. 返回当前组的新头
    return kth
```

---

## 逐步推演

### 迭代法完整推演（k=3）

以 `head = [1,2,3,4,5], k = 3` 为例，逐步展示指针变化：

```
初始状态:
  dummy → 1 → 2 → 3 → 4 → 5
  prevGroup = dummy
  ↑
  要处理第 1 组


═══════════════ 第 1 组 ═══════════════

步骤 1：找 kth（往后走 3 步）
  dummy → 1 → 2 → 3 → 4 → 5
    ↑                 ↑
  prevGroup          kth

步骤 2：记录边界
  dummy → 1 → 2 → 3 → 4 → 5
    ↑     ↑           ↑     ↑
  prevGroup groupHead kth  nextGroup

步骤 3：反转 [1,2,3]（以 nextGroup=4 为终止边界）

  初始:
    prev = 4, cur = 1
    链表: dummy → 1 → 2 → 3 → 4 → 5

  第 1 次迭代（处理节点 1）:
    next = cur.next = 2
    cur.next = prev  →  1 → 4
    prev = cur = 1
    cur = next = 2
    当前: 1 → 4 → 5

  第 2 次迭代（处理节点 2）:
    next = cur.next = 3
    cur.next = prev  →  2 → 1
    prev = cur = 2
    cur = next = 3
    当前: 2 → 1 → 4 → 5

  第 3 次迭代（处理节点 3）:
    next = cur.next = 4
    cur.next = prev  →  3 → 2
    prev = cur = 3
    cur = next = 4
    当前: 3 → 2 → 1 → 4 → 5

  cur === nextGroup (4)，退出循环
  反转结果: prev = 3（新头）

步骤 4：连接 prevGroup → 新头
  prevGroup.next = kth  →  dummy → 3 → 2 → 1 → 4 → 5

步骤 5：更新 prevGroup = groupHead（即节点 1）
  dummy → 3 → 2 → 1 → 4 → 5
                 ↑
              prevGroup（第 2 组的前驱）


═══════════════ 第 2 组 ═══════════════

步骤 1：找 kth（从 prevGroup=1 往后走 3 步）
  dummy → 3 → 2 → 1 → 4 → 5
                 ↑         ↑
              prevGroup   (走第 3 步时为 null！)

  kth 在第 3 步时变成了 null → 不足 k=3 个节点

结束！返回 dummy.next = 3

最终链表: 3 → 2 → 1 → 4 → 5 ✅
```

### 递归法完整推演（k=2）

以 `head = [1,2,3,4,5], k = 2` 为例：

```
reverseKGroup([1,2,3,4,5], k=2)

第 1 层调用（head=1）:
  步骤 1：找 kth（走 k-1=1 步）→ kth = 2
  步骤 2：nextGroup = 3
  步骤 3：反转 [1,2]（终止边界=3）
    初始: prev=3, cur=1
    → 迭代1: cur=1, next=2, 1→3, prev=1, cur=2
    → 迭代2: cur=2, next=3, 2→1, prev=2, cur=3
    → cur===nextGroup，退出
    结果: 2 → 1 → 3 → 4 → 5，新头 kth=2
  步骤 4：递归调用 reverseKGroup(3, 2)

    第 2 层调用（head=3）:
      步骤 1：找 kth（走 1 步）→ kth = 4
      步骤 2：nextGroup = 5
      步骤 3：反转 [3,4]（终止边界=5）
        结果: 4 → 3 → 5，新头 kth=4
      步骤 4：递归调用 reverseKGroup(5, 2)

        第 3 层调用（head=5）:
          步骤 1：找 kth（走 1 步）→ 5.next = null，kth 走不到
          不足 2 个 → 返回 head = 5

      回溯第 2 层:
        head.next = 第 3 层结果 = 5
        链表: 4 → 3 → 5
        返回 kth = 4

  回溯第 1 层:
    head.next = 第 2 层结果 = 4 → 3 → 5
    链表: 2 → 1 → 4 → 3 → 5
    返回 kth = 2

最终链表: 2 → 1 → 4 → 3 → 5 ✅
```

### 迭代法推演（k=2，展示多组连接）

以 `head = [1,2,3,4,5], k = 2`，关注组间连接：

```
初始:
  dummy → 1 → 2 → 3 → 4 → 5
  prevGroup = dummy

第 1 组 [1,2]:
  找 kth: dummy → 1 → 2（走了 2 步，到达 2 ✓）
  边界: groupHead=1, nextGroup=3
  反转 [1,2]（终止=3）→ 2 → 1 → 3
  连接: dummy → 2 → 1 → 3 → 4 → 5
  更新: prevGroup = 1

第 2 组 [3,4]:
  找 kth: 1 → 3 → 4（走了 2 步，到达 4 ✓）
  边界: groupHead=3, nextGroup=5
  反转 [3,4]（终止=5）→ 4 → 3 → 5
  连接: 1 → 4 → 3 → 5
  链表: dummy → 2 → 1 → 4 → 3 → 5
  更新: prevGroup = 3

第 3 组 [5]:
  找 kth: 3 → 5 → ?（第 2 步，5.next = null ✗）
  不足 k=2 个 → 结束

最终链表: 2 → 1 → 4 → 3 → 5 ✅
```

> 💡 **核心观察**：每次反转后，`prevGroup` 从原位置"跨越"当前组，落到当前组的尾节点上。这个尾节点恰好是下一组的前驱——完美衔接。整个算法不需要显式维护"下一个 prevGroup 在哪"，因为反转操作本身就保证了这一点。

---

## 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| **迭代法** ⭐ | O(n) | O(1) | 一趟扫描，空间最优，变量少易调试 | 需要显式维护 `prevGroup` 指针 |
| **递归法** | O(n) | O(n/k) ≈ O(n) | 代码最优雅，组间连接自然 | 递归栈占用，链表过长时有栈溢出风险 |

> - **时间复杂度**：每个节点恰好被访问常数次（遍历找 kth 一次，反转时再访问一次），因此是 O(n)
> - **空间复杂度**：迭代法只用 3-4 个指针变量，O(1)；递归法栈深度为 ⌈n/k⌉，最坏 O(n)
> - **递归法空间分析**：虽然递归栈深度是 O(n/k) 而非 O(n)，但这只是上界的常数倍差异。当 k=1 时就是 O(n)，当 k=n 时是 O(1)。通常按最坏情况记为 O(n)

---

## 举一反三

### 本题在链表反转体系中的位置

```
LeetCode 206 反转整个链表（Easy）
  ↓ 加入局部边界
LeetCode 92 翻转链表 II（Medium）— 反转 [left, right] 一段
  ↓ 加入多段分组
LeetCode 25 K 个一组翻转链表（Hard）— 每 k 个一段，多段反转
  ↓ 加入"不足 k 个也从尾部开始凑"
LeetCode 2074 反转偶数长度组（Medium）— 组长度递增，仅反转偶数长度组
```

### 关联题目

| 题目 | 核心思想 | 与本题的关系 |
|------|---------|-------------|
| [206. 反转链表](https://leetcode.com/problems/reverse-linked-list/) | 标准链表反转（迭代/递归） | 本题每组的反转就是 206 的"限长版" |
| [92. 翻转链表 II](./leetcode-92-reverse-linked-list-ii.md) | 头插法、分段反转、递归回溯 | 本题的前置——局部反转，K 组是它的多段版 |
| [24. 两两交换链表中的节点](https://leetcode.com/problems/swap-nodes-in-pairs/) | 相邻节点交换 | 本题 k=2 的特例，但可用更简洁的解法 |
| [2074. 反转偶数长度组](https://leetcode.com/problems/reverse-nodes-in-even-length-groups/) | 变长分组 + 条件反转 | 本题的变体——组长度递增，只反转偶数长度 |
| [1721. 交换链表中的节点](https://leetcode.com/problems/swapping-nodes-in-a-linked-list/) | 双指针定位 + 值交换 | 同样涉及 k 参数，但只需交换两个节点的值 |

### 链表分组反转通用框架

反复出现的模式提炼：

```
链表分组反转 = 循环执行以下三步：

1. 定位：确定当前 k 个节点的起止边界
   - 头: prevGroup.next（或 head）
   - 尾: 往后走 k 步到达的 kth
   - 下一组起点: kth.next

2. 反转：以"下一组起点"为终止边界，标准反转当前 k 个节点
   - 这是 LeetCode 206 的限长版本
   - 终止边界（而不是 null）确保反转不会"跑出组"

3. 连接：两个连接点
   - 前驱 → 新头（原 kth）：prevGroup.next = kth
   - 旧头 → 下一组（nextGroup）：groupHead.next 已在反转中设置好
   
4. 移动：prevGroup 前进 k 步（即落到原 groupHead），准备处理下一组
```

> **一句话总结**：K 个一组翻转链表 = 对每一组执行 LeetCode 206（以 nextGroup 为边界），然后用 LeetCode 92 的"穿针引线"思路连接各组。迭代法控制流清晰，递归法代码简洁——本质上是一样的"定位→反转→连接→前移"四步循环。

### 延伸思考

在实际面试中，如果你能用**迭代法**清晰写出这道题，面试官往往会追问：

1. **"可以只用 O(1) 额外空间吗？"** → 迭代法已经是 O(1)，解释清楚即可
2. **"递归法在 k 很大时有什么问题？"** → 栈溢出风险，迭代法更稳健
3. **"如果要求不足 k 个也从尾部开始凑呢？"** → 需要先计算链表长度，从尾部往前推。这是 LeetCode 2074 的变体
4. **"你能把反转逻辑提取成一个独立函数吗？"** → 可以，将 `reverseList(start, endNext)` 封装——返回新头，这样主逻辑更清晰

---

## 总结

| 题目 | 难度 | 核心思想 | 推荐解法 |
|------|------|---------|---------|
| **25 K 个一组翻转链表** | 🔴 Hard | 分组 + 穿针引线、递归自底向上 | 迭代法 |

这道题的"难"不在于单个操作复杂，而在于**同时维护多个指针**。只要把握住四个关键变量，问题就迎刃而解：

| 变量 | 含义 | 生命周期 |
|------|------|---------|
| `prevGroup` | 当前组的前驱节点 | 跨组移动，每处理完一组前进 k 步 |
| `kth` | 当前组的第 k 个节点 | 每组临时计算，用于判断是否够 k 个 |
| `groupHead` | 当前组反转前的首节点（反转后变尾） | 每组临时记录，用于更新 prevGroup |
| `nextGroup` | 下一组的首节点（反转的终止边界） | 每组临时记录，作为反转函数的 stop 条件 |

**两组解法各有千秋：**

1. **迭代法（推荐）**：O(1) 空间，一趟扫描。核心口诀：**prevGroup 锚定前驱，每次往后数 k 个，够就反转，不够就停。反转以 nextGroup 为墙，反转完 prevGroup 跳到原 groupHead。** 面试中最稳健的解法，没有递归栈溢出的风险。

2. **递归法**：代码精炼，自然的"分治"思维——反转当前 k 个，递归处理后面。核心口诀：**找到 kth，反转前 k 个，head 变尾连递归结果，返回 kth。** 如果面试官追问"能不能写得更简洁"，这就是答案。

> **记住一句话：K 个一组翻转链表 = 对每组做"以 nextGroup 为终止边界的标准反转"，然后用 prevGroup 把各组串起来。四个变量（prevGroup, kth, groupHead, nextGroup）搞清楚了，这道 Hard 题就是纸老虎。**

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 206. 反转链表](https://leetcode.com/problems/reverse-linked-list/)（基础反转） | [LeetCode 92. 翻转链表 II](./leetcode-92-reverse-linked-list-ii.md)（局部反转） | [LeetCode 24. 两两交换链表中的节点](https://leetcode.com/problems/swap-nodes-in-pairs/)（k=2 特例） | 链表系列持续更新中，关注不迷路。
