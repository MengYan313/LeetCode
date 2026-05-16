# 哈希

## [两数之和](https://leetcode.cn/problems/two-sum/)(1)


问题：给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。你可以按任意顺序返回答案。

分析：一次遍历 + 哈希表：遍历到 x 时，查找补数 y = target - x 是否已出现；若出现直接返回两者下标，否则记录 x 的下标。

注意：若是有序数组，可以使用相向双指针解决。但排序复杂度为 O(nlogn)，因此只能在三数之和中使用。

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        pos = {}  # value -> index
        for i, x in enumerate(nums):
            y = target - x
            if y in pos:
                return [pos[y], i]
            pos[x] = i
```

## [和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)(560)

问题：给你一个整数数组 `nums` 和一个整数 `k` ，统计并返回 *该数组中和为 `k` 的子数组的个数* 。子数组是数组中元素的连续非空序列。

分析：**前缀和+哈希表**：子数组 sum(i..j) = pre[j] - pre[i-1]。遍历到当前位置前缀和为 s，想要子数组和为 k，就需要之前出现过前缀和 s-k；其出现次数就是**以当前位置为结尾的合法子数组个数**。

复杂度：`O(n)` `O(n)`

```python
from collections import defaultdict

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        cnt = defaultdict(int)
        cnt[0] = 1  # 前缀和为 0 的“空前缀”出现 1 次
        s = ans = 0
        for num in nums:
            s += num
            ans += cnt[s - k]  # 统计有多少个前缀和等于 s-k，即以当前位置为结尾的合法子数组个数
            cnt[s] += 1
        return ans
```

## [字母异位词分组](https://leetcode.cn/problems/group-anagrams/)(49)

问题：给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

分析：用哈希表把“同一组异位词”归到同一个 key 下。key 选用 **26 个字母频次向量**（转成 tuple 可哈希），同频次即为同一组异位词。

注意：频次数组必须转成 tuple 才能作为字典 key。比起 ''.join(sorted(s))，频次 key 避免了排序

复杂度：`O(nk)` `O(nk)`

```python
from collections import defaultdict

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        groups = defaultdict(list)
        for s in strs:
            cnt = [0] * 26
            for ch in s:
                cnt[ord(ch) - ord('a')] += 1  # ord('a') == 97
            groups[tuple(cnt)].append(s)
        return list(groups.values())
```

## [缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)(41)

问题：给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

分析：最小缺失正数一定在 [1, n+1]。用“原地哈希/桶排序”的思想：把值 x（若 1<=x<=n）放到它应该在的位置 x-1。排完后再扫一遍，找到第一个位置不对的下标 i，答案就是 i+1。

注意：while 交换时**不立刻 i++**：交换后当前位置来了新数，可能还需要继续放到正确位置。判重条件 nums[x-1] != x ，否则遇到重复值会死循环。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def firstMissingPositive(self, nums: List[int]) -> int:
        n = len(nums)
        i = 0
        while i < n:  # 把每个在 [1, n] 范围内的数 x 放到下标 x-1 的位置上（原地哈希）
            x = nums[i]
            if 1 <= x <= n and nums[x - 1] != x:  # x-1 位置上已有 x 则不交换，否则死循环
                nums[i], nums[x - 1] = nums[x - 1], nums[i]
            else:  # 交换后当前位置来了新数，可能还需要继续放到正确位置，因此不能立刻 i++
                i += 1
        
        for i, x in enumerate(nums):  # 第一个 nums[i] != i+1 的位置，其 i+1 就是缺失的最小正数
            if x != i + 1:
                return i + 1
        return n + 1
```

# 双指针

## [验证回文串](https://leetcode.cn/problems/valid-palindrome/)(125)

问题：如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 **回文串** 。字母和数字都属于字母数字字符。给你一个字符串 `s`，如果它是 **回文串** ，返回 `true` ；否则，返回 `false` 。

分析：用双指针从两端向中间靠拢。遇到非字母数字字符就跳过，遇到有效字符后，转成小写再比较；只要有一对不同，就不是回文串。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        n = len(s)
        left, right = 0, n - 1
        while left < right:
            while left < right and not s[left].isalnum():  # 遇到非字母数字字符就跳过
                left += 1
            while left < right and not s[right].isalnum():  # 遇到非字母数字字符就跳过
                right -= 1
            if s[left].lower() != s[right].lower():  # 遇到有效字符后，转成小写再比较
                return False
            left += 1
            right -= 1
        return True
```

## [两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)(167)

问题：给你一个下标从 **1** 开始的整数数组 `numbers` ，该数组已按 **非递减顺序排列** ，请你从数组中找出满足相加之和等于目标数 `target` 的两个数。如果设这两个数分别是 `numbers[index1]` 和 `numbers[index2]` ，则 `1 <= index1 < index2 <= numbers.length` 。以长度为 2 的整数数组 `[index1, index2]` 的形式返回这两个整数的下标 `index1` 和 `index2`。你可以假设每个输入 **只对应唯一的答案** ，而且你 **不可以** 重复使用相同的元素。你所设计的解决方案必须只使用常量级的额外空间。

分析：数组有序，使用相向双指针：如果两数之和小于 target，说明需要更大，左指针右移；大于 target，说明需要更小，右指针左移

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        n = len(numbers)
        left, right = 0, n - 1
        while left < right:
            s = numbers[left] + numbers[right]
            if s == target:
                return [left + 1, right + 1]
            if s < target:  # 如果两数之和小于 target，说明需要更大，左指针右移
                left += 1
            else:           # 如果两数之和大于 target，说明需要更小，右指针左移
                right -= 1
```

## [三数之和](https://leetcode.cn/problems/3sum/)(15)


问题：给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。**注意：**答案中不可以包含重复的三元组。

分析：枚举 `i`，将问题转化为两数之和为 `nums[i]`，为了使用相向双指针，需要先对 `nums` 排序

* **顺序不重要，那么就规定一个顺序 `i < j < k`，保证不重复**（否则有 `6` 种顺序）
* 不包含重复三元组：如果当前 `nums[i]` 和上一个数 `nums[i - 1]` 相同，直接跳过 `continue`（每找到一个三元组，需要对 `j` 和 `k` 做同样的检查与跳过操作）
* 优化1：若 `nums[i] + nums[i + 1] + nums[i + 2] > 0`，表明当前 `nums[i]` 和后续最小的两个值加起来都大于目标值 `0`，那么后续 `i, j, k` 的枚举只会越来越大，故已不存在满足要求的三元组，直接退出循环返回答案
* 优化2：若 `nums[i] + nums[n - 2] + nums[n - 1] < 0`，表明当前  `nums[i]` 和后续最大的两个值加起来都小于目标值 `0`，那么无论 `j, k` 怎么取值都会小于 `0`，故当前 `i` 值太小，可直接 `continue` 枚举下一个 `i`

复杂度： `O(n`^2^`)` `O(1)`

```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        nums.sort()  # 顺序不重要，那么就规定一个顺序 i < j < k
        res = []
        
        for i in range(n - 2):  # 需要留两个位置给 j,k 故枚举到 n-2
            x = nums[i]
            if i > 0 and x == nums[i - 1]:  # 针对i跳过重复的三元组
                continue
            if x + nums[i + 1] + nums[i + 2] > 0:  # 优化一
                break
            if x + nums[n - 2] + nums[n - 1] < 0:  # 优化二
                continue

            j, k = i + 1, n - 1  # 定义双指针从两端向中间移动
            while j < k:
                s = x + nums[j] + nums[k]
                if s > 0:  # 右指针向左移动
                    k -= 1
                elif s < 0:  # 左指针向右移动
                    j += 1
                else:
                    res.append([x, nums[j], nums[k]])
                    j += 1
                    while j < k and nums[j] == nums[j - 1]:  # 先往后走一步，再针对j跳过重复的三元组
                        j += 1
                    k -= 1
                    while k > j and nums[k] == nums[k + 1]:  # 先往前走一步，再针对k跳过重复的三元组
                        k -= 1

        return res
```

## [盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)(11)

问题：给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。返回容器可以储存的最大水量。

分析：双指针分别指向两端，每次移动 `height` 较小的指针，变大时说明有可能增大面积，记出现过的最大面积为 `res`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        n = len(height)
        left, right = 0, n - 1
        ans = 0
        while left < right:  # 每次移动高度较小的指针，直至变大，因为这样才有可能增大面积
            if height[left] < height[right]:
                ans = max(ans, (right - left) * height[left])  # 更新当前最大面积
                left += 1
            else:
                ans = max(ans, (right - left) * height[right])
                right -= 1
        return ans
```

## [接雨水](https://leetcode.cn/problems/trapping-rain-water/)(42)


问题：给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![img](./assets/rainwatertrap.png)

分析：将题目**转化为一排宽度为 `1` 的空木桶：`装水体积 - 石头体积`**

解法1：前后缀分解，使用 `preMax[i]` 和 `sufMax[i]` 记录前后缀最大值，再计算 `Math.min(preMax[i], sufMax[i]) - height[i]`

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)
        preMax = [0] * n  # preMax[i] 表示从 height[0] 到 height[i] 的最大值
        preMax[0] = height[0]
        for i in range(1, n):
            preMax[i] = max(preMax[i - 1], height[i])

        sufMax = [0] * n  # sufMax[i] 表示从 height[i] 到 height[n-1] 的最大值
        sufMax[-1] = height[-1]
        for i in range(n - 2, -1, -1):
            sufMax[i] = max(sufMax[i + 1], height[i])

        ans = 0
        for i in range(n):
            ans += min(preMax[i], sufMax[i]) - height[i]  # 累加每个水桶能接多少水
        return ans
```

解法2：相向双指针，并使用 `preMax` 和 `sufMax` 记录前缀最大值和后缀最大值，利用以下性质优化：

* 若 `preMax < sufMax`，那么 `left` 指向的木桶的容量就是 `preMax`，计算完 `preMax - height[left]` 后 `left++`
* 若 `preMax >= sufMax`，那么 `right` 指向的木桶的容量就是 `sufMax`，计算完 `sufMax - height[right]` 后 `right--`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)
        left, right = 0, n - 1
        preMax = sufMax = 0
        ans = 0
        while left <= right:  # 双指针必在最高点相遇
            preMax = max(preMax, height[left])   # 前缀最大值，随着左指针 left 的移动而更新
            sufMax = max(sufMax, height[right])  # 后缀最大值，随着右指针 right 的移动而更新
            if preMax < sufMax:  # 空木桶容量由双指针的最小值确定，每次可确定出较小桶的容量
                ans += preMax - height[left]
                left += 1
            else:
                ans += sufMax - height[right]
                right -= 1
        return ans
```

## [搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)(240)


问题：编写一个高效的算法来搜索 *m* x *n* 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：每行的元素从左到右升序排列；每列的元素从上到下升序排列。

分析：“行递增 + 列递增”，希望找到一个位置，使得每次移动的增加和减小具有顺序性（类似相向双指针解决两数之和）。从**右上角**开始：若当前值 > target，这一列下面都更大，整列可排除 → 左移；若当前值 < target，这一行左边都更小，整行可排除 → 下移

复杂度：`O(m+n)` `O(1)`

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        m, n = len(matrix), len(matrix[0])
        i, j = 0, n - 1  # 从右上角开始
        while i < m and j >= 0:
            if matrix[i][j] == target:
                return True
            if matrix[i][j] > target:  # 当前值太大，往左（变小）
                j -= 1
            else:  # 当前值太小，往下（变大）
                i += 1
        return False
```

# 滑动窗口

## [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)(3)


问题：给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

解法2：滑动窗口，**从左向右枚举 `right`，不断移除 `left` 直至满足**，记出现过的最大长度为 `ans`

注意：由于字符 `s` 的种类是`ASCII` 字符，故可以用数组 `cnt[128]` 替代哈希表/集合，优化空间复杂度

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        st = set()
        left = ans = 0
        for right, ch in enumerate(s):
            while ch in st:  # 如果有重复字符，就不断收缩左边界
                st.remove(s[left])
                left += 1
            st.add(ch)
            ans = max(ans, right - left + 1)
        return ans

class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        cnt = [0] * 128  # ASCII 字符计数
        left = ans = 0
        for right, ch in enumerate(s):  # 枚举右端点
            c = ord(ch)
            cnt[c] += 1
            while cnt[c] > 1:  # 从不满足要求不断变为满足，即单调性，双指针的使用条件
                cnt[ord(s[left])] -= 1  # 不断移除左端点直至满足 cnt[c] == 1
                left += 1
            ans = max(ans, right - left + 1)
        return ans
