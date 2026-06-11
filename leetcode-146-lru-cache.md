---
title: LeetCode 146. LRU 缓存
tags: [设计, 哈希表, 双向链表, 数据结构]
difficulty: Medium
category: 设计
date: 2026-06-11
---

# LeetCode 146. LRU 缓存 —— 亲手造一个 O(1) 的缓存淘汰算法

## 前言

你有没有想过：当你打开浏览器、刷短视频、使用数据库时，那些"最近经常访问"的数据为什么能瞬间返回？答案藏在操作系统、Redis、MySQL、浏览器、CDN 都在用的一个经典算法里——**LRU（Least Recently Used，最近最少使用）**。

LRU 的核心思想朴素但精妙：**如果数据最近被访问过，那么将来被访问的几率也更高**。基于这个思想，当缓存空间不够时，优先淘汰最久没被访问的数据。

LeetCode 146 让你亲手实现一个 LRU 缓存。这不仅是面试超高频题（字节、阿里、腾讯几乎必考），更是理解缓存设计的绝佳入口。本文将带你从需求出发，一步步推导出 HashMap + 双向链表的经典组合，并给出 JS 与 Python 双版本代码。

---

## 问题描述

### LeetCode 146. LRU Cache（LRU 缓存）

> 请你设计并实现一个满足 **LRU（最近最少使用）缓存** 约束的数据结构。
>
> 实现 `LRUCache` 类：
>
> - `LRUCache(int capacity)` 以**正整数**作为容量 `capacity` 初始化 LRU 缓存
> - `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1`
> - `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value`；如果不存在，则向缓存中插入该组 `key-value`。如果插入操作导致关键字数量超过 `capacity`，则应该**逐出**最久未使用的关键字
>
> 函数 `get` 和 `put` 必须以 **O(1)** 的平均时间复杂度运行。

**示例：**

```
输入：
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]

输出：
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释：
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1，缓存变为 {2=2, 1=1}（1 被"最近使用"）
lRUCache.put(3, 3); // 缓存满了，逐出 key=2，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1（未找到）
lRUCache.put(4, 4); // 缓存满了，逐出 key=1，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1（未找到）
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

---

## 核心思想

### 需求拆解

题目要求 `get` 和 `put` 都是 **O(1)** 时间复杂度。我们来逐一分析：

1. **O(1) 查找** → 必须用**哈希表**（HashMap / Map / dict）
2. **O(1) 插入** → 哈希表可以，但还需要维护"访问顺序"
3. **O(1) 删除** → 哈希表可以 O(1) 删除，但需要知道"最久未使用的 key"
4. **O(1) 调整顺序** → 当某个 key 被访问时，需要把它挪到"最近使用"的位置

### 为什么需要双向链表？

哈希表虽然能做到 O(1) 查找和插入，但它**不记录顺序**——你不知道哪个 key 最久没被访问。

数组/队列能记录顺序，但把中间某个元素挪到末尾是 O(n)。

**双向链表**完美解决了这个问题：

- 表头 → 最早使用（最久没访问），待淘汰
- 表尾 → 最近使用，当前活跃
- 每次 `get` 或 `put` 已存在的 key → 把该节点移到链表尾部（标记为"最近使用"）
- 缓存满了要逐出 → 删除链表头节点（最久未使用）

```
双向链表结构示意（capacity = 3）：

   HEAD（哨兵）                        TAIL（哨兵）
      ⇄          ⇄          ⇄
   [k1,v1]    [k2,v2]    [k3,v3]
   最早使用                最近使用

get(k1) 之后 → 把 [k1,v1] 移到尾部：

   HEAD（哨兵）                        TAIL（哨兵）
      ⇄          ⇄          ⇄
   [k2,v2]    [k3,v3]    [k1,v1]
   最早使用                最近使用（刚访问过）

put(k4,v4) → 缓存满了，删除头部 [k2,v2]，插入 [k4,v4] 到尾部：

   HEAD（哨兵）                        TAIL（哨兵）
      ⇄          ⇄          ⇄
   [k3,v3]    [k1,v1]    [k4,v4]
   最早使用                最近使用
```

### 数据结构组合

最终的方案是 **HashMap + 双向链表**：

```
┌─────────────────────────────────────┐
│              哈希表                   │
│  key → 链表节点指针（O(1) 定位）      │
│                                      │
│  Map {                              │
│    1 → Node(1,1)                    │
│    2 → Node(2,2)  ←── 每个 value    │
│    3 → Node(3,3)      是一个节点引用 │
│  }                                  │
└─────────────────────────────────────┘
          │           │           │
          ▼           ▼           ▼
    ┌─ HEAD(哨兵) ⇄ Node(1,1) ⇄ Node(2,2) ⇄ Node(3,3) ⇄ TAIL(哨兵) ─┐
    └────────────────── 双向链表（维护访问顺序）───────────────────────┘
