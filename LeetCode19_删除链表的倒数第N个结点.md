# LeetCode 19. 删除链表的倒数第 N 个结点

## 一、问题描述

题目链接：[LeetCode 19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

给定一个链表的头结点 `head` 和整数 `n`，删除链表的倒数第 `n` 个结点，并返回删除后的链表头结点。

要求尽可能通过**一次遍历**完成删除。

**示例：**

```text
输入：head = [1,2,3,4,5], n = 2

原链表：
1 → 2 → 3 → 4 → 5
            ↑
        倒数第 2 个

删除后：
1 → 2 → 3 → 5

输出：[1,2,3,5]
```

**注意：**

- 链表结点数量至少为 1。
- `1 <= n <= 链表长度`。
- 待删除结点可能是头结点，需要正确处理该情况。

## 二、算法思路

### 2.1 双指针：利用固定距离定位目标

**基本思路：** 删除倒数第 `n` 个结点，需要先定位其前驱结点。通过维护两个指针之间的固定距离，将依赖链表总长度的位置计算转化为双指针的相对位置关系，从而实现一次遍历。

**（1）建立固定距离关系**

引入虚拟头结点 `dummy`，使其指向 `head`，并将快慢指针 `fast`、`slow` 初始化为 `dummy`。

首先让 `fast` 提前移动 `n+1` 步，使两个指针之间保持固定距离：

```math
distance(slow,fast)=n+1
```

例如，`n = 2`：

```text
slow          fast
 ↓             ↓
dummy → 1 → 2 → 3 → 4 → 5 → None
       1    2    3  （fast 提前走 3 步）
```

此时 `slow` 位于 `dummy`，`fast` 位于结点 `3`。

**（2）同步移动定位目标**

接下来让两个指针同步移动，每次各前进一步，始终保持 `n+1` 步的距离：

```text
n = 2

同步移动前：slow = dummy，fast = 3
第 1 步：   slow = 1，    fast = 4
第 2 步：   slow = 2，    fast = 5
第 3 步：   slow = 3，    fast = None

dummy → 1 → 2 → 3 → 4 → 5 → None
                ↑   ↑
              slow  待删除结点
```

当 `fast` 到达 `None` 时，`slow` 恰好位于倒数第 `n` 个结点的前驱。

**为什么能够准确定位？**

设链表长度为 $ `L` $。从 `dummy` 到 `None` 共需走 $ `L+1` $ 步。由于 `fast` 始终领先 `slow` 共 $ `n+1` $ 步，终止时 `slow` 从 `dummy` 走过：

```math
(L+1)-(n+1)=L-n
```

这恰好是待删除结点前驱的位置。

**核心思想：利用固定距离关系，将需要预先知道链表长度的全局定位问题，转化为两个指针同步移动的局部问题。**

**（3）删除目标结点**

当 `fast` 到达 `None` 时，`slow.next` 就是待删除结点。执行：

```python
slow.next = slow.next.next
```

即可完成删除，最后返回 `dummy.next`。

虚拟头结点使删除原头结点的情况也能使用相同操作，无需额外分支。

### 2.2 复杂度分析

设链表长度为 $ `L` $。

- **时间复杂度：** $ `O(L)` $。两个指针都只向前移动，整体为线性时间。
- **空间复杂度：** $ `O(1)` $。仅使用虚拟头结点和常数个指针。

**算法思想总结：** 当目标位置与链表末尾之间存在固定距离关系时，可以让两个指针保持固定间距，并利用其中一个指针到达末尾这一终止条件，直接确定另一个指针的位置，避免预先计算链表长度。

## 三、算法实现

采用**固定距离双指针**，利用虚拟头结点统一处理删除操作。

```python
class Solution:
    def removeNthFromEnd(
        self, head: ListNode | None, n: int
    ) -> ListNode | None:
        dummy = ListNode(0, head)
        slow = fast = dummy

        # 1. fast 提前移动 n+1 步
        for _ in range(n + 1):
            fast = fast.next

        # 2. 同步移动，保持固定距离
        while fast is not None:
            slow = slow.next
            fast = fast.next

        # 3. 删除目标结点
        slow.next = slow.next.next

        return dummy.next
```

## 四、正确性证明

采用**循环不变量法**，证明双指针能够正确定位待删除结点的前驱。

### 4.1 循环不变量

在双指针同步移动过程中，始终满足：

```math
distance(slow,fast)=n+1
```

即 `fast` 始终领先 `slow` 共 `n+1` 步。

### 4.2 初始化

两个指针均从 `dummy` 出发，`fast` 先移动 `n+1` 步，因此循环不变量成立。

### 4.3 保持

每轮循环中，`fast` 和 `slow` 同时前进一步，两者的距离保持不变，因此循环不变量继续成立。

### 4.4 终止

设链表长度为 $ `L` $。

当 `fast` 到达 `None` 时，从 `dummy` 到 `None` 共经过 $ `L+1` $ 步。

由于 `fast` 领先 `slow` 共 `n+1` 步，因此 `slow` 从 `dummy` 出发走过：

```math
(L+1)-(n+1)=L-n
```

恰好位于倒数第 `n` 个结点的前驱。执行 `slow.next = slow.next.next` 能够正确删除目标结点，虚拟头结点保证删除头结点时同样成立。

因此，算法正确。