```

## [长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)(209)

问题：给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**找出该数组中满足其总和大于等于 `target` 的长度最小的 **子数组**

`[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

分析：滑动窗口，从左向右枚举 `right`，不断移除 `left` 直至不满足，记出现过的最小长度为 `ans`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        n = len(nums)
        ans = n + 1
        left = 0
        s = 0
        for right in range(n):  # 枚举右端点
            s += nums[right]
            while s >= target:  # 从满足要求不断变为不满足，即单调性，双指针的使用条件
                ans = min(ans, right - left + 1)
                s -= nums[left]  # 不断移除左端点直至不满足
                left += 1
        return ans if ans < n + 1 else 0
```

## [乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/)(713)

问题：给你一个整数数组 `nums` 和一个整数 `k` ，请你返回子数组内所有元素的乘积严格小于 `k` 的连续子数组的数目。

分析：滑动窗口，从左向右枚举 `right`，不断移除 `left` 直至满足，累加计算当前右端点的连续子数组数量为 `ans`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def numSubarrayProductLessThanK(self, nums: List[int], k: int) -> int:
        if k <= 1:  # 如果 k <= 1，则任何正数的乘积都不可能小于 k
            return 0
        n = len(nums)
        ans = 0
        prod = 1
        left = 0
        for right in range(n):  # 枚举右端点
            prod *= nums[right]  # 扩张窗口，累乘右端点的值
            while prod >= k:  # 当乘积 >= k 时，不断收缩左端点，直到乘积 < k
                prod //= nums[left]
                left += 1
            ans += right - left + 1  # 当前右端点的合法连续子数组数量 = right - left + 1
        return ans
```

## [最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)(76)


问题：给定两个字符串 `s` 和 `t`，长度分别是 `m` 和 `n`，返回 s 中的 **最短窗口 子串**，使得该子串包含 `t` 中的每一个字符（**包括重复字符**）。如果没有这样的子串，返回空字符串 `""`。测试用例保证答案唯一。

分析：**用 diff 记录还需要的每个字符数量（含重复），用 need 表示总共还缺多少字符**。右指针扩展窗口：遇到“仍需要”的字符就 need--，并对 diff[ch]--（多出来会变负）。当 need==0 表示覆盖了 t，左指针尽量右移，丢掉多余字符

复杂度：`O(n)`

```python
from collections import Counter

class Solution:
    def minWindow(self, s: str, t: str) -> str:
        n, m = len(s), len(t)
        if n < m:  # 处理特殊情况，一定不存在
            return ""

        diff = Counter(t)  # diff[c] = t[c] - window_count[c]
        need = m  # 当前窗口还缺多少个字符
        left = start = 0
        min_len = n + 1
        
        for right, ch in enumerate(s):
            if diff[ch] > 0:  # 加入前，这个字符正是窗口所需要的
                need -= 1
            diff[ch] -= 1  # 允许为负，表示窗口内该字符“多出来了”

            if need == 0:  # 当前窗口已经覆盖 t，尝试收缩左边界
                while left <= right and diff[s[left]] < 0:
                    diff[s[left]] += 1
                    left += 1

                if right - left + 1 < min_len:  # 更新最短窗口，记录其左端点及长度
                    min_len = right - left + 1
                    start = left
                
                diff[s[left]] += 1  # 继续寻找下一个窗口：把左端关键字符移出
                need += 1
                left += 1

        return "" if min_len == n + 1 else s[start:start + min_len]
```

## [找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)(438)

问题：给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

分析：固定窗口长度为 len(p)，滑动窗口在 s 上移动。用**长度 26 的数组 diff 维护：窗口字符计数 - p 的字符计数**。当 diff 全为 0 时，窗口就是 p 的异位词。为了 O(1) 判断“是否全 0”，**用 bad 记录当前 diff 中非零项的个数，窗口每滑动一次只更新两个字符的影响**。

注意：当字符串 *s* 的长度小于字符串 *p* 的长度时，字符串 *s* 中一定不存在字符串 *p* 的异位词。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        n, m = len(s), len(p)
        if n < m:  # 处理特殊情况，一定不存在
            return []

        diff = [0] * 26  # diff[c] = window_count[c] - p_count[c]
        for ch in p:
            diff[ord(ch) - 97] -= 1  # ord('a') == 97
        for ch in s[:m]:
            diff[ord(ch) - 97] += 1
        bad = sum(1 for v in diff if v)  # 有多少个字符计数不为 0
        res = [0] if bad == 0 else []

        for i in range(m, n):
            add = ord(s[i]) - 97
            rem = ord(s[i - m]) - 97
            
            if diff[add] == 0:  # 加入前已经平衡
                bad += 1
            diff[add] += 1  # 加入 s[i]
            if diff[add] == 0:  # 加入后刚好平衡
                bad -= 1
            
            if diff[rem] == 0:  # 移除前已经平衡
                bad += 1
            diff[rem] -= 1  # 移除 s[i-m]
            if diff[rem] == 0:  # 移除后刚好平衡
                bad -= 1

            if bad == 0:  # 即 diff 全为 0，加入答案
                res.append(i - m + 1)

        return res
```

## [字符串的排列](https://leetcode.cn/problems/permutation-in-string/)(567)

问题：给你两个字符串 `s1` 和 `s2` ，写一个函数来判断 `s2` 是否包含 `s1` 的 排列。如果是，返回 `true` ；否则，返回 `false` 。换句话说，`s1` 的排列之一是 `s2` 的 **子串** 。

分析：固定窗口长度为 len(p)，滑动窗口在 s 上移动。用**长度 26 的数组 diff 维护：窗口字符计数 - p 的字符计数**。当 diff 全为 0 时，窗口就是 p 的异位词。为了 O(1) 判断“是否全 0”，**用 bad 记录当前 diff 中非零项的个数，窗口每滑动一次只更新两个字符的影响**。

注意：当字符串 *s* 的长度小于字符串 *p* 的长度时，字符串 *s* 中一定不存在字符串 *p* 的异位词。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        m, n = len(s1), len(s2)
        if n < m:  # 处理特殊情况，一定不存在
            return False
        
        diff = [0] * 26  # diff[c] = window[c] - s1[c]
        for ch in s1:
            diff[ord(ch) - 97] -= 1
        for ch in s2[:m]:
            diff[ord(ch) - 97] += 1
        bad = sum(1 for v in diff if v)  # 有多少个字符计数不为 0
        if bad == 0:  # res = [0] if bad == 0 else []
            return True
        
        for i in range(m, n):
            add = ord(s2[i]) - 97
            rem = ord(s2[i - m]) - 97
            
            if diff[add] == 0:  # 加入前已经平衡
                bad += 1
            diff[add] += 1  # 加入 s[i]
            if diff[add] == 0:  # 加入后刚好平衡
                bad -= 1
            
            if diff[rem] == 0:  # 移除前已经平衡
                bad += 1
            diff[rem] -= 1  # 移除 s[i-m]
            if diff[rem] == 0:  # 移除后刚好平衡
                bad -= 1
            
            if bad == 0:  # 即 diff 全为 0，加入答案
                return True  # res.append(i - m + 1)
        
        return False  # res != []
```

# 链表

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

## [反转链表](https://leetcode.cn/problems/reverse-linked-list/)(206)


问题：给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

解法 1：原地反转，`cur.next = pre`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        pre, cur = None, head
        while cur:
            nxt = cur.next  # 记录当前节点的下一个，防止丢失
            cur.next = pre  # 反转当前节点指针
            pre = cur       # 指针后移
            cur = nxt       # 指针后移
        return pre          # cur 为空时，pre 指向新头结点
```

解法2：递归

* 函数定义：反转链表并返回头节点
* 边界条件：单个节点直接返回本身
* 转移关系：递归到下一个节点，再拼接

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head or not head.next:  # 边界：空链表或单节点
            return head

        new_head = self.reverseList(head.next)  # 先反转后面的子链表
        head.next.next = head                   # 2 -> 1
        head.next = None                        # 1 -> None（断开原链接）
        return new_head
```

## [环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)(142)


问题：给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

分析：先用 Floyd 判圈找相遇点；再用头指针和相遇点指针找环入口

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = fast = head

        # 1) Floyd 判圈：先找相遇点
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                break
        else:
            return None  # 无环

        # 2) 找环入口：一指针从头出发，一指针从相遇点出发，相遇即入口
        p = head
        while p is not slow:
            p = p.next
            slow = slow.next
        return p
```

## [两数相加](https://leetcode.cn/problems/add-two-numbers/)(2)


问题：给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。请你将两个数相加，并以相同形式返回一个表示和的链表。

分析：同时顺序遍历并计算，用 `carry` 保存进位值，将每次计算结果**尾插**入新链表

注意：若长度不一致则高位补 0；结束后若 carry 仍为 1，需要额外补一个节点

复杂度：`O(Max(M,N))` `O(Max(M,N))`

```python
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode()  # 用虚拟头结点 dummy + tail 统一“尾插”构造结果链表
        tail = dummy
        carry = 0  # 进位值

        while l1 or l2:
            n1 = l1.val if l1 else 0  # 若长度不一致则高位补 0
            n2 = l2.val if l2 else 0
            s = n1 + n2 + carry
            tail.next = ListNode(s % 10)  # 尾部插入新节点
            tail = tail.next
            carry = s // 10  # 更新进位值
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None

        if carry:  # 最高位有进位则会多一位
            tail.next = ListNode(carry)

        return dummy.next
```

## [合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)(21)


问题：将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

分析：使用 `merge` 将 `list1` 和 `list2` 按升序连接，使用 `dummyHead` 处理 `head == null`

注意：若希望稳定（相等时保留 list1 在前），把条件改为 <= 让 list1 优先。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode()   # 虚拟头节点，统一处理头节点为空的情况
        cur = dummy
        
        while list1 and list2:  # 双指针遍历，按升序拼接
            if list1.val < list2.val:
                cur.next = list1
                list1 = list1.next
            else:
                cur.next = list2
                list2 = list2.next
            cur = cur.next
        
        cur.next = list1 or list2  # 将剩余的某段链表直接接上
        return dummy.next
```

## [合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)(23)


问题：给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

分析：用小顶堆维护当前 k 条链表的下一个候选节点：每次取最小节点接到答案链表，再把该节点的 next 再入堆。始终保持堆大小 ≤ k

注意：堆元素用 (val, i, node)：i 用来在 val 相同的时候打破比较（避免 Python 直接比较节点对象报错）。

```python
import heapq

class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        heap = []  # 用小顶堆维护当前 k 条链表的下一个候选节点
        for i, node in enumerate(lists):
            if node:  # 先将 k 个升序链表的头节点入堆
                heapq.heappush(heap, (node.val, i, node))  # (值, 来源链表编号, 节点)

        dummy = ListNode()
        cur = dummy
        while heap:  # 每次取最小节点接到答案链表，再把该节点的 next 再入堆。
            _, i, node = heapq.heappop(heap)
            cur.next = node
            cur = cur.next
            if node.next:
                heapq.heappush(heap, (node.next.val, i, node.next))
        
        return dummy.next
```

## [删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)(19)


问题：给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

解法 1：先统计链表长度，定位倒数第 `n + 1` 个节点进行删除

解法 2：使用[双指针](#[链表中倒数第k个节点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)(LCR140))定位倒数第 `n + 1` 个节点进行删除，使用 `dummyHead` 处理 `head == null`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        dummy = ListNode(0, head)  # 处理删除头节点的统一写法
        left = right = dummy
        
        for _ in range(n):  # right 先走 n 步，使得 right 与 left 间隔 n 个节点
            right = right.next
        
        while right.next:  # 同步前进直到 right 到达最后一个节点
            left = left.next
            right = right.next
        
        left.next = left.next.next  # left.next 就是倒数第 n 个节点，删除它
        return dummy.next
```

## [K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)(25)


问题：给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

分析：原地反转，`cur.next = pre`，使用 `dummyHead` 处理 `head == null`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def reverseKGroup(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        n, cur = 0, head
        while cur:  # 先统计节点数，决定能翻转多少组
            n += 1
            cur = cur.next

        dummy = ListNode(0, head)  # 哨兵节点，方便处理头结点变化
        p0 = dummy                 # p0 指向每一组翻转前的“前驱节点”
        pre, cur = None, head

        while n >= k:
            for _ in range(k):     # 原地反转 k 个节点：cur.next = pre
                nxt = cur.next     # 记录当前节点的下一个，防止丢失
                cur.next = pre     # 反转当前节点
                pre = cur          # 指针后移
                cur = nxt          # 指针后移

            # 此时 pre 是本组翻转后的头，p0.next 是本组翻转前的头（翻转后变尾）
            nxt0 = p0.next         # 记录本组翻转前的头（翻转后将作为尾），用于更新 p0
            nxt0.next = cur        # 连接反转段与后续未处理部分
            p0.next = pre          # 连接前驱与反转后的头
            p0 = nxt0              # p0 移到本组尾部，为下一组做准备

            # pre = None           # 下一组翻转前重置 pre
            n -= k                 # 已处理 k 个节点

        return dummy.next
```

## [排序链表](https://leetcode.cn/problems/sort-list/)(148)


问题：给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。

分析：递归实现归并排序，并且利用辅助函数 [`middleNode`](#[链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)(876)) 和 [`mergeTwoLists`](#[合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)(21))

* 函数定义：归并排序链表，并返回头节点
* 边界条件：单个节点直接返回本身（`middleNode`）
* 转移关系：根据中间节点分为两个子列表，分别递归进行归并排序，再合并两个有序列表（`mergeTwoLists`）

复杂度：`O(nlog(n))` `O(log(n))`

```python
class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        """ 返回链表的中间节点(876) """
        slow = fast = head
        while fast.next and fast.next.next:  # 偶数返回第一个节点
            # while fast and fast.next:      # 偶数返回第二个节点
            slow = slow.next
            fast = fast.next.next
        return slow

    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        """ 合并两个有序列表(21) """
        dummy = ListNode()   # 虚拟头节点，统一处理头节点为空的情况
        cur = dummy
        
        while list1 and list2:  # 双指针遍历，按升序拼接
            if list1.val < list2.val:
                cur.next = list1
                list1 = list1.next
            else:
                cur.next = list2
                list2 = list2.next
            cur = cur.next
        
        cur.next = list1 or list2  # 将剩余的某段链表直接接上
        return dummy.next

    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        """ 归并排序 """
        if not head or not head.next:  # 递归边界条件
            return head

        mid = self.middleNode(head)     # 获取第一个中间节点（第二个不好拆分）
        left, right = head, mid.next    # 将链表分为左右两段
        mid.next = None                 # 左链表断开

        leftHead = self.sortList(left)  # 递归左右链表进行归并排序
        rightHead = self.sortList(right)
        head = self.mergeTwoLists(leftHead, rightHead)  # 合并两个有序链表
        return head
```

## [LRU 缓存](https://leetcode.cn/problems/lru-cache/)(146)


问题：请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

分析：通过 **哈希表 + 双向链表** 维护所有缓存中的 `k-v` 对：首先使用哈希表定位缓存项在双向链表中的位置，再将其移动到双向链表头部

* 双向链表按照被**使用的顺序**存储了这些 `k-v` 对，靠近头部的 `k-v` 对是**最近**使用的，而靠近尾部的 `k-v` 对是**最久未使用**的。
* 哈希表通过缓存数据的 `key` 映射到其在双向链表中的位置。

双向链表使用**伪头部** `dummyHead` 和**伪尾部** `dummyTail` 标记界限，使得添加节点和删除节点的时无需检查相邻的节点是否存在。

复杂度：`O(1)` `O(capacity)`

```python
class LRUCache:
    class Node:  # 定义双向链表
        def __init__(self, key: int, value: int):
            self.key = key
            self.value = value
            self.pre = None
            self.next = None

    def __init__(self, capacity: int):
        self.capacity = capacity  # 最大容量
        self.size = 0             # 当前容量
        self.map = {}             # hashmap+双向链表 映射k-v对

        self.dummyHead = self.Node(-1, -1)  # 虚拟头节点（辅助头插）
        self.dummyTail = self.Node(-1, -1)  # 虚拟尾节点（辅助尾删）
        self.dummyHead.next = self.dummyTail  # 连接虚拟头节点和虚拟尾节点
        self.dummyTail.pre = self.dummyHead

    def get(self, key: int) -> int:  # O(1)
        tmp = self.map.get(key)  # 先判断是否存在
        if tmp is not None:      # 存在
            self.delete(tmp)     # 先删除
            self.insertToHead(tmp)  # 再头插法
            return tmp.value
        return -1                # 不存在，返回-1

    def put(self, key: int, value: int) -> None:  # O(1)
        tmp = self.map.get(key)  # 先判断是否存在
        if tmp is not None:      # 存在
            self.delete(tmp)     # 先删除
            tmp.value = value    # 修改value
            self.insertToHead(tmp)  # 再头插法
        else:                    # 不存在
            newNode = self.Node(key, value)  # 创建新节点准备插入
            self.insertToHead(newNode)       # 头插法插入
            self.map[key] = newNode          # 同时更新哈希表
            self.size += 1
            if self.size > self.capacity:    # 再判断是否超过容量，超过则删除尾节点
                tail = self.dummyTail.pre    # 利用虚拟尾节点快速定位并删除
                self.delete(tail)
                self.map.pop(tail.key)       # 同时更新哈希表
                self.size -= 1

    def delete(self, tmp: "LRUCache.Node") -> None:
        """ 利用双向链表快速删除节点 """
        tmp.pre.next = tmp.next   # 前一个节点指向后一个节点
        tmp.next.pre = tmp.pre    # 后一个节点指向前一个节点

    def insertToHead(self, tmp: "LRUCache.Node") -> None:
        """ 将节点使用头插法（虚拟头节点作辅助） """
        tmp.next = self.dummyHead.next  # 先连接后面的节点，因为dummyHead指针在前面，而后面的需要利用前面
        self.dummyHead.next.pre = tmp
        self.dummyHead.next = tmp       # 后面的连接完成后，对前面的指针没有影响
        tmp.pre = self.dummyHead
```

# 二叉树

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## [二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)(94)

问题：给定一个二叉树的根节点 `root` ，返回 *它的 **中序** 遍历* 。

解法 1：递归

* 函数定义：中序遍历二叉树，`res` 记录当前遍历的节点
* 边界条件：空节点直接返回
* 转移关系：先递归到左子树，再记录当前节点，最后递归到右子树

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        res = []

        def inorder(node: Optional[TreeNode]) -> None:
            if not node:
                return
            inorder(node.left)
            res.append(node.val)
            inorder(node.right)

        inorder(root)
        return res
```

解法2：利用栈模拟递归过程，先找到最左端节点，然后处理右子树

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        res = []
        stack = []
        while stack or root:  # 若左子树处理完则栈为空，但此时右子树未处理，故需判断 root
            while root:       # 找到最左端节点
                stack.append(root)  # 一直走到最左端
                root = root.left    # 不断入栈（保证了左->根的顺序）
            root = stack.pop()      # 最左端节点出栈（视为子树的根）
            res.append(root.val)
            root = root.right       # 处理该节点的右子树（保证了根->右的顺序）
        return res
```

## [二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)(124)


问题：二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。**路径和** 是路径中各节点值的总和。给定二叉树的根节点 `root` ，返回其 **最大路径和**

分析：对每个节点，考虑两类值：

* **贡献值**（返回给父节点）：从该节点向下走，最多选一边（左或右），因为路径不能分叉。子树贡献若为负，直接当作 0
* **以该节点为拐点的完整路径**：node.val + left_gain + right_gain，可以左右都选，用来更新全局最大值。

注意：ans 初始设为 -inf，保证全是负数时也能正确返回（至少选一个节点）

复杂度：`O(n)` `O(h)`

```python
from math import inf

class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        ans = -inf  # 全局最大路径和（路径可不经过根）# float("-inf")

        def gain(node: Optional[TreeNode]) -> int:
            """ 返回“从 node 出发、向下走（只能选左或右其中一边）”所能获得的最大贡献值，
            	至少包含 node 节点，用于给 node 的父节点拼接路径 """
            nonlocal ans
            if node is None:
                return 0
            left = max(gain(node.left), 0)  # 子树贡献为负就不选，相当于截断
            right = max(gain(node.right), 0)
            ans = max(ans, node.val + left + right)  # 以 node 为“拐点”的路径：node + 左贡献 + 右贡献
            return node.val + max(left, right)  # 返回给父节点的只能是一条“向下单链”，不能左右都选

        gain(root)
        return ans
```

## [二叉树的逐层层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)(102)


问题：给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

分析：利用队列存储每一层节点，出队时将下一层的左右孩子入队，同时用列表记录每一层的节点依次放入二维列表

复杂度：`O(n)` `O(n)`

```python
from collections import deque

class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        res = []
        if root is None:
            return res

        q = deque([root])  # 双端队列，根节点入队
        while q:
            n = len(q)                 # 获取当前队列（当前层）的结点数量
            vals = []                  # 存当前层的值
            for _ in range(n):         # 将当前层全部出队，并将下一层全部入队
                node = q.popleft()     # 队头元素出队
                vals.append(node.val)  # 出队时访问
                if node.left:          # 将该结点的左右孩子入队（下一层）
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            res.append(vals)  # 将当前层元素加入答案中

        return res
```

## [二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)(230)


问题：给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 小的元素（从 1 开始计数）。

分析：递归（中序遍历），中序遍历二叉搜索树结果单调不减，返回第 `k` 个即可

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        stack = []  # BST 的中序遍历是递增序列，第 k 个访问到的节点就是第 k 小
        while stack or root:  # 若左子树处理完则栈为空，但此时右子树未处理，故需判断 root
            while root:
                stack.append(root)  # 一直走到最左端
                root = root.left    # 不断入栈（保证了左->根的顺序）
            root = stack.pop()      # 最左端节点出栈（视为子树的根）
            k -= 1
            if k == 0:
                return root.val
            root = root.right       # 处理该节点的右子树（保证了根->右的顺序）
```

## [二叉树的最近公共祖先-LCA](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)(236)


问题：给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

分析：递归，由于不能判断 `p` 和 `q` 在哪棵子树，故每次都需递归左右子树

* 函数定义：对以 `root` 为根节点的子树，返回 `p` 和 `q` 的最近公共祖先，**否则返回所包含的 `p` 或 `q` 单个节点**
* 边界条件：返回当前节点
  * 当前节点为空
  * 当前节点是 `p `（会一直向上返回）
  * 当前节点是 `q `（会一直向上返回）

* 转移关系：先递归到左右子树，再判断
  * 左右子树均不为空：表明 `p` 和 `q` 分布在 `root` 的左右子树，返回当前节点作为最近公共祖先
  * 左右子树有一个不为空：表明 `root` 的子树仅有单个节点 `p` 或 `q` ，返回该节点以便后续判断最近公共祖先
  * 左右子树均为空：表明 `root` 的子树无目标节点，返回 `null`（可合并到上一种情况）

  复杂度：`O(n)` `O(h)`

```python
class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        if root in (None, p, q):  # 边界条件：空节点，或当前节点就是 p / q
            return root
        left = self.lowestCommonAncestor(root.left, p, q)  # 类似先序遍历
        right = self.lowestCommonAncestor(root.right, p, q)
        if left and right:  # p、q 分别在左右子树，当前 root 就是最近公共祖先
            return root
        return left or right  # 否则返回非空的那一侧（表示 p、q 都在同一侧，或只找到其中一个）
```

# 图论

## [岛屿数量](https://leetcode.cn/problems/number-of-islands/)(200)


问题：给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。此外，你可以假设该网格的四条边均被水包围。

分析：扫描网格，遇到一个 '1' 就发现一座新岛，答案 +1。从该点出发做 BFS/DFS，把所有连通的 '1' 都标记成 '0'，避免重复计数。

复杂度：`O(mn)` `O(mn)`

```python
class Solution:  # DFS 解法，需要用到递归/栈
    def numIslands(self, grid: List[List[str]]) -> int:
        m, n = len(grid), len(grid[0])

        def dfs(i, j):
            """ 从 (i, j) 出发，把与它连通的所有陆地 1 都改成 0 """
            if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] != "1":
                return
            grid[i][j] = "0"  # 把陆地 1 改成 0
            dfs(i + 1, j)
            dfs(i - 1, j)
            dfs(i, j + 1)
            dfs(i, j - 1)

        ans = 0
        for i in range(m):
            for j in range(n):
                if grid[i][j] == "1":  # 遇到一个还没访问过的陆地，说明发现了一座新岛
                    ans += 1
                    dfs(i, j)  # 把这整座岛都标记掉

        return ans
```

```python
from collections import deque

class Solution:  # BFS 解法，需要用到队列
    def numIslands(self, grid: List[List[str]]) -> int:
        m, n = len(grid), len(grid[0])
        ans = 0
        for i in range(m):
            for j in range(n):
                if grid[i][j] != "1":
                    continue
                ans += 1  # 扫描网格，遇到一个 '1' 就发现一座新岛，答案 +1
                # 从该点出发做 BFS，把所有连通的 '1' 都标记成 '0'（相当于把整座岛淹掉），避免重复计数
                grid[i][j] = "0"  # 先把起点标记为已访问，并加入队列
                q = deque([(i, j)])
                while q:
                    x, y = q.popleft()
                    for dx, dy in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                        nx, ny = x + dx, y + dy
                        if 0 <= nx < m and 0 <= ny < n and grid[nx][ny] == "1":
                            grid[nx][ny] = "0"
                            q.append((nx, ny))
        return ans
```

## [腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)(994)

问题：在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：值 `0` 代表空单元格；值 `1` 代表新鲜橘子；值 `2` 代表腐烂的橘子。每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。

分析：多源 BFS，所有初始腐烂橘子同时作为起点，每一层 BFS 表示过了 1 分钟，向四周感染一圈新鲜橘子。

注意：“岛屿数量(200)”的多源 BFS **未知且无需同时做**，而此题的多源 BFS **已知且需要同时做**，因此需要**按层次出队**。

复杂度：`O(mn)` `O(mn)`

```python
from collections import deque

class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        m, n = len(grid), len(grid[0])
        q = deque()
        fresh = 0
        
        for i in range(m):  # 初始化：统计新鲜橘子数量，并把所有腐烂橘子作为 BFS 起点入队
            for j in range(n):
                if grid[i][j] == 2:
                    q.append((i, j))
                elif grid[i][j] == 1:
                    fresh += 1

        minutes = 0
        while q and fresh > 0:
            for _ in range(len(q)):  # 当前这一层表示“这一分钟内会扩散的所有腐烂橘子”
                x, y = q.popleft()
                for dx, dy in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                    nx, ny = x + dx, y + dy
                    if 0 <= nx < m and 0 <= ny < n and grid[nx][ny] == 1:
                        grid[nx][ny] = 2
                        fresh -= 1
                        q.append((nx, ny))
            minutes += 1

        return minutes if fresh == 0 else -1
```

## [单词搜索](https://leetcode.cn/problems/word-search/)(79)

问题：给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。单词必须按照字母顺序，通过相邻的单元格内的字母构成，同一个单元格内的字母不允许被重复使用。

分析：枚举每个格子作为起点回溯。dfs(i, j, k) 表示当前走到 (i, j)，正在匹配 word[k]。若当前字符匹配，则继续上下左右搜索下一个字符

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        m, n = len(board), len(board[0])
        visited = [[False] * n for _ in range(m)]  # 访问标记数组

        def dfs(i: int, j: int, k: int) -> bool:
            """ 判断(i,j)位置出发 word[k:]是否有效 """
            if board[i][j] != word[k]:  # 当前格子字符不匹配
                return False
            
            if k == len(word) - 1:  # 已经匹配到最后一个字符
                return True

            visited[i][j] = True  # 标记当前格子已访问
            # word[k] 有效，遍历判定 word[k:] 是否有效，尝试4个方向
            for dx, dy in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                ni, nj = i + dx, j + dy
                if 0 <= ni < m and 0 <= nj < n and not visited[ni][nj]:  # 未超边界且未被使用
                    if dfs(ni, nj, k + 1):  # 若 word[k+1:] 有效且当前字符匹配，则 word[k:] 有效
                        return True

            visited[i][j] = False  # 恢复现场
            return False

        for i in range(m):  # 尝试每个格子作为 word 的起点进行 dfs 搜索
            for j in range(n):
                if dfs(i, j, 0):
                    return True
        return False
```

## [课程表](https://leetcode.cn/problems/course-schedule/)(207)


问题：你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

分析：把课程关系看成有向图：b -> a 表示学 a 前要先学 b。问能否学完全部课程，本质就是判断图中是否有环。用拓扑排序：不断学习“入度为 0”的课程；如果最后能处理完全部课程，说明无环，可以学完。

复杂度：`O(n)` `O(n)`

```python
from collections import deque

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        # 拓扑排序（BFS / Kahn），若图中无环，则一定能按某种顺序学完所有课程
        graph = [[] for _ in range(numCourses)]  # 邻接表：b -> a
        indegree = [0] * numCourses              # 每门课的入度（还需要多少先修课）
        for a, b in prerequisites:
            graph[b].append(a)
            indegree[a] += 1

        # 先把所有“没有先修要求”的课程入队
        q = deque(i for i in range(numCourses) if indegree[i] == 0)
        taken = 0
        while q:
            b = q.popleft()
            taken += 1
            for a in graph[b]:  # 学完当前课后，它指向的后续课程入度减 1
                indegree[a] -= 1
                if indegree[a] == 0:
                    q.append(a)

        return taken == numCourses
```

## [寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)(287)


问题：给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。

分析：**「Floyd 判圈算法」快慢指针：建图，每个位置 i 连一条 i→nums[i] 的边。**由于存在的重复的数字 target，因此 target 这个位置一定有起码两条指向它的边，因此整张图一定存在环，且我们要找到的 target 就是这个环的入口，问题等价于环形链表 II(142)。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        # 1) Floyd 判圈：把 nums 看成 next 指针：i -> nums[i]
        slow = fast = 0
        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]
            if slow == fast:
                break

        # 2) 找环入口（重复数）
        p = 0
        while p != slow:
            p = nums[p]
            slow = nums[slow]
        return p  # 最后得到环入口的下标，根据我们的建图方式，恰好等于重复数的值 target
```

## [实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)(208)

问题：**[Trie](https://baike.baidu.com/item/字典树/9825209?fr=aladdin)**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补全和拼写检查。请你实现 Trie 类：

- `Trie()` 初始化前缀树对象。
- `void insert(String word)` 向前缀树中插入字符串 `word` 。
- `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false` 。
- `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix` 返回 `true` ；否则返回 `false`

分析：Trie 的每个节点表示一个前缀，children 存后续字符分支，is_end 标记是否正好有单词在这里结束。

```python
class Trie:
    def __init__(self):
        self.children = {}   # 当前节点的子节点映射：字符 -> Trie 节点
        self.is_end = False  # 标记是否有单词在该节点结束

    def insert(self, word: str) -> None:
        node = self
        for ch in word:  # 沿字符逐层创建/下探
            if ch not in node.children:
                node.children[ch] = Trie()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end  # search 要求整条路径存在且最后 is_end=True

    def startsWith(self, prefix: str) -> bool:
        node = self
        for ch in prefix:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return True
```

# 回溯

## [电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)(17)

问题：给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

分析：

* 当前操作：枚举 `path[i]` 填入字符
* 子问题：构造字符串 `>= i` 的部分
* 下一个子问题：构造字符串 `>= i + 1` 的部分

复杂度：`O(n4`^n^`)` `O(n)` 其中 `n` 为 `digits` 的长度。最坏情况下每次需要枚举 `4` 个字母，递归次数为一个满四叉树的节点个数，那么一共会递归 `O(4`^n^`)` 次（等比数列和），再算上加入答案时需要 `O(n)` 的时间，所以时间复杂度为 `O(n4`^n^`)`。

```python
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        mapping = ["", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"]
        n = len(digits)
        res = []
        path = [""] * n  # 长度固定，则用 path 数组

        def dfs(i: int) -> None:
            """ 含义是字符串下标 >= i 的子问题 """
            if i == n:
                res.append("".join(path))
                return

            for ch in mapping[ord(digits[i]) - ord('0')]:  # ord('0') == 48
                path[i] = ch  # 当前操作：枚举 path[i] 填入字符
                dfs(i + 1)    # 下一个子问题：构造字符串 >= i+1 的部分

        dfs(0)
        return res
```

## [子集](https://leetcode.cn/problems/subsets/)(78)

问题：给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的**子集（幂集）**。解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

解法1：**子集型回溯**（输入视角）：每个元素可以选/不选

* 当前操作：枚举第 `i` 个数选或不选
* 子问题：从下标 `>= i` 的数字中构造子集
* 下一个子问题：从下标 `>= i + 1` 的数字中构造子集

复杂度：`O(n2`^n^`)` `O(n)` 其中 `n` 为 `nums` 的长度。每次都是选或不选，递归次数为一个满二叉树的节点个数，那么一共会递归 `O(2`^n^`)` 次（等比数列和），再算上加入答案时复制 `path` 需要 `O(n)` 的时间，所以时间复杂度为 `O(n2`^n^`)`。

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []
        path = []

        def dfs(i: int) -> None:
            """ 子问题：从下标 >=i 的数字中构造子集 """
            if i == n:
                res.append(path.copy())  # 固定答案
                return

            dfs(i + 1)            # 不选 nums[i]
            path.append(nums[i])  # 选 nums[i]
            dfs(i + 1)
            path.pop()            # 恢复现场

        dfs(0)
        return res
```

解法2：**子集型回溯**（答案视角）：每次必须选一个数，关键是选哪个数

* 当前操作：枚举一个下标 `j > i` 的数字，加入 `path`
  * 由于不重复，即不考虑顺序，那么就规定一个严格递增的顺序，每次枚举必须有 `j > i`
* 子问题：从下标 `>= i` 的数字中构造子集
* 下一个子问题：从下标 `>= i + 1` 的数字中构造子集

复杂度：`O(n2`^n^`)` `O(n)` 其中 `n` 为 `nums` 的长度。答案的长度为子集的个数，即 `2`^n^ ，同时每次递归都把一个数组放入答案，因此会递归 `2`^n^ 次，再算上加入答案时复制 `path` 需要 `O(n)` 的时间，所以时间复杂度为 `O(n2`^n^`)`。

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []
        path = []

        def dfs(i: int) -> None:
            res.append(path.copy())  # 递归到的每个节点都是答案
            for j in range(i, n):  # 从 i 开始枚举当前要选的数字
                path.append(nums[j])
                dfs(j + 1)
                path.pop()  # 恢复现场

        dfs(0)
        return res
```

## [分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)(131)

问题：给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**子集型回溯**：转化为**枚举逗号的位置**，截取逗号间子串并判断回文，**注意最后一个逗号一定要取到（保证回文才能加入答案）**

解法2**（答案视角）：枚举子串结束位置**

* 当前操作：枚举一个 `j > i` 的回文子串 `s[i...j]` 加入 `path`
* 子问题：从下标 `>= i` 的后缀中构造回文分割
* 下一个子问题：从下标 `>= j + 1` 的后缀中构造回文分割

复杂度：`O(n2`^n^`)` `O(n)` 其中 `n` 为 `s` 的长度。答案的长度至多为逗号子集的个数，即 `O(2`^n^`)` ，因此会递归 `O(2`^n^`)` 次，再算上判断回文和加入答案时需要 `O(n)` 的时间，所以时间复杂度为 `O(n2`^n^`)`。

```python
class Solution:
    def partition(self, s: str) -> List[List[str]]:
        n = len(s)
        res = []
        path = []

        def is_palindrome(left: int, right: int) -> bool:
            """ 相向双指针判断是否回文 """
            while left < right:
                if s[left] != s[right]:
                    return False
                left += 1
                right -= 1
            return True

        def dfs(i: int) -> None:
            """ i 表示当前这段回文子串的开始位置 """
            if i == n:
                res.append(path.copy())
                return
            
            for j in range(i, n):  # 枚举 j 作为子串的结束部分
                if is_palindrome(i, j):  # 若不是回文子串则只能不选，继续递归
                    path.append(s[i:j + 1])  # 实际上是截取了 [i, j+1) == [i, j] 的子串
                    dfs(j + 1)
                    path.pop()  # 恢复现场

        dfs(0)
        return res
```

## [组合](https://leetcode.cn/problems/combinations/)(77)

问题：给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。你可以按 **任何顺序** 返回答案。

**组合型回溯**（答案视角）：枚举下一个数选哪个

复杂度：`O(k*C(n,k))` `O(k)`

```python
class Solution:
    def combine(self, n: int, k: int) -> List[List[int]]:
        res = []
        path = []

        def dfs(i: int) -> None:
            if len(path) == k:  # path 长度达到 k，说明找到一种组合
                res.append(path.copy())
                return
            
            for j in range(i, n + 1):  # 从 i 开始枚举当前要选的数字
                path.append(j)
                dfs(j + 1)
                path.pop()  # 恢复现场

        dfs(1)
        return res
```

## [组合总和](https://leetcode.cn/problems/combination-sum/)(39)

问题：给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的 所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。`candidates` 中的 **同一个** 数字可以 **无限制重复被选取** 。如果至少一个数字的被选数量不同，则两种组合是不同的。 对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

**组合型回溯**（答案视角）：枚举选哪个

```python
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()  # 排序后便于剪枝
        n = len(candidates)
        res = []
        path = []

        def dfs(i: int, remain: int) -> None:
            if remain == 0:
                res.append(path.copy())
                return

            for j in range(i, n):
                x = candidates[j]
                if x > remain:      # 剪枝：后面的数只会更大
                    break
                path.append(x)
                dfs(j, remain - x)  # 递归到 j 而不是 j+1，表示当前数可以重复选取
                path.pop()          # 恢复现场

        dfs(0, target)
        return res
```

## [组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)(40)

问题：给定一个候选人编号的集合 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。`candidates` 中的每个数字在每个组合中只能使用 **一次** 。**注意：**解集不能包含重复的组合。

**组合型回溯**（答案视角）：枚举选哪个

```python
class Solution:
    def combinationSum2(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()  # 排序方便剪枝和去重
        n = len(candidates)
        res = []
        path = []

        def dfs(i: int, remain: int) -> None:
            if remain == 0:
                res.append(path[:])
                return
            
            for j in range(i, n):
                x = candidates[j]
                if j > i and candidates[j] == candidates[j - 1]:  # 去掉重复项，只取第一个数进行递归
                    continue  # 保证了搜索时不重复，比得到答案再去重效率更高
                if x > remain or (n - j) * candidates[-1] < remain:
                    break  # 剪枝：当前数字过大或后续数字累加不足 target
                path.append(x)
                dfs(j + 1, remain - x)  # 递归到 j+1，不能重复选取
                path.pop()              # 恢复现场

        dfs(0, target)
        return res
```



## [括号生成](https://leetcode.cn/problems/generate-parentheses/)(22)


问题：数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

**组合型回溯**：`C(2n,n)`，**转化为从 `2n` 个位置中选 `n` 个位置放左括号，左括号等价于选，右括号等价于不选**

解法1：**输入视角：枚举选左括号还是右括号**

* 当前操作：枚举 `path[i]` 是左括号还是右括号（选或不选）
* 子问题：构造字符串 `>= i` 的部分
* 下一个子问题：构造字符串 `>= i + 1` 的部分

复杂度：`O(n*C(2n,n))` `O(n)` 但由于左右括号的约束，实际上没有这么多叶子，根据 `Catalan` 数，只有 `C(2n,n)/(n+1)`，所以实际的时间复杂度为`O(C(2n,n))`。此外，根据阶乘的 `Stirling` 公式，时间复杂度也可以表示为 `O(4`^n^`/n`^1/2^`)`

```python
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        res = []
        path = [""] * (2 * n)  # 长度固定，用数组记录当前构造的括号串

        def dfs(i: int, open_count: int) -> None:
            """ 含义是字符串下标 >= i 的子问题 
            	因为能否枚举左括号和右括号会遭到限制，不是简单的C(n,k)，故引入左括号的数量open，在递归中描述这种限制 """
            if i == 2 * n:
                res.append("".join(path))
                return
            
            if open_count < n:  # 可以填左括号：左括号总数还没到 n
                path[i] = "("
                dfs(i + 1, open_count + 1)
            
            if i - open_count < open_count:  # 可以填右括号：当前右括号数量 < 左括号数量
                path[i] = ")"
                dfs(i + 1, open_count)

        dfs(0, 0)
        return res
```

## [全排列](https://leetcode.cn/problems/permutations/)(46)


问题：给定一个**不含重复数字**的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。

**排列型回溯**：`A(n,n)`，数组 `path` 记录路径上已选的数，集合 `s` 记录剩余未选数字

* 当前操作：从 `s` 中枚举 `path[i]` 要填的数字 `x`
* 子问题：构造排列 `>= i` 的部分，剩余未选数字集合为 `s`
* 下一个子问题：构造排列 `>= i + 1` 的部分，剩余未选数字集合为 `s - {x}`

复杂度：`O(n*n!)` `O(n)` 其中 `n` 为 `nums` 的长度。根据等比数列求和估计，搜索树中的节点个数低于 `3⋅n!`。实际上，精确值为 `A(n,0) + ... + A(n,n-1) + A(n,n) = ⌊e⋅n!⌋ = O(n!)`。此外，每个非叶节点要花费 `O(n)` 的时间遍历 `onPath` 数组，每个叶结点也要花费 `O(n)` 的时间复制 `path` 数组，因此时间复杂度为 `O(n*n!)`。

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []
        path = []
        visited = [False] * n  # visited[i] 表示 nums[i] 是否已被使用

        def dfs(i: int) -> None:
            if i == n:
                res.append(path.copy())
                return
            
            for j in range(n):  # 因为是排列型不是组合型，因此 j 从 0 开始而不是从 i 开始枚举
                if not visited[j]:
                    visited[j] = True   # 打上标记
                    path.append(nums[j])
                    dfs(i + 1)
                    visited[j] = False  # 恢复现场
                    path.pop()

        dfs(0)
        return res
```

## [全排列 II](https://leetcode.cn/problems/permutations-ii/)(47)


问题：给定一个可**包含重复数字**的序列 `nums` ，***按任意顺序*** 返回所有不重复的全排列。

核心思想：所有相同的数字只能有一种访问顺序！即 `x1,x2,...,xn` **`!vis[i - 1]` 来去重主要是通过限制一下两个相邻的重复数字的访问顺序**，如对于两个相同的数 `x1, x2`，不做去重的话，会有两种重复排列 `x1x2, x2x1`，而我们只需要取其中任意一种排列。为了达到这个目的，限制一下 `x1, x2` 访问顺序即可，即按顺序只取 `x1x2`。

方法：**遇到相同数字时，只有当 `visit nums[i-1]` 之后才去 `visit nums[i]`**，也就是如果 `!visited[i - 1]` 的话则 `continue`

注意：由于已经提前排序完毕，故所有相同的数字必相邻聚集，因此**对于若干连续的相同数字，只需规定访问 `i - 1` 后再访问 `i` 即可**

复杂度：`O(n*n!)` `O(n)`

```python
class Solution:
    def permuteUnique(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []
        path = []
        nums.sort()  # 先排序，为去重做准备
        visited = [False] * n  # visited[i] 表示 nums[i] 是否已被使用

        def dfs(i: int) -> None:
            if i == n:
                res.append(path.copy())
                return
            
            for j in range(n):  # 因为是排列型不是组合型，因此 j 从 0 开始而不是从 i 开始枚举
                if visited[j] or (j > 0 and nums[j] == nums[j - 1] and not visited[j - 1]):
                    continue  # 后半段的去重逻辑：相邻重复数字，前一个同数字未被使用则跳过
                visited[j] = True   # 打上标记
                path.append(nums[j])
                dfs(i + 1)
                visited[j] = False  # 恢复现场
                path.pop()

        dfs(0)
        return res
```

## [N 皇后](https://leetcode.cn/problems/n-queens/)(51)

问题：按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

**排列型回溯**：`A(n,n)`，**设第 `r` 行的皇后在第 `col[r]` 列，则 `col` 是 `[0...n-1]` 的全排列**（保证任意皇后不同行不同列），又因为也不能在同一斜线，利用 **/ 方向 r + c 是定值； \ 方向 r - c 是定值**的性质，放置前判断即可，`n` 皇后需要 `O(n)` 的时间

优化：利用数组或哈希表，将 `r + c` 记录到 `boolean[] diag1` 中，放皇后前判断，若为真则无法放皇后；同理将 `r - c` 记录到 `boolean[] diag2` 中，但为了避免负数需要加上 `n - 1` ，放皇后的时间复杂度由 `O(n)` 降为 `O(1)`

复杂度：`O(n`^2^`*n!)` `O(n)` 搜索树中至多有 `O(n!)` 个叶子，每个叶子生成答案每次需要 `O(n`^2^`)` 的时间，所以时间复杂度为 `O(n`^2^`*n!)`，优化后若不记录答案为`O(3*n!)`。实际上搜索树中远没有这么多叶子，`n=9` 时只有 `352` 种放置方案，远远小于 `9!=362880`。

```python
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        res = []
        visited = [False] * n          # 记录列是否已放皇后
        diag1 = [False] * (2 * n - 1)  # 记录 / 方向对角线：r + c
        diag2 = [False] * (2 * n - 1)  # 记录 \ 方向对角线：r - c + n - 1
        col = [0] * n                  # 设第 r 行的皇后放在第 col[r] 列

        def dfs(r: int) -> None:
            if r == n:  # 记录答案 O(n^2)
                board = []
                for c in col:
                    row = ["."] * n
                    row[c] = "Q"
                    board.append("".join(row))
                res.append(board)
                return

            for c in range(n):
                rc = r - c + n - 1  # 避免负数下标（Python 无需考虑）
                # 判断当前位置是否可以放皇后：列、两条对角线都不能冲突
                if not visited[c] and not diag1[r + c] and not diag2[rc]:  # 放皇后 O(n) 优化到 O(1)
                    visited[c] = diag1[r + c] = diag2[rc] = True  # 打上标记
                    col[r] = c  # 自动覆盖式记录第 r 行皇后所在列
                    dfs(r + 1)
                    visited[c] = diag1[r + c] = diag2[rc] = False  # 恢复现场

        dfs(0)
        return res
```

## [N 皇后 II](https://leetcode.cn/problems/n-queens-ii/)(52)

问题：**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n × n` 的棋盘上，并且使皇后彼此之间不能相互攻击。给你一个整数 `n` ，返回 **n 皇后问题** 不同的解决方案的数量。

**排列型回溯**：仅需计算答案是数量，递归到边界条件 `r == n` 时统计数量即可

注意：由于仅需考虑不同方案的数量，不必记录第 r 行的皇后放在第 col[r] 列，仅需用 visited 记录哪一列已经使用，确保方案合法即可

复杂度：`O(n!)` `O(n)` 根据通用公式计算为 `O(n*n!)`，但是可以精确算出全排列搜索树的节点个数为 `A(n,0) + ... + A(n,n-1) + A(n,n) = ⌊e⋅n!⌋ = O(n!)`，加入答案还需要 `O(n)` 的时间，但本题仅需求解的数量，故计算答案的时间为 `O(1)`，总时间为 `O(n!)`

```python
class Solution:
    def totalNQueens(self, n: int) -> int:
        ans = 0
        visited = [False] * n          # 记录列是否已放皇后
        diag1 = [False] * (2 * n - 1)  # 记录 / 方向对角线：r + c
        diag2 = [False] * (2 * n - 1)  # 记录 \ 方向对角线：r - c + n - 1

        def dfs(r: int) -> None:
            if r == n:
                nonlocal ans
                ans += 1
                return

            for c in range(n):
                rc = r - c + n - 1  # 避免负数下标（Python 无需考虑）
                # 判断当前位置是否可以放皇后：列、两条对角线都不能冲突
                if not visited[c] and not diag1[r + c] and not diag2[rc]:  # 放皇后 O(n) 优化到 O(1)
                    visited[c] = diag1[r + c] = diag2[rc] = True  # 打上标记
                    dfs(r + 1)
                    visited[c] = diag1[r + c] = diag2[rc] = False  # 恢复现场

        dfs(0)
        return ans
```

# 二分查找

## lower_bound 模板


```python
def lower_bound(nums: List[int], target: int) -> int:
    """ 返回 nums 中第一个 >= target 的元素下标。如果所有元素都 < target，则返回 len(nums)。闭区间写法 """
    left, right = 0, len(nums) - 1
    while left <= right:  # 闭区间 [left, right] 不为空
        mid = left + (right - left) // 2
        if nums[mid] < target:  # 范围缩小到 [mid + 1, right]
            left = mid + 1
        else:                   # 范围缩小到 [left, mid - 1]
            right = mid - 1
    return left                 # left == right + 1
```

扩展：在闭区间 [left, right] 中，寻找第一个满足 check(x) 的整数 x。若不存在，则返回初始搜索区间的右端点之后

```python
def lower_bound(nums: List[int], target: int) -> int:
    """ 返回 nums 中第一个满足 check(target) 的元素下标。 """
    left, right = 0, len(nums) - 1
    while left <= right:  # 闭区间 [left, right] 不为空
        mid = left + (right - left) // 2
        if not check(nums, mid, target):  # mid 不满足条件，答案在右侧
            left = mid + 1
        else:                             # mid 满足条件，答案可能是 mid 或更左侧
            right = mid - 1
    return left                           # left == right + 1
```

直接使用 Python 标准库 bisect：

```python
from bisect import bisect_left, bisect_right

a = [1, 2, 2, 2, 3, 4]
bisect_left(a, 2)   # → 1  第一个 >= 2 的位置（等价 lower_bound）
bisect_right(a, 2)  # → 4  第一个 >  2 的位置（等价 upper_bound）
```

## [搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)(74)


问题：给你一个满足下述两条属性的 `m x n` 整数矩阵：每行中的整数从左到右按非严格递增顺序排列；每行的第一个整数大于前一行的最后一个整数。给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

分析：因为“后一行第一个数 > 前一行最后一个数”，所以整个矩阵可以看成一个一维升序数组。直接在“虚拟数组”上套lower_bound 模板

复杂度：`O(logmn)` `O(1)`

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        m, n = len(matrix), len(matrix[0])
        
        def lower_bound(target: int) -> int:
            """ 返回“一维升序数组”中第一个 >= target 的元素下标。
            	这里把 matrix 看成长度为 m*n 的一维升序数组：idx -> matrix[idx // n][idx % n] """
            left, right = 0, m * n - 1
            while left <= right:  # 闭区间 [left, right] 不为空
                mid = left + (right - left) // 2
                x = matrix[mid // n][mid % n]
                if x < target:    # 范围缩小到 [mid + 1, right]
                    left = mid + 1
                else:             # 范围缩小到 [left, mid - 1]
                    right = mid - 1
            return left           # left == right + 1

        i = lower_bound(target)
        return i < m * n and matrix[i // n][i % n] == target
```

## [在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)(34)


问题：给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

分析：将问题转化为：查找第一个 `>= target` 的元素，如果不是 `target` 表示不存在，否则已经找到了开始位置，且结束位置一定存在。设计 `lowerBound` 函数，使用二分查找返回 `nums` 中第一个 `>= target` 的元素下标，具体如下：

* **红蓝染色法：将不满足要求的**（`< target`）**染红色**（置 `left` 左边），**满足要求的**（`>= target`）**染蓝色**（置 `right` 右边）
  * `[0, left - 1]`：已经染红色，即均不满足要求
  * `[right + 1, nums.length - 1]`：已经染蓝色，即均满足要求
  * `[left, right]`：待染色，即可能满足要求也可能不满足
* **循环不变量：`left - 1` 始终为红色，`right + 1` 始终为蓝色**
  * 终止时：`left` 指向第一个蓝色，`right` 指向最后一个红色，且 `left == right + 1`，返回 `left`
  * 若不存在满足要求的元素（全部为红色）：返回 `left` 实际上返回了数组长度 `nums.length`
* 问题拓展：利用 `lowerBound(int[] nums, int target)` 完成 `4` 种二分查找的要求
  * 返回第一个 `>= target` 的元素下标：`lowerBound(nums, target)`
  * 返回第一个 `> target` 的元素下标：`lowerBound(nums, target + 1)`
  * 返回最后一个 `< target` 的元素下标：`lowerBound(nums, target) - 1`
  * 返回最后一个 `<= target` 的元素下标：`lowerBound(nums, target + 1) - 1`
  * 注意：最后一个 `>[=]` 及第一个 `<[=]` 没有意义，因为升序数组的最后一个数一定最大，第一个数一定最小，只需判断最后一个数是否 `>[=]`，第一个数是否 `<[=]` 即可

  复杂度：`O(logn)` `O(1)`

```python
class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        def lower_bound(nums: List[int], target: int) -> int:
            """ 返回 nums 中第一个 >= target 的元素下标。如果所有元素都 < target，则返回 len(nums)。 """
            left, right = 0, len(nums) - 1
            while left <= right:  # 闭区间 [left, right] 不为空
                mid = left + (right - left) // 2
                if nums[mid] < target:  # 范围缩小到 [mid + 1, right]
                    left = mid + 1
                else:                   # 范围缩小到 [left, mid - 1]
                    right = mid - 1
            return left                 # left == right + 1

        start = lower_bound(nums, target)  # 对应 >=8 即找到第一个蓝色
        if start == len(nums) or nums[start] != target:  # 空数组 或 查找失败（找到最后或找到不等）
            return [-1, -1]
        end = lower_bound(nums, target + 1) - 1  # 对应 <=8 如果 start 存在，那么 end 必定存在
        return [start, end]
```

直接使用 Python 标准库 bisect：

```python
from bisect import bisect_left, bisect_right

class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        start = bisect_left(nums, target)  # 对应 >=8 即找到第一个蓝色
        if start == len(nums) or nums[start] != target:  # 空数组 或 查找失败（找到最后或找到不等）
            return [-1, -1]
        end = bisect_right(nums, target) - 1  # 对应 >8 前面一个位置
        # end = bisect_left(nums, target + 1) - 1  # 对应 >=9 前面一个位置
        return [start, end]
```

## [寻找峰值](https://leetcode.cn/problems/find-peak-element/)(162)

问题：峰值元素是指其值严格大于左右相邻值的元素。给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，返回 **任何一个峰值** 所在位置即可。你可以假设 `nums[-1] = nums[n] = -∞` 。对于所有有效的 `i` 都有 `nums[i] != nums[i + 1]`

分析：**红蓝染色法：红色表示目标峰顶左侧，蓝色表示目标峰顶及右侧，转化为找出第一个蓝色**

注意：目标峰顶不是第一个峰顶，而是使用二分法判断合法峰顶时步骤最少的那个。事实上，就是二分结束后的第一个蓝色节点。

* 思路为不断地缩小范围，并最终找到其中的一个峰值。由于**相邻元素不等**，且**数组必定有峰**，可以使用二分快速缩小范围。即每次二分后，可以通过比较大小确定某个半区内至少有一个解。

复杂度：`O(logn)` `O(1)`

```python
class Solution:
    def findPeakElement(self, nums: List[int]) -> int:
        """ 红蓝染色法二分：红色表示目标峰顶左侧，蓝色表示目标峰顶及右侧，返回 nums 中第一个蓝色，其一定是某个峰值 """
        left, right = 0, len(nums) - 2  # 根据定义，最右侧必然是蓝色，二分范围为[0, n-2]
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] > nums[mid + 1]:  # 大于说明M是峰顶或在峰顶右侧，染蓝色
                right = mid - 1
            else:  # 小于说明M向右爬坡可以到达峰顶（右边一定存在一个峰顶），左边舍弃，染红色
                left = mid + 1
        return left  # left == right + 1
```

## [寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)(153)


问题：已知一个长度为 `n` 的数组，预先按照升序排列，经由 `1` 到 `n` 次 **旋转** 后，得到输入数组。注意，数组 `[a[0], a[1], a[2], ..., a[n-1]]` **旋转一次** 的结果为数组 `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]` 。给你一个元素值 **互不相同** 的数组 `nums` ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 **最小元素** 。

分析：**红蓝染色法：红色表示最小值左侧，蓝色表示最小值及其右侧，转化为找出第一个蓝色。**设 `x = nums[mid]` 是现在二分取到的数。我们需要判断 `x` 和数组最小值的位置关系，谁在左边，谁在右边？把 `x` 与最后一个数 `nums[n − 1]` 比大小：

* 如果 `x > nums[n − 1]`，那么可以推出以下结论：
  * `nums` 一定被分成左右两个递增段；
  * 第一段的所有元素均大于第二段的所有元素；
  * `x` 在第一段。
  * 最小值在第二段。
  * 所以 **`x` 一定在最小值的左边（染红色）**
* 如果 `x ≤ nums[n − 1]`，那么 `x` 一定在第二段。（或者 `nums` 就是递增数组，此时只有一段。）
  * **x 要么是最小值，要么在最小值右边（染蓝色）**

  所以，**只需要比较 `x` 和 `nums[n − 1]` 的大小关系，就间接地知道了 `x` 和数组最小值的位置关系**，从而不断地缩小数组最小值所在位置的范围，二分找到数组最小值。

  复杂度：`O(logn)` `O(1)`

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        """ 红蓝染色法二分：返回 nums 中第一个满足 nums[i] <= nums[-1] 的元素下标。 """
        left, right = 0, len(nums) - 1
        while left <= right:  # 闭区间 [left, right] 不为空
            mid = left + (right - left) // 2
            if nums[mid] > nums[-1]:  # mid 不满足条件，答案在右侧
                left = mid + 1
            else:                     # mid 满足条件，答案可能是 mid 或更左侧
                right = mid - 1
        return nums[left]             # left == right + 1
```

## [搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)(33)


问题：整数数组 `nums` 按升序排列，数组中的值 **互不相同** 。在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，如果 `nums` 中存在目标值 `target` ，则返回下标，否则返回 `-1`。

分析：两次二分查找

* 先由 [`findMin`](#[寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)(153)) 寻找旋转排序数组中的最小值 `index`，再通过比较 `target` 与 `nums[n - 1]` 确定其在哪一段
* 最后在左端 `[0, index - 1]` 或右端 `[index, nums.length - 1]` 中二分查找即可

注意：也可以不确定在哪一段，把数组“逻辑上拉直”后直接对整个数组进行二分查找

复杂度：`O(logn)` `O(1)`

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        n = len(nums)
        # 1) 找旋转点 k：最小值下标，即第一个满足 nums[i] <= nums[-1] 的位置
        left, right = 0, n - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] > nums[-1]:  # mid 不满足条件，答案在右侧
                left = mid + 1
            else:                     # mid 满足条件，答案可能是 mid 或更左侧
                right = mid - 1
        k = left  # min_index

        # 2) 在“逻辑拉直后的有序数组”上找第一个 >= target 的位置，拉直后第 i 个位置对应原数组下标 (i + k) % n
        left, right = 0, n - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[(mid + k) % n] < target:  # mid 不满足条件，答案在右侧
                left = mid + 1
            else:                             # mid 满足条件，答案可能是 mid 或更左侧
                right = mid - 1
        
        if left == n:
            return -1
        idx = (left + k) % n
        return idx if nums[idx] == target else -1
```

## [寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)(4)


问题：给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

分析：把两个数组切成左右两部分，要求：左半部分元素个数等于总长度的一半（奇数时左边多一个）；左半部分最大值 <= 右半部分最小值。把二分目标设为：**找第一个满足 left2 <= right1 的切分位置 i**。由于二分查找需要一个**单调判定条件**：

* left2 <= right1：观察 i 增大时，left1 变大，right1 变大，j = total_left - i 变小，left2 变小，right2 变小，**所以 left2 <= right1 会随着 i 增大变得更容易成立**。也就是它是一个“**从 False 到 True**”的单调条件，因此可以二分找**第一个满足 left2 <= right1 的位置**。
* left1 <= right2：会随着 i 增大变得更难成立，它是一个“从 True 到 False”的单调条件，需要找**最后一个满足** left1 <= right2 的位置。

注意：一定要让较短数组作为二分对象，保证 j 不越界；二分的是“切分位置”，搜索区间为 [0, m]；边界要用正负无穷处理。

复杂度：`O(log(m+n))` `O(1)`

![image-20260310220552441](./assets/image-20260310220552441.png)

```python
from math import inf

class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        if len(nums1) > len(nums2):  # 让 nums1 更短，二分范围更小，同时保证 j 不越界
            nums1, nums2 = nums2, nums1
        m, n = len(nums1), len(nums2)
        total_left = (m + n + 1) // 2

        def check(i: int) -> bool:
            """ check(i): 切分位置 i 是否已经满足 nums2[j-1] <= nums1[i]，其中 j = total_left - i """
            j = total_left - i
            left2 = nums2[j - 1] if j > 0 else -inf  # float("-inf")
            right1 = nums1[i] if i < m else inf
            return left2 <= right1  # 另一个条件 left1 <= right2 可推导得出（仅对于答案位置）
        
        def lower_bound() -> int:
            """ 返回 nums1 的切分位置 i，使其成为第一个满足 check(i) 的位置 """
            left, right = 0, m  # 这里 i 的合法范围是 [0, m]，表示切分的位置
            while left <= right:  # 闭区间 [left, right] 不为空
                mid = left + (right - left) // 2
                if not check(mid):  # mid 不满足条件，答案在右侧
                    left = mid + 1
                else:               # mid 满足条件，答案可能是 mid 或更左侧
                    right = mid - 1
            return left             # left == right + 1

        i = lower_bound()
        j = total_left - i
        left1 = nums1[i - 1] if i > 0 else -inf
        right1 = nums1[i] if i < m else inf
        left2 = nums2[j - 1] if j > 0 else -inf
        right2 = nums2[j] if j < n else inf
        if (m + n) % 2:
            return max(left1, left2)
        else:
        	return (max(left1, left2) + min(right1, right2)) / 2
```

# 栈/队列

## [用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)(225)

问题：请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（`push`、`top`、`pop` 和 `empty`）。

分析：先将新元素入队，再将队中原有元素依次出队再入队，使得新元素位于队头

```python
class MyStack:
    def __init__(self):
        self.queue = deque()  # 使用一个队列模拟栈

    def push(self, x: int) -> None:
        n = len(self.queue)   # 获取队列中原有元素数量
        self.queue.append(x)  # 新元素入队
        for _ in range(n):    # 将原队列的元素依次出队再入队
            self.queue.append(self.queue.popleft())  # 新元素最终位于队首

    def pop(self) -> int:  # 队首元素即栈顶，出栈
        return self.queue.popleft()

    def top(self) -> int:  # 队首元素即栈顶
        return self.queue[0]

    def empty(self) -> bool:  # 队列为空则栈空
        return not self.queue
```

## [用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/)(232)

问题：请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）

分析：仅输入到 `inStack`，仅从 `outStack` 中输出（为空则先把 `inStack` 中所有元素转移到 `outStack`）

```python
class MyQueue:
    def __init__(self):
        self.inStack = []   # 输入栈，用于 push
        self.outStack = []  # 输出栈，用于 pop / peek

    def push(self, x: int) -> None:  # 仅输入到 inStack
        self.inStack.append(x)

    def pop(self) -> int:  # 仅从 outStack 中输出
        if not self.outStack:  # 若 outStack 为空，则把 inStack 所有元素倒入 outStack
            while self.inStack:
                self.outStack.append(self.inStack.pop())
        return self.outStack.pop()

    def peek(self) -> int:  # 与 pop 类似，但不删除元素
        if not self.outStack:
            while self.inStack:
                self.outStack.append(self.inStack.pop())
        return self.outStack[-1]

    def empty(self) -> bool:  # inStack 与 outStack 都为空才说明队列为空
        return not self.inStack and not self.outStack
```

## [最小栈](https://leetcode.cn/problems/min-stack/)(155)


问题：设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。`int getMin()` 获取堆栈中的最小元素。

分析：因为栈底元素始终最后出栈，因此栈中的最小值<=栈底元素，且栈具有顺序性，每次只需记录<=当前最小值的元素

复杂度：`O(1)` `O(n)`

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = [float("inf")]  # 存入最大值，方便后续每次入栈时取最小

    def push(self, val: int) -> None:
        self.stack.append(val)
        self.min_stack.append(min(val, self.min_stack[-1]))  # 每次取当前栈的最小值和当前元素的最小值

    def pop(self) -> None:  # 主栈与辅助栈同步出栈
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

## [滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)(239)


问题：给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。返回 *滑动窗口中的最大值* 。

分析：利用双端队列存储当前滑动窗口内的元素，**越早入队的应越大（否则会被更大且生命周期更长的上位替代）**，故单调递减。

* 若当前元素可以上位替代，从**队尾出队**更小的，再从**队尾入队**当前元素；若不可上位替代则直接**队尾入队**
* 若滑动窗口范围超出当前最大值，则将最先存入的**队头元素出队**
  * 由于需要判断最早的下标是否已经超出窗口范围，故将双端队列由存储元素转为**存储下标**
* 由于队头存放当前窗口的最大值，记录答案时仅需**查询队头元素**即可

总结：**及时去掉无用数据，保证双端队列有序**

复杂度：`O(n)` `O(min(k, U))`，其中 `U` 是 `nums` 中的不同元素个数（本题至多为 `20001`）

```python
from collections import deque

class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        q = deque()  # 单调递减队列，存下标，其队头元素存储当前滑动窗口的最大值下标
        res = []
        for i, num in enumerate(nums):  # i 枚举滑动窗口的右端点
            while q and nums[q[-1]] <= num:  # 从队尾移除会被当前元素“压住”的元素（更小或相等）
                q.pop()
            q.append(i)
            if i - q[0] + 1 > k:  # 队头若已滑出窗口，则弹出
                q.popleft()
            if i >= k - 1:  # 从第一个完整窗口开始记录答案
                res.append(nums[q[0]])
        return res
```

## [有效的括号](https://leetcode.cn/problems/valid-parentheses/)(20)


问题：给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

分析：利用哈希表定义匹配括号对，遇左括号入栈，遇右括号检查栈顶元素是否与之匹配，若匹配则出栈

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def isValid(self, s: str) -> bool:
        n = len(s)
        if n & 1:  # 奇数长度一定不满足
            return False
        pairs = {'(': ')', '[': ']', '{': '}'}  # 定义匹配对
        stack = []  # 利用栈按序匹配
        for ch in s:
            if ch in pairs:  # 左括号：入栈
                stack.append(ch)
            else:  # 右括号：栈为空或与当前栈顶不匹配则不满足（peek 前需保证栈不空）
                if not stack or pairs[stack[-1]] != ch:
                    return False
                stack.pop()  # 若存在匹配对，则相应的左括号出栈
        return not stack  # 若栈为空，表明所有括号得到了正确的匹配
```

## [最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)(32)

问题：给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（格式正确且连续）括号 子串 的长度。

分析：栈里始终保存"最近一个未匹配字符的索引"作为左边界。无法匹配的 `)` 是一道"墙"，之后的有效子串绝对不能包含它，所以用它的索引作为新的左边界，替代失效的旧边界。

- 遇到 `(` → 压入索引
- 遇到 `)` → 弹出栈顶
  - **弹完栈空：说明这个 `)` 无法匹配，把它的索引压入作为新边界**
  - 弹完栈不空：当前有效串长度 = `i - stack[-1]`

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        stack = [-1]  # 哨兵：记录上一个未匹配位置
        ans = 0
        for i, c in enumerate(s):
            if c == '(':  # 左括号：入栈
                stack.append(i)
            elif c == ')':  # 右括号：无论是否匹配都要弹栈
                stack.pop()
                if not stack:  # 栈为空说明当前 ')' 无法匹配，成为新的左边界供后续计算有效串长度
                    stack.append(i)
                else:  # 栈不为空说明匹配成功，可以更新当前有效串长度
                    ans = max(ans, i - stack[-1])
        return ans
```

## [逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)(150)

问题：给你一个字符串数组 `tokens` ，表示一个根据 [逆波兰表示法](https://baike.baidu.com/item/逆波兰式/128437) 表示的算术表达式。请你计算并返回一个表示表达式值的整数。

分析：遇数字入栈，遇操作符取出栈顶的两个操作数计算，并将计算值入栈

注意：这题除法要求**向 0 截断**，所以用：int(a / b)。不能直接用 a // b，因为 Python 对负数整除是向下取整，不符合题意。

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def evalRPN(self, tokens: List[str]) -> int:
        stack = []
        for token in tokens:
            if token not in {"+", "-", "*", "/"}:  # 若为数字，则入栈
                stack.append(int(token))
            else:  # 若为符号，取出栈顶的两个操作数计算，并将计算值入栈
                b = stack.pop()  # 先弹出的是右操作数 b
                a = stack.pop()  # 后弹出的是左操作数 a
                if token == "+":
                    stack.append(a + b)
                elif token == "-":
                    stack.append(a - b)
                elif token == "*":
                    stack.append(a * b)
                else:  # 题目要求除法向 0 截断，不能直接用 a // b，因为 Python 对负数整除是向下取整
                    stack.append(int(a / b))
        return stack[-1]  # 栈中剩余的唯一元素即为计算结果
```



## [字符串解码](https://leetcode.cn/problems/decode-string/)(394)


问题：给定一个经过编码的字符串，返回它解码后的字符串。编码规则为: `k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。你可以认为原始数据不包含数字，所有的数字只表示重复的次数 `k`

![image-20260311120340160](./assets/image-20260311120340160.png)

分析：用栈处理嵌套结构：遇到 [ 把“当前已解析字符串 + 重复次数 k”入栈并清空当前；遇到 ] 弹栈，把当前串重复 k 次接回上一层。

注意：k 可能是多位数，逐位累加。

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def decodeString(self, s: str) -> str:
        stack = []      # 栈元素: (上一层已构建的字符串, 当前层重复次数 k)
        cur = []        # 当前层字符串，用 list 收集字符/片段，最后再 join（避免频繁拼接）
        k = 0           # 当前即将作用于下一对 [] 的重复次数（可能是多位数）
        for ch in s:
            if ch.isdigit():  # 解析多位数字：例如 "12[a]"
                k = k * 10 + (ord(ch) - ord('0'))  # ord('0') == 48
            elif ch == '[':  # 进入新的一层：把上一层状态入栈，然后清空当前层
                stack.append(("".join(cur), k))
                cur = []
                k = 0
            elif ch == ']':  # 当前层结束：弹出上一层，把当前层结果重复 times 次并拼回去
                prev, times = stack.pop()
                cur = [prev + "".join(cur) * times]
            else:  # 普通字母，追加到当前层
                cur.append(ch)
        return "".join(cur)
```

## [商品折扣后的最终价格](https://leetcode.cn/problems/final-prices-with-a-special-discount-in-a-shop/)(1475)

问题：给你一个数组 `prices` ，其中 `prices[i]` 是商店里第 `i` 件商品的价格。商店里正在进行促销活动，如果你要买第 `i` 件商品，那么你可以得到与 `prices[j]` 相等的折扣，其中 `j` 是满足 `j > i` 且 `prices[j] <= prices[i]` 的 **最小下标** ，如果没有满足条件的 `j` ，你将没有任何折扣。请你返回一个数组，数组中第 `i` 个元素是折扣后你购买商品 `i` 最终需要支付的价格。

分析：倒序遍历 `prices`，并用单调栈中维护当前位置右边的更小的元素列表，**从栈底到栈顶（从右到左）的元素是单调递增的**。

* 每次移动到数组中一个新的位置 `i`，就将单调栈中所有 `> prices[i]` 的元素弹出单调栈
* 从而位置 `i`右边的第一个 `<= prices[i]` 的元素即为栈顶元素，如果栈为空则说明当前位置右边没有更小的元素
* 随后将 `prices[i]` 入栈，满足了单调性

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def finalPrices(self, prices: List[int]) -> List[int]:
        n = len(prices)
        res = [0] * n  # 存储折扣后的价格
        stack = []  # 单调递增栈，用于存储右边更小的元素
        for i in range(n - 1, -1, -1):  # 从右向左遍历，保证可以快速找到右侧第一个 <= prices[i] 的元素
            while stack and stack[-1] > prices[i]:  # 弹出栈中大于当前 prices[i] 的元素，因为这些无法作为折扣
                stack.pop()
            # 栈为空，说明右边没有比 prices[i] 小的元素；栈不为空，栈顶元素就是右边第一个 <= prices[i] 的折扣
            res[i] = prices[i] if not stack else prices[i] - stack[-1]
            stack.append(prices[i])  # 当前元素入栈，维护单调递增栈
        return res
```

## [每日温度](https://leetcode.cn/problems/daily-temperatures/)(739)

问题：给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

分析：**及时去掉无用数据，保证双端队列/栈有序**

解法2：从左到右：栈中记录还没算出「下一个更大元素」的那些数（的下标）。

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        res = [0] * n
        stack = []  # 单调递减栈：存下标，temperatures[st] 从栈底到栈顶递减
        for i, t in enumerate(temperatures):  # 从左往右
            while stack and t > temperatures[stack[-1]]:  # 只要当前温度比栈顶对应温度高，就可以结算栈顶那天的答案
                j = stack.pop()
                res[j] = i - j
            stack.append(i)
        return res
```

# 堆

## [数组中的第 K 个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)(215)


问题：给定整数数组 `nums` 和整数 `k`，请返回数组中第 `k` 个最大的元素。

分析：利用优先队列，维护大小为 `k` 的小顶堆，元素>=堆顶则入堆。遍历后获得前 `k` 个最大的元素，且堆顶最小恰为第 `k` 个

复杂度：`O(nlogk)` `O(k)`

```python
import heapq

class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        heap = []  # 小顶堆，维护当前最大的 k 个元素
        for num in nums:
            if len(heap) < k:  # 堆不足 k 时直接入堆
                heapq.heappush(heap, num)
            elif num > heap[0]:  # 当前元素更大，则替换堆顶最小值
                heapq.heappop(heap)
                heapq.heappush(heap, num)
        return heap[0]  # 堆顶是“前 k 大”中的最小值，即第 k 大
```

## [前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)(347)


问题：给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。

分析：先利用哈希表统计每个元素出现的次数，再利用优先队列存储频率前 `k` 高的元素（按频率排序的小顶堆）

复杂度：`O(nlogk)` `O(n)`

```python
from collections import Counter
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        count = Counter(nums)         # 统计每个元素出现次数 {num: cnt}
        heap = []                     # 小顶堆：(cnt, num)
        for num, cnt in count.items():
            if len(heap) < k:         # 堆不足 k 时直接入堆
                heapq.heappush(heap, (cnt, num))
            elif cnt > heap[0][0]:    # 频次更大则替换堆顶（最小频次）
                heapq.heappop(heap)
                heapq.heappush(heap, (cnt, num))
        return [num for _, num in heap]
```

## [数据流的中位数](https://leetcode.cn/problems/find-median-from-data-stream/)(295)

问题：**中位数**是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是两个中间值的平均值。

分析：保持不变量：len(low) == len(high) 或 len(low) == len(high) + 1；**max(low) <= min(high)**

* 维护存储较小部分的大顶堆，若两个堆大小相等，先存入小顶堆，再转入大顶堆
* 维护存储较大部分的小顶堆，若大顶堆比小顶堆多一个，先存入大顶堆，再转入小顶堆

```python
import heapq

class MedianFinder:
    def __init__(self):
        self.low = []   # 大顶堆（用负数实现）：存较小的一半（若为奇数，多出来的中位数存于此）
        self.high = []  # 小顶堆：存较大的一半

    def addNum(self, num: int) -> None:  # O(logn)
        if len(self.low) == len(self.high):  # 偶数：即两个堆大小相等，需存入大顶堆中
            heapq.heappush(self.high, num)          # 先存入小顶堆
            heapq.heappush(self.low, -heapq.heappop(self.high)) # 再转入大顶堆(利用自动排序，转入的一定是最小的)
        else:  # 奇数：即大顶堆比小顶堆多一个，需存入小顶堆中
            heapq.heappush(self.low, -num)          # 先存入大顶堆（取负）
            heapq.heappush(self.high, -heapq.heappop(self.low)) # 再转入小顶堆(利用自动排序，转入的一定是最大的)

    def findMedian(self) -> float:  # O(1)
        if len(self.low) > len(self.high):  # 奇数：中位数在大顶堆 low 堆顶
            return float(-self.low[0])
        else:  # 偶数：两堆堆顶平均
			return (-self.low[0] + self.high[0]) / 2.0
```

# 贪心算法

## [买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)(121)


问题：给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

分析：一次遍历，用 mn 维护到当前位置为止的**最低买入价**，用 ans 维护当前能得到的**最大利润**

复杂度：`O(n)` `O(1)`

```python
from math import inf

class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        mn = inf  # 到当前为止的最低买入价
        ans = 0            # 最大利润
        for price in prices:
            ans = max(ans, price - mn)  # 更新截止今天的最大利润
            mn = min(mn, price)         # 更新最低买入价
        return ans
```

## [买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)(122)

问题：整数数组 `prices`，`prices[i]` 表示某支股票第 `i` 天的价格。在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。然而，你可以在 **同一天** 多次买卖该股票，但要确保你持有的股票不超过一股。返回 *你能获得的 **最大** 利润* 。

分析：一段连续上涨（在最低点买、最高点卖）等价于把每天上涨的差值全部加起来。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        n = len(prices)
        ans = 0
        for i in range(1, n):
            if prices[i] > prices[i - 1]:
                ans += prices[i] - prices[i - 1]
        return ans
```

## [螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)(54)


问题：给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

分析：用四个边界 top/bottom/left/right 表示当前还没遍历的“外圈矩形”。每一圈按顺序走：上右下左，然后收缩边界进入下一圈。

注意：走下边、左边前要判断 top <= bottom / left <= right，避免在单行/单列时重复添加。

复杂度：`O(mn)` `O(mn)`

```python
class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        top, bottom = 0, len(matrix) - 1
        left, right = 0, len(matrix[0]) - 1
        res = []

        while top <= bottom and left <= right:
            for j in range(left, right + 1):  # 上
                res.append(matrix[top][j])
            top += 1

            for i in range(top, bottom + 1):  # 右
                res.append(matrix[i][right])
            right -= 1

            if top <= bottom:  # 下
                for j in range(right, left - 1, -1):
                    res.append(matrix[bottom][j])
                bottom -= 1

            if left <= right:  # 左
                for i in range(bottom, top - 1, -1):
                    res.append(matrix[i][left])
                left += 1

        return res
```

## [下一个排列](https://leetcode.cn/problems/next-permutation/)(31)

问题：给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。

分析：找到从右往左的第一个“上升拐点” i（nums[i] < nums[i+1]），它左边不变、右边是尽可能大的降序尾巴。在尾巴里从右往左找第一个比 nums[i] 大的数 j 交换，让整体刚好变大。交换后，把尾巴反转成升序，使得新排列是“比原排列大且增量最小”的下一个排列。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """ Do not return anything, modify nums in-place instead. """
        n = len(nums)
        # (1) 首先从后向前查找第一个递增顺序对 (i,i+1) 若无则 i 降至-1
        i = n - 2
        while i >= 0 and nums[i] >= nums[i+1]:
            i -= 1
        
        # (2) 一般情况下需要先转换再排列顺序，i == -1 除外（说明整个数组已经是降序的最大排列）
        if i >= 0:
            j = n - 1  # 在区间 [i+1,n) 中从后向前查找第一个元素 j 满足 nums[j] > nums[i]
            while nums[j] <= nums[i]:
                j -= 1
            nums[i], nums[j] = nums[j], nums[i]  # 交换 nums[i] 与 nums[j]
        
        # (3) 此时 nums[i+1:n] 必为降序，需要将其转为升序
        left, right = i + 1, n - 1
        while left < right:
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
            right -= 1
```

## [摆动序列](https://leetcode.cn/problems/wiggle-subsequence/)(376)

问题：如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为 **摆动序列 。**第一个差（如果存在的话）可能是正数或负数。仅有一个元素或者含两个不等元素的序列也视作摆动序列。**子序列** 可以通过从原始序列中删除一些（也可以不删除）元素来获得，剩下的元素保持其原始顺序。给你一个整数数组 `nums` ，返回 `nums` 中作为 **摆动序列** 的 **最长子序列的长度** 。

分析：用 `pre` 和 `cur` 分别保存前一个和当前的差值，满足 `pre <= 0 && cur > 0 || pre >= 0 && cur < 0` 即找到一个摆动点

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def wiggleMaxLength(self, nums: List[int]) -> int:
        n = len(nums)
        ans = 1
        pre = 0  # 保存前一个差值是正还是负
        cur = 0  # 保存当前差值
        for i in range(1, n):
            cur = nums[i] - nums[i - 1]
            if (pre <= 0 and cur > 0) or (pre >= 0 and cur < 0):  # 注意 pre 需要取等号
                ans += 1
                pre = cur
        return ans
```

## [跳跃游戏](https://leetcode.cn/problems/jump-game/)(55)

问题：给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。

分析：不断更新当前能达到的最远距离 `Math.max(max, i + nums[i])`，若 `max < i` 则无法到达

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        n = len(nums)
        max_reach = 0  # 当前最远能到达的位置
        for i in range(n):
            if i <= max_reach:  # 当前位置可达
                max_reach = max(max_reach, i + nums[i])
            else:               # 当前位置不可达
                return False
        return True
```

## [跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)(45)

问题：给定一个长度为 `n` 的 **0 索引**整数数组 `nums`。初始位置为 `nums[0]`。每个元素 `nums[i]` 表示从索引 `i` 向前跳转的最大长度。换句话说，如果你在 `nums[i]` 处，你可以跳转到任意 `nums[i + j]` 处，返回到达 `nums[n - 1]` 的最小跳跃次数。

分析：不断更新当前**区间**能达到的最远距离（同上），并选择最大值作为下一个区间右端 `end = max`，表示当前这一步最远能到哪

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        n = len(nums)
        ans = 0
        end = 0        # 当前这一跳所能覆盖区间的右端点
        max_reach = 0  # 在当前区间内，下一跳最远能到的位置
        for i in range(n - 1):  # 最后一个位置不用再跳
            max_reach = max(max_reach, i + nums[i])
            if i == end:                # 当前区间遍历结束，必须再跳一步
                end = max_reach
                ans += 1
        return ans
```

## [划分字母区间](https://leetcode.cn/problems/partition-labels/)(763)

问题：给你一个字符串 `s` 。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。例如，字符串 `"ababcc"` 能够被分为 `["abab", "cc"]`，但类似 `["aba", "bcc"]` 或 `["ab", "ab", "cc"]` 的划分是非法的。注意，划分结果需要满足：将所有划分结果按顺序连接，得到的字符串仍然是 `s` 。返回一个表示每个字符串片段的长度的列表。

分析：先记录每个字符最后一次出现的位置。从左到右扫描字符串，维护当前片段的最远右端点 end。到达片段右端时可切，且段数最多

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        last = {}  # 记录每个字符最后一次出现的位置
        for i, ch in enumerate(s):
            last[ch] = i

        res = []
        start = end = 0
        for i, ch in enumerate(s):    # 从左到右扫描字符串，维护当前片段的最远右端点 end
            end = max(end, last[ch])  # 扫到字符 ch 时，当前片段必须至少延伸到 last[ch]
            if i == end:              # 到达片段右端点，可以切分，且这样切分段数最多
                res.append(end - start + 1)
                start = end + 1

        return res
```

## [合并区间](https://leetcode.cn/problems/merge-intervals/)(56)

问题：以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

分析：先按区间起点排序；排序后所有可能合并的区间会出现在相邻位置。线性扫描：若当前区间与最后一个已合并区间重叠，就更新右端点，否则直接加入。

复杂度：`O(nlogn)` `O(n)`

```python
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        intervals.sort(key=lambda x: x[0])  # 按照区间的左端点排序，使得可以合并的区间一定是连续的
        res = []
        for interval in intervals:
            if not res or res[-1][1] < interval[0]:  # 不重叠，开启新区间
                res.append(interval)
            else:  # 有重叠，与上一区间进行合并（更新右端点）
                res[-1][1] = max(res[-1][1], interval[1])
        return res
```

## [无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)(435)

问题：给定一个区间的集合 `intervals` ，其中 `intervals[i] = [starti, endi]` 。返回 *需要移除区间的最小数量，使剩余区间互不重叠* 。**注意** 只在一点上接触的区间是 **不重叠的**。

分析：转化问题：保存数量最多的不重叠区间。贪心：按照每个区间的**结束时间**升序排序，一定能得到最多的不重叠区间

复杂度：`O(nlogn)` `O(1)`

```python
class Solution:
    def eraseOverlapIntervals(self, intervals: List[List[int]]) -> int:
        n = len(intervals)
        intervals.sort(key=lambda x: x[1])  # 按区间结束时间升序排序，一定能得到最多的不重叠区间
        i = 0      # 当前选择的最后一个不重叠区间的下标
        count = 1  # 不重叠区间的数量
        for j in range(1, n):  # 遍历所有区间
            if intervals[j][0] >= intervals[i][1]:  # 当前区间与前一个选择的区间不重叠
                count += 1
                i = j  # 更新当前选择区间
        return n - count  # 需要移除的区间数量
```

# 动态规划

## [打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/)(213)

问题：这个地方所有的房屋都 **围成一圈**，相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。给定一个代表每个房屋存放金额的非负整数数组，计算今晚能够偷窃到的最高金额。

分析：环形数组的关键在于：**第一个房子和最后一个房子不能同时偷**。所以把原问题拆成两个线性子问题：**只考虑房屋 0 ~ n-2；只考虑房屋 1 ~ n-1**。分别用“打家劫舍 I”的标准 DP 求解，再取最大值。

```python
class Solution:
    def rob1(self, nums: List[int]) -> int:
        """ 线性排列 """
        n = len(nums)
        dp = [0] * (n + 1)  # dp[i] 定义为从前 i 个房子中得到的最大金额和
        dp[1] = nums[0]  # 初始化
        
        for i in range(2, n + 1):
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1])
        
        return dp[n]
    

    def rob(self, nums: List[int]) -> int:
        """ 环状排列 """
        n = len(nums)
        if n == 1:
            return nums[0]
        else:
            return max(self.rob1(nums[1:]), self.rob1(nums[:-1]))
