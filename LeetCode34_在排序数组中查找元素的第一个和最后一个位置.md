# LeetCode 34. 在排序数组中查找元素的第一个和最后一个位置

## 一、问题描述

题目链接：[LeetCode 34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

给定一个按非递减顺序排列的整数数组 `nums` 和整数 `target`，返回目标值第一次和最后一次出现的下标。如果目标值不存在，返回 `[-1, -1]`。

要求时间复杂度为 $ `O(\log n)` $。

**示例：**

```text
nums = [5, 7, 7, 8, 8, 10]
target = 8
输出：[3, 4]

target = 6
输出：[-1, -1]
```

## 二、算法思路

### 2.1 基本思路：两次二分查找

将问题规约为一个统一的子问题：

**`lower_bound(t)`：查找数组中第一个大于等于 `t` 的元素位置；如果不存在，返回 `n`。**

由于本题数组元素均为整数，只需调用两次：

1. `left = lower_bound(target)`：第一个大于等于 `target` 的位置。
2. `right = lower_bound(target + 1)`：第一个严格大于 `target` 的位置。

如果目标值存在，其全部出现位置恰好构成区间 `[left, right - 1]`。

### 2.2 统一二分查找：显式记录候选位置

定义 `searchRangeI(t)`，返回第一个大于等于 `t` 的位置。

使用三个变量，明确区分**搜索边界**与**候选结果**：

| 变量 | 含义 |
|---|---|
| `left` | 尚待搜索区间的左边界 |
| `right` | 尚待搜索区间的右边界 |
| `pos` | 已经找到的最靠左合法候选位置；尚未找到时为 `n` |

初始化：

```python
left, right = 0, n - 1
pos = n
```

每轮检查中点 `mid`：

- **`nums[mid] < t`**：根据有序性，`mid` 及其左侧都不可能是答案，更新 `left = mid + 1`。
- **`nums[mid] >= t`**：`mid` 是一个合法候选位置，记录 `pos = mid`；继续搜索更靠左的位置，更新 `right = mid - 1`。

循环结束后返回 `pos`。若不存在满足条件的位置，`pos` 保持为 `n`。

**设计要点：`left/right` 只表示当前搜索范围，`pos` 专门保存已发现的候选答案。**

### 2.3 结果构造与边界处理

例如：

```text
nums = [5, 7, 7, 8, 8, 10]
target = 8

searchRangeI(8) = 3
searchRangeI(9) = 5

结果 = [3, 5 - 1] = [3, 4]
```

还需判断目标值是否存在：

```python
if left == n or nums[left] != target:
    return [-1, -1]
```

其中 `left == n` 必须先检查，以避免数组越界。

**边界情况：**

| 输入 | 查找结果 |
|---|---|
| `nums=[]`，`target=1` | `[-1,-1]` |
| `nums=[1,3,5]`，`target=2` | `[-1,-1]` |
| `nums=[2,2,2]`，`target=2` | `[0,2]` |
| `nums=[1,3,5]`，`target=5` | `[2,2]` |

## 三、算法实现

### 3.1 Python 实现

保留 `pos` 显式记录候选位置。

```python
class Solution:
    def searchRange(self, nums: list[int], target: int) -> list[int]:
        n = len(nums)

        def searchRangeI(t):
            left = 0
            right = n - 1
            pos = n

            while left <= right:
                mid = (left + right) // 2

                if nums[mid] < t:
                    left = mid + 1
                else:
                    pos = mid
                    right = mid - 1

            return pos

        left = searchRangeI(target)
        right = searchRangeI(target + 1)

        if left == n or nums[left] != target:
            return [-1, -1]

        return [left, right - 1]
```

### 3.2 复杂度分析

- **时间复杂度：** $ `O(\log n)` $。两次二分查找，每次都将搜索范围缩小约一半。
- **空间复杂度：** $ `O(1)` $。只使用常数个变量。

## 四、正确性证明

### 4.1 循环不变量

首先证明 `searchRangeI(t)` 返回第一个大于等于 `t` 的位置，若不存在则返回 `n`。

每轮循环开始时，保持以下不变量：

1. **左侧已排除部分：** 对所有 `i < left`，都有 `nums[i] < t`。
2. **右侧已确认部分：** 对所有 `i > right` 且 `i < n`，都有 `nums[i] >= t`。
3. **候选结果：** `pos = right + 1`，即右侧已确认部分的第一个位置；若该部分为空，则 `pos = n`。

因此，未知的候选区域仅为 `[left, right]`。

### 4.2 初始化与保持

**（1）初始化**

初始时 `left = 0`、`right = n - 1`、`pos = n`。

左右两侧已排除、已确认部分均为空，且 `pos = right + 1 = n`，不变量成立。

**（2）保持**

假设本轮开始时不变量成立。

- **情况一：`nums[mid] < t`。** 因数组有序，所有 `i <= mid` 均不满足条件，更新 `left = mid + 1` 不会遗漏答案。右边界和 `pos` 不变，三个不变量仍然成立。
- **情况二：`nums[mid] >= t`。** 因数组有序，所有 `i >= mid` 都满足条件，更新 `pos = mid`、`right = mid - 1`，将这一段纳入右侧已确认区域，并继续向左搜索。此时依然有 `pos = right + 1`，三个不变量成立。

因此，循环不变量在每次迭代后都成立。

### 4.3 终止与最终结果

每轮循环都会缩小搜索区间，且循环执行时 `left <= mid <= right`：

- `left = mid + 1` 时，左边界严格增大。
- `right = mid - 1` 时，右边界严格减小。

因此循环必然终止，终止时 `left = right + 1`。

结合循环不变量中的 `pos = right + 1`，得到 `left = pos`。

此时：

- `pos` 左侧所有元素均小于 `t`。
- 从 `pos` 开始的所有有效下标对应元素均大于等于 `t`。

所以 `pos` 必然是第一个大于等于 `t` 的位置；如果不存在，则 `pos = n`。

由此证明 `searchRangeI(t)` 正确。

最后，对整数数组：

- `searchRangeI(target)` 给出目标值可能出现的最左位置。
- `searchRangeI(target + 1)` 给出第一个严格大于目标值的位置。

若最左位置越界或对应元素不等于 `target`，目标不存在，返回 `[-1, -1]`；否则，所有等于目标值的元素恰好位于二者之间，返回 `[left, right - 1]`。

因此，整个算法正确。