```

- **哈希表**负责 O(1) 查找：给定 key，瞬间找到链表节点
- **双向链表**负责 O(1) 维护顺序：增删节点、挪到尾部，都是指针操作

---

## 思路分析

### 核心操作拆解

围绕 HashMap + 双向链表，`get` 和 `put` 可以拆解为几个原子操作：

#### 1. `get(key)` — O(1)

```
1. 查哈希表：key 存在？
   ├── 不存在 → 返回 -1
   └── 存在 → 获取节点
         2. 将该节点移到链表尾部（标记为"最近使用"）
         3. 返回节点的 value
```

#### 2. `put(key, value)` — O(1)

```
1. 查哈希表：key 已存在？
   ├── 存在 → 更新节点的 value，将该节点移到链表尾部
   └── 不存在 →
         2. 缓存已满？
            ├── 是 → 删除链表头节点（最久未使用），从哈希表中移除对应 key
            └── 否 → 跳过
         3. 创建新节点，加入链表尾部，加入哈希表
```

#### 3. 四个原子方法

为了代码清晰，我们提取四个辅助方法：

| 方法 | 作用 | 复杂度 |
|------|------|--------|
| `addToTail(node)` | 将节点添加到链表尾部（标记为最近使用） | O(1) |
| `removeNode(node)` | 从链表中删除一个节点（断开前后指针） | O(1) |
| `moveToTail(node)` | 将节点移到尾部 = `removeNode` + `addToTail` | O(1) |
| `removeHead()` | 删除链表头节点（缓存满时淘汰最久未使用） | O(1) |

### 哨兵节点技巧

使用 **head 哨兵** 和 **tail 哨兵** 可以省去大量空指针判断：

```
没有哨兵：每次操作都要判断「前驱/后继是否为 null」
有哨兵：链表永远非空，操作统一

哨兵初始化：
  head ⇄ tail
  （空的，head.next === tail, tail.prev === head）
```

哨兵节点不存储实际数据，它们在链表两端作为"岗哨"，让所有插入删除操作都不需要判空。

---

## 代码实现

### JavaScript 版本

```javascript
/**
 * 双向链表节点
 */
class ListNode {
    constructor(key, value) {
        this.key = key;
        this.value = value;
        this.prev = null;
        this.next = null;
    }
}

/**
 * @param {number} capacity
 */
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.map = new Map(); // key → ListNode

    // 哨兵节点：head 和 tail 不存数据，简化边界处理
    this.head = new ListNode(0, 0);
    this.tail = new ListNode(0, 0);
    this.head.next = this.tail;
    this.tail.prev = this.head;
};

/**
 * @param {number} key
 * @return {number}
 */
LRUCache.prototype.get = function(key) {
    if (!this.map.has(key)) {
        return -1;
    }
    const node = this.map.get(key);
    // 每次访问，将该节点移到链表尾部（最近使用）
    this._moveToTail(node);
    return node.value;
};

/**
 * @param {number} key
 * @param {number} value
 * @return {void}
 */
LRUCache.prototype.put = function(key, value) {
    if (this.map.has(key)) {
        // key 已存在：更新值 + 移到尾部
        const node = this.map.get(key);
        node.value = value;
        this._moveToTail(node);
    } else {
        // key 不存在：新建节点
        const newNode = new ListNode(key, value);
        this.map.set(key, newNode);
        this._addToTail(newNode);

        // 超过容量：淘汰最久未使用的（链表头部）
        if (this.map.size > this.capacity) {
            const removed = this._removeHead();
            this.map.delete(removed.key);
        }
    }
};

// ─── 四个原子操作 ──────────────────────────

/** 将节点添加到链表尾部 */
LRUCache.prototype._addToTail = function(node) {
    node.prev = this.tail.prev;
    node.next = this.tail;
    this.tail.prev.next = node;
    this.tail.prev = node;
};

/** 从链表中移除指定节点 */
LRUCache.prototype._removeNode = function(node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
};

/** 将节点移到尾部 = 移除 + 添加到尾部 */
LRUCache.prototype._moveToTail = function(node) {
    this._removeNode(node);
    this._addToTail(node);
};

/** 移除链表头节点（最久未使用），返回被移除的节点 */
LRUCache.prototype._removeHead = function() {
    const node = this.head.next;
    this._removeNode(node);
    return node;
};
```

```python
class ListNode:
    """双向链表节点"""
    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None