```

## [完全平方数](https://leetcode.cn/problems/perfect-squares/)(279)

问题：给你一个整数 `n` ，返回 *和为 `n` 的完全平方数的最少数量* 。

分析：设 dp[i] 表示“和为 i 的完全平方数的最少数量”。对于每个 i，枚举最后使用的完全平方数 j x j，则可以从 i - j x j 转移过来：dp[i] = min(dp[i], dp[i - j*j] + 1)，本质上是一个完全背包型动态规划。

```python
from math import inf

class Solution:
    def numSquares(self, n: int) -> int:
        dp = [inf] * (n + 1)  # dp[i] 表示和为 i 的完全平方数的最少数量
        dp[0] = 0  # 第 0 位留空但有实际意义：dp[0] = 0 表示组成 0 不需要任何完全平方数
        
        # 状态转移：枚举最后一个选的完全平方数 j*j，如果 j*j <= i，那么可以由 i - j*j 转移过来
        for i in range(1, n + 1):
            j = 1
            while j * j <= i:
                dp[i] = min(dp[i], dp[i - j * j] + 1)
                j += 1
        
        return dp[n]
```

## [单词拆分](https://leetcode.cn/problems/word-break/)(139)

问题：给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

分析：设 dp[i] 表示前 i 个字符 s[0:i] 能否被成功拆分。枚举最后一个单词的起点，如果前面的部分已经能拆分，且最后这一段在字典中，那么当前前缀也能拆分成功。

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)
        st = set(wordDict)
        dp = [False] * (n + 1)  # dp[i] 表示 s 的前 i 个字符 s[0:i] 是否可以被成功拆分
        dp[0] = True  # 第 0 个位置留空：dp[0] = True 表示空字符串可以被成功拆分

# 状态转移：枚举最后一个单词的分割点j，如果前j个字符可以被成功拆分，并且子串 s[j:i] 在字典中，那么前i个字符也可以被成功拆分
        for i in range(1, n + 1):
            for j in range(i):
                if dp[j] and s[j:i] in st:  # 即：dp[i] = dp[j] and (s[j:i] in st)
                    dp[i] = True
                    break
        
        return dp[n]
```

