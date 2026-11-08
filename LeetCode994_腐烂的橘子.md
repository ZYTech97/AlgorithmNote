# LeetCode 994. 腐烂的橘子

## 一、问题描述

题目链接：[LeetCode 994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)

给定二维网格 `grid`：`0` 表示空单元格，`1` 表示新鲜橘子，`2` 表示腐烂橘子。每分钟，腐烂橘子会使上下左右相邻的新鲜橘子腐烂。

求使所有橘子腐烂所需的最少分钟数；如果无法全部腐烂，返回 `-1`。

**示例：**

```text
输入：
2 1 1
1 1 0
0 1 1

输出：4
```

## 二、算法思路

**基本思路：多源广度优先搜索（BFS）。从所有初始腐烂橘子同时出发，按分钟逐层传播。**

1. 扫描网格，记录初始腐烂橘子和新鲜橘子的数量。
2. 每轮遍历当前层的腐烂橘子，将相邻新鲜橘子标记为腐烂，并加入下一层。
3. 每传播一层，时间加一；新鲜橘子全部腐烂时结束，否则无法继续传播时返回 `-1`。

**核心思想：多源 BFS 的每一层对应一分钟，按层扩展即可得到最短传播时间。**

## 三、算法实现

采用**分层 BFS**，新鲜橘子加入下一层时立即标记为腐烂，避免重复入队；全部腐烂后提前终止。

```python
class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        m, n = len(grid), len(grid[0])
        rotten = []
        fresh = 0

        for i in range(m):
            for j in range(n):
                if grid[i][j] == 2:
                    rotten.append((i, j))
                elif grid[i][j] == 1:
                    fresh += 1

        minutes = 0
        directions = [(-1, 0), (1, 0), (0, -1), (0, 1)]

        while rotten and fresh > 0:
            next_rotten = []

            for x, y in rotten:
                for dx, dy in directions:
                    nx, ny = x + dx, y + dy

                    if (0 <= nx < m and 0 <= ny < n
                            and grid[nx][ny] == 1):
                        grid[nx][ny] = 2
                        fresh -= 1
                        next_rotten.append((nx, ny))

            rotten = next_rotten
            minutes += 1

        return minutes if fresh == 0 else -1
```

**复杂度分析：**

设网格大小为 $ `m\times n` $。

- **时间复杂度：** $ `O(mn)` $。每个橘子最多入队一次，每次检查四个方向。
- **空间复杂度：** $ `O(mn)` $。最坏情况下，需要保存线性数量的腐烂橘子。

**注意：** 算法会原地修改 `grid`，将新腐烂的橘子从 `1` 标记为 `2`。

## 四、正确性证明

使用循环不变量证明。

### 4.1 循环不变量

每轮 BFS 开始时：

1. `rotten` 保存上一轮新腐烂的橘子（初始时为所有腐烂橘子）。
2. `fresh` 是当前剩余的新鲜橘子数量。
3. `minutes` 是已完成的传播轮数。

### 4.2 初始化

初始扫描得到所有腐烂橘子与新鲜橘子数量，`minutes = 0`，故不变量成立。

### 4.3 保持

每轮只让当前层感染相邻的新鲜橘子，新腐烂的橘子进入下一层，并同步更新 `fresh`。完成一层后 `minutes` 增加一，因此不变量保持成立。

### 4.4 终止

若 `fresh == 0`，说明全部橘子已经腐烂。BFS 按最短传播距离逐层扩展，因而返回的是最少分钟数。

若 `rotten` 为空而 `fresh > 0`，说明没有可继续传播的腐烂橘子，剩余新鲜橘子无法腐烂，正确返回 `-1`。因此算法正确。