class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.map = {}  # key → ListNode

        # 哨兵节点：head 和 tail 不存数据，简化边界处理
        self.head = ListNode()
        self.tail = ListNode()
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key not in self.map:
            return -1
        node = self.map[key]
        # 每次访问，将该节点移到链表尾部（最近使用）
        self._move_to_tail(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        if key in self.map:
            # key 已存在：更新值 + 移到尾部
            node = self.map[key]
            node.value = value
            self._move_to_tail(node)
        else:
            # key 不存在：新建节点
            node = ListNode(key, value)
            self.map[key] = node
            self._add_to_tail(node)

            # 超过容量：淘汰最久未使用的（链表头部）
            if len(self.map) > self.capacity:
                removed = self._remove_head()
                del self.map[removed.key]

    # ─── 四个原子操作 ──────────────────────────

    def _add_to_tail(self, node: ListNode) -> None:
        """将节点添加到链表尾部"""
        node.prev = self.tail.prev
        node.next = self.tail
        self.tail.prev.next = node
        self.tail.prev = node

    def _remove_node(self, node: ListNode) -> None:
        """从链表中移除指定节点"""
        node.prev.next = node.next
        node.next.prev = node.prev

    def _move_to_tail(self, node: ListNode) -> None:
        """将节点移到尾部 = 移除 + 添加到尾部"""
        self._remove_node(node)
        self._add_to_tail(node)

    def _remove_head(self) -> ListNode:
        """移除链表头节点（最久未使用），返回被移除的节点"""
        node = self.head.next
        self._remove_node(node)
        return node
```

### 逐步推演

以 `capacity = 2` 为例，逐步推演示例的完整过程：

```
初始状态（capacity = 2）：
  map = {}
  head ⇄ tail

put(1, 1):
  map = {1 → Node(1,1)}
  head ⇄ Node(1,1) ⇄ tail

put(2, 2):
  map = {1 → Node(1,1), 2 → Node(2,2)}
  head ⇄ Node(1,1) ⇄ Node(2,2) ⇄ tail
  最近使用 [2,2]（在尾部）

get(1):
  命中 Node(1,1) → 移到尾部
  map 不变
  head ⇄ Node(2,2) ⇄ Node(1,1) ⇄ tail
  最近使用 [1,1]，最久未使用 [2,2]

put(3, 3):
  key=3 不在 map 中，且 map.size === capacity
  → 删除头部 Node(2,2)，map 移除 key=2
  → 创建 Node(3,3) 加入尾部
  map = {1 → Node(1,1), 3 → Node(3,3)}
  head ⇄ Node(1,1) ⇄ Node(3,3) ⇄ tail

get(2):
  map 中无 key=2 → 返回 -1

put(4, 4):
  key=4 不在 map 中，且 map.size === capacity
  → 删除头部 Node(1,1)，map 移除 key=1
  → 创建 Node(4,4) 加入尾部
  map = {3 → Node(3,3), 4 → Node(4,4)}
  head ⇄ Node(3,3) ⇄ Node(4,4) ⇄ tail

get(1): → 返回 -1（已被淘汰）
get(3): → 返回 3，并把 Node(3,3) 移到尾部
  head ⇄ Node(4,4) ⇄ Node(3,3) ⇄ tail
get(4): → 返回 4，并把 Node(4,4) 移到尾部
  head ⇄ Node(3,3) ⇄ Node(4,4) ⇄ tail
```

---

## 复杂度分析

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间复杂度 — `get` | O(1) | 哈希表查找 O(1) + 链表移动 O(1) |
| 时间复杂度 — `put` | O(1) | 哈希表插入/更新 O(1) + 链表插入/删除 O(1) |
| 空间复杂度 | O(capacity) | 哈希表最多存 capacity 个 key，链表最多 capacity 个节点 |

每个操作都是严格的 O(1)，不依赖于缓存中的数据量。

---

## 优化方向

### 1. 使用语言内置的有序字典

在实际工程中（特别是面试），可以先用语言自带的有序结构简化实现：

**JavaScript — `Map` 本身就是有序的（按插入顺序迭代）：**

```javascript
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.map = new Map();
};

LRUCache.prototype.get = function(key) {
    if (!this.map.has(key)) return -1;
    const val = this.map.get(key);
    // 删除后重新插入 → 该 key 变为"最近使用"
    this.map.delete(key);
    this.map.set(key, val);
    return val;
};

LRUCache.prototype.put = function(key, value) {
    if (this.map.has(key)) {
        this.map.delete(key);
    } else if (this.map.size >= this.capacity) {
        // Map 的 keys().next() 返回"最先插入的 key"（最久未使用）
        const oldestKey = this.map.keys().next().value;
        this.map.delete(oldestKey);
    }
    this.map.set(key, value);
};
```

> **面试提醒**：虽然 JS 的 `Map` 本身有序，但面试官通常期望你展示"双向链表 + HashMap"的底层实现。建议先用内置 `Map` 快速写出正确解，再展开手写双向链表版本——体现"先会用，再会造"的深度。

**Python — `collections.OrderedDict`：**

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)  # 移到末尾（最近使用）
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)  # 弹出第一个（最久未使用）
```