## [最大子数组和](https://leetcode.cn/problems/maximum-subarray/)(53)


问题：给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。**子数组** 是数组中的一个连续部分。

分析：动态规划，**dp[i] 表示“以 i 结尾的最大子数组和”**。若 dp[i-1] > 0，就把它接到 nums[i] 前面；否则从 nums[i] 重新开始。

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        n = len(nums)
        dp = [0] * (n + 1)  # dp[i] 表示：以第 i 个元素 nums[i-1] 结尾的连续子数组的最大和，第 0 个位置留空
        dp[1] = nums[0]
        for i in range(2, n + 1):
            if dp[i - 1] >= 0:  # 以 i-1 结尾的最大子数组非负，可以扩展到 i
                dp[i] = dp[i - 1] + nums[i - 1]
            else:  # 以 i-1 结尾的最大子数组为负，丢弃
                dp[i] = nums[i - 1]
        return max(dp[1:])  # 取所有位置结尾的最大子数组的最大值
```

## [乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)(152)

问题：找出整数数组 `nums` 数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

分析：之所以要同时维护最大值和最小值，是因为负数会把大小关系翻转：一个很小的负数乘上当前负数，可能变成很大的正数。转移时考虑三种情况：只取当前元素 x；把前一个最大乘积子数组接上 x；把前一个最小乘积子数组接上 x

```python
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        n = len(nums)
        # 为什么同时维护最大值和最小值：因为当前数如果是负数，最小乘积乘上它之后可能变成最大乘积（第 0 个位置留空）
        dp_max = [0] * (n + 1)  # dp_max[i] 表示：以 nums[i - 1] 结尾的连续子数组的最大乘积
        dp_min = [0] * (n + 1)  # dp_min[i] 表示：以 nums[i - 1] 结尾的连续子数组的最小乘积
        dp_max[1] = nums[0]  # 边界：以第 1 个元素 nums[0] 结尾的连续子数组，最大乘积和最小乘积都只能是它自己
        dp_min[1] = nums[0]
        ans = nums[0]

