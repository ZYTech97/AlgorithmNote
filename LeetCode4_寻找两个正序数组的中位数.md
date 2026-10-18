# LeetCode 4. 寻找两个正序数组的中位数：二分查找划分边界

## 一、问题描述

**题目链接**：[LeetCode 4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)

给定两个非递减排序的整数数组 `nums1` 和 `nums2`，长度分别为 `m` 和 `n`，求两个数组合并后的中位数。

**输入限制：**

- `0 <= m, n <= 1000`
- `1 <= m + n <= 2000`
- `-10⁶ <= nums1[i], nums2[i] <= 10⁶`
- 两个数组均按非递减顺序排列。

**复杂度要求：** 时间复杂度为 $O(\log(m+n))$。

## 二、基本思路

### 2.1 整体思路

将两个有序数组分别划分为左右两部分，使得**左半部分的元素数量符合中位数要求，且左半部分所有元素均不大于右半部分**。随后直接根据划分边界计算中位数。

```text
                   左半部分 │ 右半部分
nums1:            1  2  6 │ 8  10          i = 3
nums2:            3  4  5 │ 7  9  11  12   j = 3
                           │
左半部分最大值 = 6，右半部分最小值 = 7
中位数 = (6 + 7) / 2 = 6.5
```

*图 1：两个有序数组的正确划分。竖线表示划分边界。*

### 2.2 正确划分的条件

设两个数组长度分别为 $m,n$，划分位置为 $i,j$，其中 $0\le i\le m$、$0\le j\le n$。为保证左半部分包含一半元素（总数为奇数时多一个），要求：

$$
i+j=\left\lfloor\frac{m+n+1}{2}\right\rfloor
$$

由于数组本身有序，只需满足两个交叉条件：

$$
\begin{aligned}
A(i):&\quad nums1[i-1]\le nums2[j]\\
B(i):&\quad nums2[j-1]\le nums1[i]
\end{aligned}
$$

**正确划分一定存在**：将两个数组的全部元素按大小排列，在中间切分，再将切分位置映射回各自数组。由于原数组有序，这样得到的 $i,j$ 必然满足上述条件。

划分位置允许位于数组两端，对应两种特殊情况：

**情况一：左半部分为空（`i=0`）**

当 `nums1` 的元素整体较大时，正确划分可能需要将其全部元素放入右半部分。此时令 `nums1[i-1] = −∞`，使条件 A 自动成立。

```text
                   左半部分 │ 右半部分
nums1:                     │ 8  9          i = 0
nums2:            1  2  3 │ 4             j = 3
```

*图 2：`nums1` 的左半部分为空。*

**情况二：右半部分为空（`i=m`）**

当 `nums1` 的元素整体较小时，正确划分可能需要将其全部元素放入左半部分。此时令 `nums1[i] = +∞`，使条件 B 自动成立。

```text
                   左半部分 │ 右半部分
nums1:            1  2     │               i = m = 2
nums2:            3        │ 4  5  6       j = 1
```

*图 3：`nums1` 的右半部分为空。*

`nums2` 的边界 `j=0` 和 `j=n` 同理处理。使用正负无穷大作为边界哨兵，可以统一处理普通划分与边界划分。

为保证所有 $i\in[0,m]$ 对应的 $j$ 均合法，先交换数组（如有必要），使 $m\le n$。

### 2.3 二分查找划分边界

令：

$$
j=\left\lfloor\frac{m+n+1}{2}\right\rfloor-i
$$

考虑条件：

$$
A(i):\quad nums1[i-1]\le nums2[j]
$$

条件 A 具有以下两个性质：

1. **单调性**：随着 $i$ 增大，$j$ 减小，`nums1[i-1]` 单调不减，而 `nums2[j]` 单调不增。因此，A 一旦不成立，之后必然不成立。
2. **边界保证**：当 $i=0$ 时，令 `nums1[-1] = −∞`，因此 $A(0)$ 必然成立，保证至少存在一个满足 A 的位置。

由此可知，**条件 A 的取值呈现先成立、后不成立的单调结构**（也可能始终成立），可以通过二分查找确定满足 A 的最大位置。

```text
划分位置 i:    0       1       2       3       4       5
A(i):       True    True    True    True    False   False
             ↑                       ↑
          必然成立                最大成立位置 i*
```

*图 4：条件 A 的单调性与目标边界（示意）。*

算法在较短数组上二分查找：

- 若 A 成立，记录候选结果，继续向右搜索。
- 若 A 不成立，向左搜索。

最终找到满足 A 的最大位置 $i^*$。该位置也必然满足条件 B，其证明留在第四节。

## 三、算法设计与实现

### 3.2 Python 实现

```python
from typing import List

class Solution:
    def findMedianSortedArrays(
        self, nums1: List[int], nums2: List[int]
    ) -> float:
        # 始终在较短数组上进行二分查找
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1

        m, n = len(nums1), len(nums2)
        left, right = 0, m
        half = (m + n + 1) // 2

        neg_inf, pos_inf = float('-inf'), float('inf')
        median1, median2 = 0, 0

        while left <= right:
            # 确定两个数组的划分位置
            i = (left + right) // 2
            j = half - i

            # 获取划分边界的四个元素
            a_left = neg_inf if i == 0 else nums1[i - 1]
            a_right = pos_inf if i == m else nums1[i]
            b_left = neg_inf if j == 0 else nums2[j - 1]
            b_right = pos_inf if j == n else nums2[j]

            # 寻找满足 A(i) 的最大划分位置
            if a_left <= b_right:
                median1 = max(a_left, b_left)
                median2 = min(a_right, b_right)
                left = i + 1
            else:
                right = i - 1

        # 根据总长度的奇偶性返回中位数
        if (m + n) % 2 == 1:
            return median1
        return (median1 + median2) / 2
```