### 2. 避免重复代码（DRY）

`get` 和 `put` 都需要"访问后移到尾部"，可以统一为 `_touch(node)` 方法：

```javascript
LRUCache.prototype._touch = function(node) {
    this._removeNode(node);
    this._addToTail(node);
};

// get 和 put 中统一调用 this._touch(node)
```

### 3. 为什么不用单向链表？

单向链表只能从头到尾遍历，无法 O(1) 删除中间节点（因为不知道前驱是谁）。双向链表既然能往前找，也能往后找，删除中间节点只需修改前驱和后继的指针，是 O(1)。

---

## 举一反三

理解了"哈希表 + 双向链表"的组合模式，以下变种题都能迎刃而解：

| 题目 | 关键点 |
|------|--------|
| **LeetCode 146. LRU 缓存** | 经典 LRU，HashMap + 双向链表 |
| **LeetCode 460. LFU 缓存** | 基于"访问频次"淘汰，需要 HashMap + 频次双向链表，比 LRU 多一层 |
| **LeetCode 432. 全 O(1) 的数据结构** | 支持 O(1) 插入、删除、获取最大值/最小值，频次桶 + 双向链表 |
| **LeetCode 716. 最大栈** | 双栈/双向链表 + 平衡树，支持 O(1) 获取最大值 |
| **LeetCode 1465. 切割后面积最大的蛋糕** | 排序 + 找最大间隔（非 LRU，但考察缓存思想） |
| **LeetCode 1797. 设计一个验证系统** | 基于过期时间的 token 管理，类似 LRU 的时间维度变种 |

它们共享同一个思维模式：

```
需要 O(1) CRUD + 维护某种顺序（时间/频次）
      ↓
HashMap（O(1) 定位） + 双向链表（O(1) 调整顺序）
      ↓
哨兵节点优化边界操作
```

### 进阶思考

以下是几个真实的 LRU 应用场景，方便你在面试中和面试官展开讨论：

- **操作系统页面置换**：当物理内存不足时，操作系统需要换出一些内存页面。LRU 是最经典的页面置换算法之一，但实际 OS 更多使用"时钟算法"（Clock Algorithm）——因为硬件支持的 LRU 开销太大。
- **Redis 缓存淘汰**：Redis 提供了多种淘汰策略（`maxmemory-policy`），`allkeys-lru` 就是基于 LRU 的。不过 Redis 使用的是**近似 LRU**——抽样 N 个 key，淘汰其中最久未使用的，而不是维护完整的双向链表（节省内存）。
- **MySQL Buffer Pool**：InnoDB 使用 **LRU 的改进版**——将链表分为"热区"和"冷区"，防止一次全表扫描把真正的热数据全部挤出缓存。

---

## 总结

LRU 缓存这道题的精髓在于**数据结构组合**：

1. **哈希表**负责 O(1) 查找——给定 key 瞬间找到节点
2. **双向链表**负责 O(1) 维护顺序——表头最久没用，表尾最近使用
3. **哨兵节点**消除边界判断——让插入和删除逻辑统一，不用判空

三个关键细节值得反复体会：

- **`moveToTail` 是两个原子操作的组合**：先从链表中摘除节点（`removeNode`），再接到尾部（`addToTail`）。把复杂操作拆成原子的增删，代码会清爽很多
- **节点中要存 key**：很多人只存 `value`，但淘汰链表头部时，你需要知道被淘汰的 key 是什么，才能从哈希表中删掉它。节点中的 `key` 就是为这个场景准备的
- **哨兵不存数据**：`head` 和 `tail` 只是占位符，永远不参与淘汰。真正的数据节点夹在两个哨兵之间

面试中建议这样展示：

1. 先说清楚"为什么单独一个 HashMap 或单独一个链表都不够"（一个缺顺序，一个缺 O(1) 查找）
2. 画出示意图：哈希表指向链表节点，链表维护顺序
3. 先用语言内置的 `Map` / `OrderedDict` 快速写出正确版本，再展开手写双向链表——体现深度
4. 如果能说出 Redis 的近似 LRU 和 MySQL Buffer Pool 的冷热分离，就是满分级别的拓展

建议在纸上手动模拟一遍 `capacity=2` 的 `put(1,1) → put(2,2) → get(1) → put(3,3)` 过程——把每一步的链表结构和 map 内容画出来。图一画，豁然开朗。

---

> **关于作者**：LeetCode 刷题中，致力于用最清晰的方式讲透算法题。欢迎在评论区交流讨论！

> **相关题解**：[LeetCode 460. LFU 缓存](https://leetcode.cn/problems/lfu-cache/) | [LeetCode 432. 全 O(1) 的数据结构](https://leetcode.cn/problems/all-oone-data-structure/) | 后续会继续更新设计类数据结构系列，关注不迷路。