# 状态转移：以nums[i-1]结尾的连续子数组，只有两种来源：(1)只取当前元素 (2)在前一个位置结尾的连续子数组后面接上nums[i-1]
        for i in range(2, n + 1):
            x = nums[i - 1]
            dp_max[i] = max(x, dp_max[i - 1] * x, dp_min[i - 1] * x)
            dp_min[i] = min(x, dp_max[i - 1] * x, dp_min[i - 1] * x)
            ans = max(ans, dp_max[i])
        
        return ans
```

## [最小路径和](https://leetcode.cn/problems/minimum-path-sum/)(64)


问题：给定一个包含非负整数的 `m x n` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。每次只能向下或者向右移动一步。

分析：设 dp[i] [j] 表示从左上角走到第 i 行第 j 列的不同路径条数。只能向下或向右走，到 (i, j) 只能从上方 (i - 1, j) 左方 (i, j - 1) 转移而来

```python
class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        m, n = len(grid), len(grid[0])
        # dp[i][j] 表示从左上角走到第 i 行第 j 列的最小路径和，这里行列下标都从 1 开始，第 0 行和第 0 列自然留空
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        dp[1][1] = grid[0][0]  # 边界：起点位置只能从自己开始，因此路径和就是 grid[0][0]
        for j in range(2, n + 1):  # 第一行只能从左边走过来
            dp[1][j] = dp[1][j - 1] + grid[0][j - 1]
        for i in range(2, m + 1):  # 第一列只能从上边走过来
            dp[i][1] = dp[i - 1][1] + grid[i - 1][0]
        
        # 状态转移：到达 (i, j) 只能从上方 (i-1, j) 或左方 (i, j-1) 过来，取两者较小值，再加上当前格子的代价
        for i in range(2, m + 1):
            for j in range(2, n + 1):
                dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1]
        
        return dp[m][n]
