# LeetCode 438. 找到字符串中所有字母异位词

## 一、问题描述

[题目链接](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

给定字符串 `s` 和 `p`，返回 `s` 中所有与 `p` 互为字母异位词的子串起始索引。字母异位词的各字符出现次数相同，顺序可以不同。

```text
输入：s = "cbaebabacd", p = "abc"
输出：[0, 6]
解释："cba" 和 "bac" 均为 "abc" 的字母异位词。
```

字符串仅包含小写英文字母，长度均在 1 到 30000 之间。

## 二、基本思路

### 2.1 解空间分解：按起点枚举

将问题分解为判断每个起点 `i` 对应的长度为 `m=len(p)` 的子串是否满足条件。朴素解法对每个子串重新统计字符频次并比较，时间复杂度为 $ `O(nm)` $。

### 2.2 利用滑动窗口优化

相邻两个长度为 `m` 的窗口有 `m-1` 个字符重叠。窗口右移一位，只需移除左侧字符、加入右侧字符，不必重新统计整个窗口。

```text
s:        c b a e b a ...
窗口 i:  [c b a]
窗口 i+1:  [b a e]
          移除 c，加入 e
```

使用 `target` 记录 `p` 的字符频次，`window` 记录当前窗口的字符频次。初始化首个窗口后，逐个比较两张频次表，移动窗口并重复检查。窗口长度固定；若 `len(p)>len(s)`，直接返回空列表。

设字符集大小为 $ `\Sigma=26` $，总时间复杂度为 $ `O(m+n\Sigma)` $，辅助空间为 $ `O(\Sigma)` $。

### 2.3 利用差异计数优化

用 `diff` 记录**频次不相等的字符种类数**：

```math
diff=|\{c\mid window[c]\ne target[c]\}|
```

当 `diff == 0` 时，窗口匹配。窗口移动只涉及移出和移入的两个字符，因而只需更新它们的贡献。令 `I(c)` 表示字符 `c` 的频次是否不匹配（不匹配为 1，否则为 0），则：

```math
diff_{\mathrm{new}}=diff_{\mathrm{old}}+I_{\mathrm{new}}(c)-I_{\mathrm{old}}(c)
```

对应操作：

```python
old = window[c] != target[c]
window[c] += delta
diff += (window[c] != target[c]) - old
```

其中 `delta=-1` 表示移除，`delta=1` 表示加入。**新贡献减去旧贡献**使每次窗口移动只需常数时间。总时间复杂度为 $ `O(n+m+\Sigma)` $，辅助空间为 $ `O(\Sigma)` $；固定字符集下分别为 $ `O(n+m)` $ 和 $ `O(1)` $。

## 三、算法实现

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> list[int]:
        n, m = len(s), len(p)
        if m > n:
            return []

        target = [0] * 26
        window = [0] * 26

        for c in p:
            target[ord(c) - ord('a')] += 1
        for c in s[:m]:
            window[ord(c) - ord('a')] += 1

        diff = sum(window[i] != target[i] for i in range(26))

        def update(c: str, delta: int) -> None:
            nonlocal diff
            i = ord(c) - ord('a')
            old = window[i] != target[i]
            window[i] += delta
            diff += (window[i] != target[i]) - old

        ans = []
        if diff == 0:
            ans.append(0)

        for i in range(1, n - m + 1):
            update(s[i - 1], -1)
            update(s[i + m - 1], 1)
            if diff == 0:
                ans.append(i)

        return ans
```

**复杂度：** 时间 $ `O(n+m)` $，辅助空间 $ `O(1)` $（不计结果）。

## 四、正确性证明

### 4.1 循环不变量

每次循环开始时，准备处理起点 `i`：`target` 准确记录 `p` 的字符频次；`window` 准确记录上一窗口 `s[i-1:i+m-1]` 的频次；`diff` 是频次不匹配的字符种类数；`ans` 恰好包含所有起点小于 `i` 的合法索引。

### 4.2 初始化

初始化两个频次表及 `diff`，检查第一个窗口并按需将索引 0 加入 `ans`。因此首次进入循环前不变量成立。

### 4.3 保持

处理起点 `i` 时，移除 `s[i-1]`、加入 `s[i+m-1]`，因此 `window` 恰好表示新窗口。每次 `update` 用**新贡献减去旧贡献**修正 `diff`，其余字符的贡献不变，所以 `diff` 仍准确。由于 `diff == 0` 当且仅当两张频次表相同，算法恰好在当前窗口合法时将 `i` 加入 `ans`，循环不变量保持成立。

### 4.4 终止

所有长度为 `m` 的窗口均被检查后，`ans` 恰好包含全部合法起点。若 `m>n`，不存在候选窗口，直接返回空列表也正确。
