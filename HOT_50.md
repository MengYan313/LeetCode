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

# 双指针

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
        nums.sort()  # 顺序不重要，那么就规定一个顺序 i < j < k
        res = []
        n = len(nums)

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

分析：滑动窗口：**依次递增地枚举子串的起始位置，那么子串的结束位置也是递增的**

* 利用 `set` 检查重复字符，枚举 `left`，同时每次枚举将 `left - 1` 位置的字符移出 `set`

* 每次枚举 `left`，若 `set` 不重复则不断移动 `right`，并将这些不重复元素加入 `set`，记出现过的最大长度为 `ans`

复杂度：`O(n)` `O(n)`

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        st = set()  # 集合，记录窗口内出现过的字符
        right = -1  # 右指针，初始在窗口左边界左侧
        ans = 0
        n = len(s)
        for left in range(n):
            if left:  # 左指针右移一格，移除上一个字符
                st.remove(s[left - 1])
            while right + 1 < n and s[right + 1] not in st:  # 不断地移动右指针
                right += 1
                st.add(s[right])
                ans = max(ans, right - left + 1)  # 更新最大长度
        return ans
```

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
    def reverseList(self, head: Optional["ListNode"]) -> Optional["ListNode"]:
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
    def reverseList(self, head: Optional["ListNode"]) -> Optional["ListNode"]:
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
    def detectCycle(self, head: Optional["ListNode"]) -> Optional["ListNode"]:
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
    def addTwoNumbers(self, l1: Optional["ListNode"], l2: Optional["ListNode"]) -> Optional["ListNode"]:
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

            if l1:
                l1 = l1.next
            if l2:
                l2 = l2.next

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
    def mergeTwoLists(self, list1: Optional["ListNode"], list2: Optional["ListNode"]) -> Optional["ListNode"]:
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
    def mergeKLists(self, lists: List[Optional["ListNode"]]) -> Optional["ListNode"]:
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
    def removeNthFromEnd(self, head: Optional["ListNode"], n: int) -> Optional["ListNode"]:
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
    def reverseKGroup(self, head: Optional["ListNode"], k: int) -> Optional["ListNode"]:
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

    def mergeTwoLists(self, list1: Optional["ListNode"], list2: Optional["ListNode"]) -> Optional["ListNode"]:
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

    def sortList(self, head: Optional["ListNode"]) -> Optional["ListNode"]:
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
    def maxPathSum(self, root: Optional["TreeNode"]) -> int:
        ans = -inf  # 全局最大路径和（路径可不经过根）# float("-inf")

        # 返回“从 node 出发、向下走（只能选左或右其中一边）”所能获得的最大贡献值，用于给 node 的父节点拼接路径
        def gain(node: Optional["TreeNode"]) -> int:
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
    def levelOrder(self, root: Optional["TreeNode"]) -> List[List[int]]:
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
    def kthSmallest(self, root: Optional["TreeNode"], k: int) -> int:
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
        if root is None or root is p or root is q:  # 边界条件：空节点，或当前节点就是 p / q
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
        slow = fast = 0  # Floyd 判圈：把 nums 看成 next 指针：i -> nums[i]
        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]
            if slow == fast:
                break

        p = 0  # 找环入口（重复数）
        while p != slow:
            p = nums[p]
            slow = nums[slow]
        return p  # 最后得到环入口的下标，根据我们的建图方式，恰好等于重复数的值 target
```

# 回溯

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
            if ch in pairs:  # 左括号入栈
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
            if c == '(':  # 直接入栈
                stack.append(i)
            elif c == ')':  # 无论是否匹配都要弹栈
                stack.pop()
                if not stack:  # 栈为空说明当前 ')' 无法匹配，成为新的边界
                    stack.append(i)
                else:  # 栈不为空说明匹配成功，可以更新当前有效串长度
                    ans = max(ans, i - stack[-1])
        return ans
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
        count = Counter(nums)          # 统计每个元素出现次数
        heap = []                     # 小顶堆：(count, num)
        for num, cnt in count.items():
            if len(heap) < k:         # 堆不足 k 时直接入堆
                heapq.heappush(heap, (cnt, num))
            elif cnt > heap[0][0]:    # 频次更大则替换堆顶（最小频次）
                heapq.heappop(heap)
                heapq.heappush(heap, (cnt, num))
        return [num for _, num in heap]
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

# 动态规划

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

## [零钱兑换-完全背包](https://leetcode.cn/problems/coin-change/)(322)


问题：给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。计算并返回可以凑成总金额所需的 **最少硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。你可以认为每种硬币的数量是无限的。

分析：转化为容量恰好为 `capacity == amount`, `weight[i] = coins[i]`, `value[i] = 1` 求最小价值和的完全背包问题

```python
""" 一、递归搜索 + 保存计算结果 = 记忆化搜索 """
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        n = len(coins)
        cache = [[-1] * (amount + 1) for _ in range(n + 1)]  # -1 表示这个状态还没有计算过

        def dfs(i: int, c: int) -> int:
            """ dfs(i, c) 表示：从前 i 种硬币中选，恰好组成金额 c 所需的最少硬币数 """
            if i < 1:  # 没有硬币可选时：c == 0，说明正好凑出，最少需要 0 个硬币；c > 0，说明无法凑出，记为正无穷
                return 0 if c == 0 else inf
            if cache[i][c] != -1:
                return cache[i][c]
            if c < coins[i - 1]:  # 当前金额 c 放不下第 i 种硬币，只能不选
                cache[i][c] = dfs(i - 1, c)
            else:  # 当前容量可以装下物品 i，选或不选。注意这里仍是 dfs(i, ...)，因为完全背包可以重复选
                cache[i][c] = min(dfs(i - 1, c), dfs(i, c - coins[i - 1]) + 1)
            return cache[i][c]

        ans = dfs(n, amount)
        return ans if ans != inf else -1
```

```python
""" 二、1:1 翻译成递推 """
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        n = len(coins)
        dp = [[0] * (amount + 1) for _ in range(n + 1)]  # 实际意义上，第0列为0，第0行剩余为正无穷
        for c in range(1, amount + 1):  # dp[0][0] = 0，表示没有硬币时，组成金额 0 需要 0 个硬币
            dp[0][c] = inf  # dp[0][c] = inf (c > 0)，表示没有硬币时，无法组成正金额

        for i in range(1, n + 1):
            for c in range(0, amount + 1):
                if c < coins[i - 1]:
                    dp[i][c] = dp[i - 1][c]
                else:
                    dp[i][c] = min(dp[i - 1][c], dp[i][c - coins[i - 1]] + 1)

        ans = dp[n][amount]
        return ans if ans != inf else -1
```

```python
""" 三、空间优化：两个数组（滚动数组） """
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        n = len(coins)
        dp = [[0] * (amount + 1) for _ in range(2)]  # 实际意义上，第0列为0，第0行剩余为正无穷
        for c in range(1, amount + 1):  # dp[0][0] = 0，表示没有硬币时，组成金额 0 需要 0 个硬币
            dp[0][c] = inf  # dp[0][c] = inf (c > 0)，表示没有硬币时，无法组成正金额

        for i in range(1, n + 1):
            for c in range(0, amount + 1):
                if c < coins[i - 1]:
                    dp[i % 2][c] = dp[(i - 1) % 2][c]
                else:
                    dp[i % 2][c] = min(dp[(i - 1) % 2][c], dp[i % 2][c - coins[i - 1]] + 1)

        ans = dp[n % 2][amount]
        return ans if ans != inf else -1
```

```python
""" 四、空间优化：一个数组 """
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