```

## [最大正方形](https://leetcode.cn/problems/maximal-square/)(221)

问题：在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。

分析：**“连续型”问题都要定义“后缀型”dp**，即 dp[i] [j] 表示以 (i,j) 为**右下角**的最大正方形边长，再取最值作为答案。

```python
class Solution:
    def maximalSquare(self, matrix: List[List[str]]) -> int:
        ans = 0
        m, n = len(matrix), len(matrix[0])
        # 第 0 行和第 0 列留空，表明无法以不存在的格子组成正方形，并且统一了后续的转移方程
        dp = [[0] * (n + 1) for _ in range(m + 1)]  # dp[i][j] 表示以 (i,j) 为右下角的最大正方形边长
        
        # 状态转移：取左(i, j-1)、上(i-1, j)、左上(i-1, j-1)三个方向最小值 + 1 作为以当前为右下角的最大边长
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if matrix[i - 1][j - 1] == '1':  # 当前位置是 1 才能组成正方形
                    dp[i][j] = min(dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1]) + 1
                    ans = max(ans, dp[i][j])  # 更新最大边长
        
        return ans ** 2
```

## [目标和:0-1背包](https://leetcode.cn/problems/target-sum/)(494)

问题：给你一个非负整数数组 `nums` 和一个整数 `target` 。向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

分析：记 `s` 为数组和，`p` 为正整数和，则有 `p - (s - p) = t`, `p = (s + t) / 2`，转化为容量恰好为 `capacity = (s + t) / 2`, `weight[i] = nums[i]` ，求方案数的 `0-1` 背包问题

注意：不完全等价于标准的 `0-1` 背包问题，初始化 dp 时有且仅有 dp[0] [0] = 1，因为数组元素可能为 0，故 dp[1…n] [0] 是不确定的，需要在状态转移时计算得出。而标准情况下数组元素 > 0，故 dp[0…n] [0] = 1，即只有不选这一种方案。

```python
""" 二、1:1 翻译成递推 """
class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        for num in nums:  # 计算 s + t
            target += num
        if target < 0 or target % 2:  # s + t 不能是负数或奇数
            return 0
        target //= 2  # 计算容量 (s + t) / 2
        n = len(nums)
        dp = [[0] * (target + 1) for _ in range(n + 1)]  # 事实上，第0列为1（循环中完成），第0行剩余为0
        dp[0][0] = 1  # 当 i == 0 且 j == 0 时，恰有 1 种方案（0个物品凑出0的容量）

        for i in range(1, n + 1):
            for j in range(0, target + 1):  # 必须从 0 开始而不是 1
                if j < nums[i - 1]:
                    dp[i][j] = dp[i - 1][j]
                else:
                    dp[i][j] = dp[i - 1][j] + dp[i - 1][j - nums[i - 1]]

        return dp[n][target]