代码使用 `float('inf')` 处理数组边界，避免使用固定整数作为无穷大哨兵。

**复杂度分析：** 设 $m\le n$，二分搜索范围包含 $m+1$ 个划分位置，因此时间复杂度为 $O(\log(m+1))$，空间复杂度为 $O(1)$。

## 四、正确性证明

### 4.1 二分查找的正确性

证明算法能够找到满足条件 $A(i)$ 的最大划分位置：

$$
i^*=\max\{i\in[0,m]\mid A(i)\}
$$

由第二节可知，$A(i)$ 具有单调性：随着 $i$ 增大，条件 A 由成立变为不成立，且 $A(0)$ 必然成立，因此 $i^*$ 一定存在。

**1. 循环不变量**

在每次 `while` 循环开始时，始终满足：

$$
\boxed{\begin{aligned}
&\forall i\in[0,left),\quad A(i)=\mathrm{True}\\
&\forall i\in(right,m],\quad A(i)=\mathrm{False}
\end{aligned}}
$$

即：`left` 左侧的位置均满足 A，`right` 右侧的位置均不满足 A，尚未确定的范围为 `[left, right]`。

**2. 初始化**

初始时 `left=0, right=m`。两个已确定区间均为空，因此循环不变量成立。

**3. 保持**

每次迭代取：

$$
mid=\left\lfloor\frac{left+right}{2}\right\rfloor
$$

根据条件 A 的单调性，分两种情况：

- **若 $A(mid)$ 成立**：所有 $i\le mid$ 均满足 A，因此令 `left=mid+1`，循环不变量保持成立。
- **若 $A(mid)$ 不成立**：所有 $i\ge mid$ 均不满足 A，因此令 `right=mid-1`，循环不变量保持成立。

两种情况下，搜索区间均严格缩小。

**4. 终止与候选结果的正确性**

由于搜索区间不断缩小，循环最终在 `left > right` 时终止，此时 $left=right+1$。结合循环不变量可知：

$$
\boxed{i^*=right=left-1}
$$

还需证明代码一定访问 $i^*$，并正确保留该位置的候选结果。

**为什么一定访问 $i^*$？** 初始搜索区间包含 $i^*$。若 $mid<i^*$，则 A 成立，更新后仍有 $left\le i^*$；若 $mid>i^*$，则 A 不成立，更新后仍有 $right\ge i^*$。因此，在访问 $i^*$ 之前，它始终位于搜索区间内。由于区间严格缩小，算法必然访问 $i^*$。

**为什么记录不会被覆盖？** 当 $mid=i^*$ 时，A 成立，算法记录该位置的 `median1` 和 `median2`，并令 $left=i^*+1$。此后访问的位置均大于 $i^*$，根据其最大性，条件 A 均不成立，因而不会再次更新候选结果。

综上，**算法一定找到满足 A 的最大位置 $i^*$，且最终保存的候选结果恰好对应 $i^*$**。

### 4.2 最大划分位置满足正确划分条件

由 4.1 节可知，算法能够找到满足条件 A 的最大位置 $i^*$：

$$
A(i^*):\quad nums1[i^*-1]\le nums2[j^*]
$$

其中：

$$
j^*=\left\lfloor\frac{m+n+1}{2}\right\rfloor-i^*
$$

下面证明该位置也必然满足：

$$
B(i^*):\quad nums2[j^*-1]\le nums1[i^*]
$$

**采用反证法。** 假设 $B(i^*)$ 不成立，即：

$$
nums2[j^*-1]>nums1[i^*]
$$

分两种情况讨论。

**情况一：$i^*=m$**

此时 `nums1` 的右半部分为空，根据边界约定：

$$
nums1[i^*]=+\infty
$$

因此 $B(i^*)$ 必然成立，与假设矛盾。

**情况二：$i^*<m$**

由于 $B(i^*)$ 不成立，且 `nums1[i*]` 是实际数组元素，因此必有 $j^*>0$。

考虑将划分位置向右移动一位：

$$
i'=i^*+1,\qquad j'=j^*-1
$$

由反证假设：

$$
nums1[i^*]<nums2[j^*-1]=nums2[j']
$$

因此：

$$
nums1[i'-1]=nums1[i^*]\le nums2[j']
$$

即：

$$
\boxed{A(i^*+1)\text{ 成立}}
$$

这与 $i^*$ 是满足 A 的最大位置矛盾。

综上：

$$
\boxed{A(i^*)\land B(i^*)}
$$

又因为划分位置始终满足：

$$
i^*+j^*=\left\lfloor\frac{m+n+1}{2}\right\rfloor
$$

所以，**算法找到的 $i^*$ 必然构成正确划分**：左半部分元素数量符合要求，且左半部分所有元素均不大于右半部分。

因此，代码记录的：

$$
L=\max(nums1[i^*-1],nums2[j^*-1])
$$

$$
R=\min(nums1[i^*],nums2[j^*])
$$

分别为左半部分最大值和右半部分最小值。总长度为奇数时，中位数为 $L$；总长度为偶数时，中位数为 $(L+R)/2$。因此算法返回的中位数正确。
