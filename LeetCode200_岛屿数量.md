# LeetCode 200. 岛屿数量

## 一、问题描述

题目链接：[LeetCode 200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)

给定一个由 `'1'`（陆地）和 `'0'`（水）组成的二维网格 `grid`，计算其中岛屿的数量。

一个岛屿由水平或垂直方向相邻的陆地组成，四周被水或网格边界包围。

**示例：**

```text
输入：
1 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1

输出：3
```

## 二、算法思路

**基本思路：将每个岛屿看作一个连通分量，使用深度优先搜索（DFS）遍历并标记同一岛屿中的所有陆地。每启动一次 DFS，就发现一个新的岛屿。**

具体步骤：

1. 遍历网格，寻找未访问过的陆地。
2. 每发现一块未访问的陆地，岛屿数量加一，并以该位置为起点启动 DFS。
3. 使用栈遍历上下左右相邻的陆地，并标记已访问的位置。
4. 当前 DFS 结束后，继续寻找下一个未访问的陆地。

**示例：**

```text
原始网格：          第一次 DFS：
1 1 0 0            X X 0 0
1 1 0 1            X X 0 1
0 0 0 1            0 0 0 1

岛屿数 = 0         岛屿数 = 1

第二次 DFS：
X X 0 0
X X 0 X
0 0 0 X

岛屿数 = 2
```

其中，`X` 表示已访问的陆地。

**核心思想：通过 DFS 将同一个连通分量中的所有节点一次性标记，使每个岛屿只被统计一次。**

## 三、算法实现

### 3.1 优化前：独立访问数组 + 出栈标记

使用额外的 `record` 数组记录已访问的陆地，在节点出栈时才进行标记。

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        if not grid or not grid[0]:
            return 0

        n, m = len(grid), len(grid[0])
        record = [[0] * m for _ in range(n)]
        island_n = 0

        for row in range(n):
            for col in range(m):
                if record[row][col] == 0 and grid[row][col] == '1':
                    stack = [(row, col)]
                    island_n += 1

                    while stack:
                        x, y = stack.pop()
                        record[x][y] = 1

                        if (y + 1 < m and record[x][y+1] == 0
                                and grid[x][y+1] == '1'):
                            stack.append((x, y+1))

                        if (x + 1 < n and record[x+1][y] == 0
                                and grid[x+1][y] == '1'):
                            stack.append((x+1, y))

                        if (x - 1 >= 0 and record[x-1][y] == 0
                                and grid[x-1][y] == '1'):
                            stack.append((x-1, y))

                        if (y - 1 >= 0 and record[x][y-1] == 0
                                and grid[x][y-1] == '1'):
                            stack.append((x, y-1))

        return island_n
```

因为出栈时才标记，同一块陆地可能被不同邻居重复加入栈；此外，访问状态需要额外 `O(nm)` 空间。

### 3.2 优化后：原地标记 + 入栈标记

**优化要点：**

1. **入栈时标记**：发现新陆地后立即标记，保证每块陆地最多入栈一次，避免重复操作。
2. **原地标记**：直接把已访问的 `'1'` 改为 `'0'`，无需 `record` 数组。
3. **方向数组**：统一处理上下左右四个邻居，减少重复代码。

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        if not grid or not grid[0]:
            return 0

        n, m = len(grid), len(grid[0])
        count = 0
        directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]

        for i in range(n):
            for j in range(m):
                if grid[i][j] != '1':
                    continue

                count += 1
                stack = [(i, j)]
                grid[i][j] = '0'

                while stack:
                    x, y = stack.pop()

                    for dx, dy in directions:
                        nx, ny = x + dx, y + dy

                        if (0 <= nx < n and 0 <= ny < m
                                and grid[nx][ny] == '1'):
                            grid[nx][ny] = '0'
                            stack.append((nx, ny))

        return count
```

**复杂度分析：**

设网格大小为 `n × m`。

| 指标 | 优化前 | 优化后 |
| --- | --- | --- |
| 时间复杂度 | $ `O(nm)` $ | $ `O(nm)` $ |
| 访问标记空间 | $ `O(nm)` $ | $ `O(1)` $ |
| DFS 栈空间（最坏） | $ `O(nm)` $ | $ `O(nm)` $ |
| 是否修改输入 | 否 | 是 |

两者的时间复杂度均为 $ `O(nm)` $：原始实现可能重复入栈，但每块陆地最多被四个邻居加入栈，整体仍为线性级别。优化实现避免重复入栈，并省去了访问数组。

**注意：** 优化实现会修改原始 `grid`。如果不能修改输入，可保留 `record` 数组，但仍应在入栈时标记。

## 四、正确性证明

以优化后的算法为例，采用循环不变量法证明。

### 4.1 循环不变量

在每次 DFS 的栈处理过程中，始终满足：

1. 所有入栈节点都属于当前起点所在的连通分量。
2. 每个入栈节点均已标记，因此不会重复入栈。
3. 栈中保存尚未完成邻居检查的节点。

### 4.2 初始化

DFS 从一块未访问的陆地开始。将该节点标记并入栈，此时它属于自身所在的连通分量，且尚未检查邻居，因此循环不变量成立。

### 4.3 保持

每轮从栈中取出一个节点，检查其上下左右相邻的陆地。若发现未访问陆地，则立即标记并入栈。

新节点与当前节点相邻，必然属于同一连通分量；由于入栈前已标记，也不会重复入栈。因此循环不变量保持成立。

### 4.4 终止

每块陆地最多入栈一次，DFS 必然终止。

栈为空时，当前连通分量中的陆地均已访问。否则，存在一条从起点到未访问陆地的路径；这条路径上第一个未访问节点的前驱已被访问，检查前驱的邻居时就应发现该节点，产生矛盾。

因此每次 DFS 恰好处理一个完整岛屿。外层遍历只从未访问的陆地启动 DFS，既不会重复统计，也不会遗漏岛屿。所以最终的 DFS 启动次数恰好等于岛屿数量，算法正确。

原始实现虽然可能重复入栈，但仍只沿相邻陆地扩展，并最终遍历完整连通分量，因此同样正确。