```

## [零钱兑换-完全背包](https://leetcode.cn/problems/coin-change/)(322)


问题：给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。计算并返回可以凑成总金额所需的 **最少硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。你可以认为每种硬币的数量是无限的。

分析：转化为容量恰好为 `capacity == amount`, `weight[i] = coins[i]`, `value[i] = 1` 求最小价值和的完全背包问题

```python
from math import inf
""" 二、1:1 翻译成递推 """
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        n = len(coins)
        dp = [[0] * (amount + 1) for _ in range(n + 1)]  # 实际意义上，第0列为0，第0行剩余为正无穷
        for j in range(1, amount + 1):  # dp[0][0] = 0，表示没有硬币时，组成金额 0 需要 0 个硬币
            dp[0][j] = inf  # dp[0][j] = inf (c > 0)，表示没有硬币时，无法组成正金额

        for i in range(1, n + 1):
            for j in range(1, amount + 1):
                if j < coins[i - 1]:
                    dp[i][j] = dp[i - 1][j]
                else:
                    dp[i][j] = min(dp[i - 1][j], dp[i][j - coins[i - 1]] + 1)

        ans = dp[n][amount]
        return ans if ans != inf else -1
```

## [分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)(416)


问题：给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

分析：设 nums 的元素和为 s，把 nums 分成两个子集，每个子集的元素和恰好等于 s/2。问题相当于：**能否从 nums 中选出一个子序列，其元素和恰好等于 s/2，这可以用「恰好装满」型 0-1 背包解决**。

```python
class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        s = sum(nums)
        if s % 2:  # 如果 s 是奇数，s / 2 不是整数
            return False
        target = s // 2  # 如果 s 是偶数，则选出一个子序列，其元素和恰好等于 s/2
        n = len(nums)
        dp = [[False] * (target + 1) for _ in range(n + 1)]  # dp[i][j] 表示从前i个数中选，是否能够恰好凑出和j
        for i in range(n + 1):  # 边界：从前 i 个数中选，凑出和 0 一定可以（什么都不选）
            dp[i][0] = True
        
        for i in range(1, n + 1):  # 状态转移：「恰好装满」型 0-1 背包问题
            for j in range(1, target + 1):
                if j < nums[i - 1]:  # 第 i 个数放不下，只能不选
                    dp[i][j] = dp[i - 1][j]
                else:  # 有选或不选第 i 个数两种决策，只要有一种为 True，就说明可以凑出 j
                    dp[i][j] = dp[i - 1][j] or dp[i - 1][j - nums[i - 1]]
        
        return dp[n][target]
```

## [最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)(5)

问题：给你一个字符串 `s`，找到 `s` 中最长的 回文 子串。

分析：设 dp[i] [j] 表示子串 s[i-1:j] 是否为回文串。若两端字符不同，则不是回文串；若两端字符相同，长度 >= 4 时从中间转移而来

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        n = len(s)
        # dp[i][j] 表示子串 s[i-1:j] 是否为回文串，这里 i、j 都从 1 开始，第 0 行和第 0 列自然留空
        dp = [[False] * (n + 1) for _ in range(n + 1)]
        start = 1    # 当前最长回文子串的起始位置（1-based）
        max_len = 1  # 当前最长回文子串的长度
        for i in range(1, n + 1):  # 边界：长度为 1 的子串一定是回文串
            dp[i][i] = True
        
        for length in range(2, n + 1):  # 递推开始，先枚举子串长度
            for i in range(1, n + 1):  # 枚举左边界，左边界的上限设置可以宽松一些
                j = i + length - 1  # 由子串长度和左边界确定右边界
                if j > n:  # 如果右边界越界，就可以退出当前循环
                    break
                if s[i - 1] != s[j - 1]:  # 若两端字符不同，则一定不是回文串
                    dp[i][j] = False
                else:
                    if length <= 3:  # 若长度为 2 或 3，只要两端字符相同就是回文串
                        dp[i][j] = True
                    else:  # 若长度 >= 4，则要看去掉两端后的子串是否为回文串
                        dp[i][j] = dp[i + 1][j - 1]
                
                if dp[i][j] and length > max_len:  # 更新答案
                    max_len = length
                    start = i

        return s[start - 1:start - 1 + max_len]
```

## [最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)(718)

问题：给两个整数数组 `nums1` 和 `nums2` ，返回 *两个数组中 **公共的** 、长度最长的子数组的长度* 。

分析：由于**子数组(subarray)连续**而**子序列(subsequence)不连续**，dp状态要表示“**必须以这两个位置结尾**”的最长公共子数组长度。同时，当枚举的两个字符不同时，必须强制归零，断掉不合法的公共子数组。

```python
class Solution:
    def findLength(self, nums1: List[int], nums2: List[int]) -> int:
        m, n = len(nums1), len(nums2)
        # dp[i][j] 表示：以 nums1[i-1] 和 nums2[j-1] 作为结尾时最长公共子数组的长度，第 0 行和第 0 列留空
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        ans = 0

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if nums1[i - 1] == nums2[j - 1]:  # 这两个元素相等，公共子数组可以向前延伸
                    dp[i][j] = dp[i - 1][j - 1] + 1
                    ans = max(ans, dp[i][j])
                # else:  # 默认初始化就是 0，表示一旦不相等，公共“子数组”必须断掉
                #     dp[i][j] = 0

        return ans
```

## [最长公共子序列-LCS](https://leetcode.cn/problems/longest-common-subsequence/)(1143)


问题：给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

分析：默认**子数组(`subarray`)与子串(`substring`)连续，子序列(`subsequence`)不连续**，**子序列本质就是 选或不选**，考虑最后一对字母 `s[i],t[j]`

 * 当前操作：考虑 `s[i]` 和 `t[j]` 选或不选
 * 子问题：`s[1...i]` 和 `t[1...j]` 的 `LCS` 长度
 * 下一个子问题：
   * 选 `s[i]`：`s[1...i-1]` 和 `t[1...j]` 的 `LCS` 长度
   * 选 `t[j]`：`s[1...i]`  和 `t[1...j-1]` 的 `LCS` 长度
   * 都选/都不选： `s[1...i-1]` 和 `t[1...j-1]` 的 `LCS` 长度
 * 怎么递归：
   * `s[i] == t[j]` 时，递归到 `3.3`（都选），并且 `dp` 值 `+1`
   * `s[i] != t[j]` 时，递归到 `3.1` 与 `3.2`，并取 `max`

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)
        # dp[i][j] 定义为 s[1...i] 和 t[1...j] 的 LCS 长度，第 0 行和第 0 列留空，表示有一个字符串长度为 0
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:  # 这两个字符可以作为公共子序列的最后一个字符
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:  # 这两个字符不能同时选作最后一个字符，只能二选一，选之前使得 LCS 最大的那个
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        
        return dp[m][n]
```

## [最长递增子序列-LIS](https://leetcode.cn/problems/longest-increasing-subsequence/)(300)


问题：给你一个整数数组 `nums` ，找到其中最长**严格递增**子序列的长度。**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

解法1：若对 `nums` 数组排序去重，再与 `nums` 本身求[最长公共子序列](#[最长公共子序列-LCS](https://leetcode.cn/problems/longest-common-subsequence/)(1143)) `LCS`，即得到最长递增子序列 `LIS`。若非严格递增则无需去重

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        """ 最长公共子序列 LCS """
        m, n = len(text1), len(text2)
        # dp[i][j] 定义为 s[1...i] 和 t[1...j] 的 LCS 长度，第 0 行和第 0 列留空，表示有一个字符串长度为 0
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:  # 这两个字符可以作为公共子序列的最后一个字符
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:  # 这两个字符不能同时选作最后一个字符，只能二选一，选之前使得 LCS 最大的那个
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        
        return dp[m][n]
    
    def lengthOfLIS(self, nums: List[int]) -> int:
        """ 最长递增子序列 LIS """
        arr = sorted(set(nums))  # 排序 + 去重，得到严格递增数组
        return self.longestCommonSubsequence(nums, arr)
```

解法2：子序列本质上是数组的一个子集，故采用子集型回溯的方式思考

 * 思路1-选或不选：为了比较大小，还需要知道上一个选的数字的下标
 * 思路2-枚举选哪个：比较当前选的数字和下一个要选的数字，只需要知道当前所选数字的下标（优于思路1）

将原问题如下分解为子问题，最终答案为遍历各子问题并取 `max`

 * 当前操作：枚举 `j < i && nums[j] < nums[i]` 的数字 `nums[j]`
 * 子问题：以 `nums[i]` 结尾的 `LIS` 长度
 * 下一个子问题：以 `nums[j]` 结尾的 `LIS` 长度
 * 怎么递归：枚举所有满足要求的 `j` ，将其求 `max` 再 `+1`，得到当前子问题 `i` 的值

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)
        dp = [0] * (n + 1)  # dp[i] 表示：以 nums[i - 1] 结尾的 LIS 长度，第 0 个位置留空
        for i in range(1, n + 1):  # 边界：任何单个元素本身都可以构成长度为 1 的严格递增子序列
            dp[i] = 1
        
        # 状态转移：枚举最后一个元素是 nums[i - 1]，再枚举它前面的元素 nums[j - 1] 作为倒数第二个元素
        # 如果 nums[j - 1] < nums[i - 1]，那么可以把以 nums[j - 1] 结尾的递增子序列接到 nums[i - 1] 前面
        for i in range(1, n + 1):
            for j in range(1, i):
                if nums[j - 1] < nums[i - 1]:
                    dp[i] = max(dp[i], dp[j] + 1)
        
        return max(dp[1:])
```

## [最长回文子序列-LPS](https://leetcode.cn/problems/longest-palindromic-subsequence/)(516)


问题：给你一个字符串 `s` ，找出其中最长的回文子序列，并返回该序列的长度。子序列定义为：不改变剩余字符顺序的情况下，删除某些字符或者不删除任何字符形成的一个序列。

解法1：求 `s` 和反转后 `t` 的[最长公共子序列](#[最长公共子序列-LCS](https://leetcode.cn/problems/longest-common-subsequence/)(1143)) `LCS`

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        """ 最长公共子序列 LCS """
        m, n = len(text1), len(text2)
        # dp[i][j] 定义为 s[1...i] 和 t[1...j] 的 LCS 长度，第 0 行和第 0 列留空，表示有一个字符串长度为 0
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:  # 这两个字符可以作为公共子序列的最后一个字符
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:  # 这两个字符不能同时选作最后一个字符，只能二选一，选之前使得 LCS 最大的那个
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        
        return dp[m][n]
    
    def longestPalindromeSubseq(self, s: str) -> int:
        """ 最长回文子序列 LPS """
        t = s[::-1]  # 反转字符串
        return self.longestCommonSubsequence(s, t)
```

解法2：**区间 DP**：子序列本质上是选或不选：从两侧向内缩小问题规模

 * 当前操作：考虑 `s[i]` 和 `s[j]` 选或不选
 * 子问题：`s[i...j]` 的 `LPS` 长度
 * 下一个子问题：
   * 选 `s[i]`：`s[i+1...j]` 的 `LPS` 长度
   * 选 `s[j]`：`s[i...j-1]` 的 `LPS` 长度
   * 都选/都不选：`s[i+1...j-1]` 的 `LPS` 长度
 * 怎么递归：
   * `s[i] == s[j]` 时，递归到 `3.3`（都选），并且 `dp` 值 `+2`
   * `s[i] != s[j]` 时，递归到 `3.1` 与 `3.2`，并取 `max`

```python
class Solution:
    def longestPalindromeSubseq(self, s: str) -> int:
        n = len(s)
        dp = [[0] * (n + 1) for _ in range(n + 1)]  # dp[i][j] 表示子串 s[i-1:j] 的最长回文子序列长度

        # 注意循环顺序：dp[i][j] 依赖 dp[i+1][j-1]、dp[i+1][j]、dp[i][j-1]，所以 i 要倒序枚举，j 要正序枚举
        for i in range(n, 0, -1):
            dp[i][i] = 1  # 单个字符的最长回文子序列长度为 1
            for j in range(i + 1, n + 1):
                if s[i - 1] == s[j - 1]:  # 可以同时选 2 个字符加入回文子序列
                    dp[i][j] = dp[i + 1][j - 1] + 2
                else:  # 只能二选一加入子序列，选使得 LPS 最大的那个
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])

        return dp[1][n]
```

## [编辑距离](https://leetcode.cn/problems/edit-distance/)(72)


问题：给你两个单词 `word1` 和 `word2`， *请返回将 `word1` 转换成 `word2` 所使用的最少操作数* 。可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

分析：子序列本质就是 选或不选，考虑最后一对字母 `s[i], t[j]`

 * 当前操作：考虑 `s[i]` 和 `t[j]` 选或不选
 * 子问题：`s[1...i]` 到 `t[1...j]` 的最少操作数
 * 下一个子问题：
   * 删除操作-选 `s[i]`：`s[1...i-1]` 到 `t[1...j]` 的最少操作数
   * 插入操作-选 `t[j]`：`s[1...i]` 到 `t[1...j-1]` 的最少操作数
   * 替换操作-都选：`s[1...i-1]` 到 `t[1...j-1]` 的最少操作数
   * 无操作-都不选：`s[1...i-1]` 到 `t[1...j-1]` 的最少操作数
 * 怎么递归：
   * `s[i] == t[j]` 时，递归到 `3.4`（都不选）
   * `s[i] != t[j]` 时，递归到 `3.1`(删除)，`3.2`(插入)，`3.3`(替换)，并取 `min` 后 `+1`

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)
        # dp[i][j] 表示 word1[1...i] 到 word2[1...j] 的最少操作数，第 0 行和第 0 列留空，表示其中一个字符串长度为 0
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        for i in range(1, m + 1):  # 将 word1 的前 i 个字符转换成空串，只能删除 i 次
            dp[i][0] = i
        for j in range(1, n + 1):  # 将空串转换成 word2 的前 j 个字符，只能插入 j 次
            dp[0][j] = j
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if word1[i - 1] == word2[j - 1]:  # 若当前两个字符相同，则最后一个字符无需操作
                    dp[i][j] = dp[i - 1][j - 1]
                else:  # 若当前两个字符不同，则在插入、删除、替换三种操作里取最小值，操作数+1
                    dp[i][j] = min(
                        dp[i][j - 1],     # 插入
                        dp[i - 1][j],     # 删除
                        dp[i - 1][j - 1]  # 替换
                    ) + 1                 # 操作数+1
        
        return dp[m][n]
```

# 位运算/数学

## [找出数组的最大公约数](https://leetcode.cn/problems/find-greatest-common-divisor-of-array/)(1979)

问题：给你一个整数数组 `nums` ，返回数组中最大数和最小数的 **最大公约数** 。两个数的 **最大公约数** 是能够被两个数整除的最大正整数。

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def findGCD(self, nums: List[int]) -> int:
        def gcd_(a: int, b: int) -> int:
            """ 辗转相除法求最大公约数 gcd """
            while b != 0:
                a, b = b, a % b
            return a

        a, b = max(nums), min(nums)
        return gcd_(a, b)  # Python 也自带 gcd() 函数
```

## [公因子的数目](https://leetcode.cn/problems/number-of-common-factors/)(2427)

问题：返回两个正整数 `a` 和 `b`的 **公** 因子的数目。如果 `x` 可以同时整除 `a` 和 `b` ，则认为 `x` 是 `a` 和 `b` 的一个 **公因子**。

分析：*x* 是 *a* 和 *b* 的公因子，当且仅当 *x* 是 *a* 和 *b* 的**最大公约数**的因子，故枚举到最大公约数 c = gcd(a, b) 即可。继续优化，由于因子一定是成对出现的，如果 *x* 是 *c* 的因子，那么 c / x 一定也是 *c* 的因子，故枚举到 √c 即可，并判断因子是否成对。

复杂度：`O(√n)` `O(1)` 其中求解 *a* 和 *b* 最大公约数需要 `O(logn)` 的时间，但其渐进意义下小于 `O(√n)`，因此可以省略。

```python
class Solution:
    def commonFactors(self, a: int, b: int) -> int:
        c, ans = gcd(a, b), 0
        x = 1
        while x * x <= c:  # 枚举 [1, √c]
            if c % x == 0:  # 若 x 是 c 的因子，则 c / x 也是 c 的因子
                ans += 2
                if x * x == c:  # 当 c 恰好是 x 的平方时，只有一个因子
                    ans -= 1
            x += 1
        return ans
```

## [计数质数](https://leetcode.cn/problems/count-primes/)(204)

问题：给定整数 `n` ，返回 *所有小于非负整数 `n` 的质数的数量* 。

解法 1（超时）：普通枚举，其中判断质数只需枚举 [2, √x]

复杂度：`O(n√n)` `O(1)`

```python
class Solution:
    def is_prime(self, x: int) -> bool:
        """ 判断 x 是否为质数 """
        if x < 2:  # 0 和 1 不是质数
            return False
        i = 2
        while i * i <= x:  # 判断质数只需枚举 [2, √x]
            if x % i == 0:
                return False
            i += 1
        return True

    def countPrimes(self, n: int) -> int:
        ans = 0
        for x in range(2, n):
            if self.is_prime(x):
                ans += 1
        return ans
```

解法 2：**埃氏筛**：如果 i 是质数，那么把 i 的倍数都标记成非质数。最后统计还为 True 的个数即可。优化：从 i * i 开始筛，因为 2i, 3i, ..., (i-1)*i 已经被其他质数筛去

注意：本题要求统计 < n 的质数个数，因此循环条件为 i * i < n 而非 i * i <= n（后续 j 范围同理）

复杂度：`O(nloglogn)` `O(1)`

```python
class Solution:
    def countPrimes(self, n: int) -> int:
        if n < 3:  # 不存在 < n 的质数
            return 0
        is_prime = [True] * n  # is_prime[i] 表示 i 是否是质数
        is_prime[0] = is_prime[1] = False  # 0 和 1 不是质数
        i = 2  # 本题要求统计 < n 的质数个数，因此循环条件为 i * i < n 而非 i * i <= n（后续 j 范围同理）
        while i * i < n:  # 只需枚举 [2, √n]，因为后面从 i * i 开始筛
            if is_prime[i]:  # 如果 i 是质数，那么把 i 的倍数都标记成非质数 False
                for j in range(i * i, n, i):  # 从 i*i 开始筛，因为 2i, 3i, ..., (i-1)*i 已经被其他质数筛去
                    is_prime[j] = False
            i += 1
        return sum(is_prime)
```

## [多数元素](https://leetcode.cn/problems/majority-element/)(169)

问题：给定一个大小为 `n` 的数组 `nums` ，返回其中的多数元素。多数元素是指在数组中出现次数 **大于** `⌊ n/2 ⌋` 的元素。

分析：摩尔投票法（`Moore's Voting Algorithm`），先初始化一个候选元素 `candidate` 和计数器 `count`

* 如果 `x` 等于 `candidate`，则 `count` 增加 `1`；否则 `count` 减少 `1`
* 如果 `count == 0`，则将当前元素设为候选元素 `candidate = nums[i]`，并设置 `count = 1`

复杂度：`O(n)` `O(1)`

```python
class Solution:
    def majorityElement(self, nums: List[int]) -> int:  # 摩尔投票法 (Moore's Voting Algorithm)
        candidate = None  # 候选元素
        count = 0         # 计数器
        for num in nums:
            if count == 0:  # 如果 count 为 0，则将当前元素 num 设为候选元素 candidate，并将 count 设为 1
                candidate = num
                count = 1
            elif num == candidate:  # 如果 num 等于 candidate，则 count 增加 1；否则 count 减少 1。
                count += 1
            else:
                count -= 1
        return candidate
```

## [x 的平方根 ](https://leetcode.cn/problems/sqrtx/)(69)

问题：给你一个非负整数 `x` ，计算并返回 `x` 的 **算术平方根** 。由于返回类型是整数，结果只保留 **整数部分** ，小数部分将被 **舍去 。**

分析：红蓝染色法：对满足 check(mid*mid>x) 的染蓝色，找出第一个满足check的蓝色节点，则答案是最后一个红色节点

复杂度：`O(logx)` `O(1)`

```python
class Solution:
    def mySqrt(self, x: int) -> int:
        """ 红蓝染色法：对满足check(mid*mid>x)的染蓝色，找出第一个满足check的蓝色节点，则答案是最后一个红色节点 """
        if x < 2:  # 0 和 1 直接返回本身
            return x
        left, right = 1, x // 2
        while left <= right:
            mid = left + (right - left) // 2
            if mid * mid <= x:   # mid 不满足条件，答案在右侧
                left = mid + 1   # 尝试更大的值
            else:                # mid 满足条件，答案可能是 mid 或更左侧
                right = mid - 1  # 尝试更小的值
        return right             # right == left - 1，最后一个红色节点满足mid * mid <= x，即答案
```

## [位1的个数](https://leetcode.cn/problems/number-of-1-bits/)(191)

问题：给定一个正整数 `n`，编写一个函数，获取一个正整数的二进制形式并返回其二进制表达式中 设置位 的个数（也被称为[汉明重量](https://baike.baidu.com/item/汉明重量)）

分析：`n & (n - 1)` 将 `n` 二进制最右端的 `1` 转为 `0`，同时统计数量即可

复杂度：`O(logn)` `O(1)`

```python
class Solution:
    def hammingWeight(self, n: int) -> int:
        count = 0
        while n:
            n &= n - 1  # 消去最低位的 1
            count += 1
        return count
```

## 快速幂(x的n次方)

问题：计算 `x` 的 `n` 次方，均为正整数

分析：利用指数 `n` 的二进制表示，每次先 `n & 1 == 1` 取最低位的值，再 `n = n >> 1` 右移 `1` 位直至为 `0`

复杂度：`O(logn)` `O(1)`

```python
def qpow(x: int, n: int) -> int:
    ans = 1
    while n > 0:
        if (n & 1) == 1:  # 取最低位的值，并判断是否为 1
            ans *= x
        x *= x
        n >>= 1           # 右移一位，等价于 n //= 2
    return ans

def qpow_mod(x: int, n: int, mod: int) -> int:
    ans = 1
    while n > 0:
        if (n & 1) == 1:  # 取最低位的值，并判断是否为 1
            ans = ans * x % mod
        x = x * x % mod
        n >>= 1           # 右移一位，等价于 n //= 2
    return ans
```

