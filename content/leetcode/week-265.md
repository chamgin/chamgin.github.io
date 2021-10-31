---
title: "LeetCode 周赛 265"
date: 2021-10-31T14:49:38+08:00
---

## LeetCode 265场周赛题解

### 5914. [值相等的最小索引](https://leetcode-cn.com/problems/smallest-index-with-equal-value/)
给你一个下标从 0 开始的整数数组 nums ，返回 nums 中满足 i mod 10 == nums[i] 的最小下标 i ；如果不存在这样的下标，返回 -1 。  
x mod y 表示 x 除以 y 的 余数 。  

**思路** 直接比较

**代码**
```
class Solution:
    def smallestEqual(self, nums: List[int]) -> int:
        for i, num in enumerate(nums):
            if i % 10 == num:
                return i
        return -1
```

### 5915. [找出临界点之间的最小和最大距离](https://leetcode-cn.com/problems/find-the-minimum-and-maximum-number-of-nodes-between-critical-points/)

**思路**  先找出所有的临界值，无论是最大还是最小，都把坐标放到数组中，最后最大距离就是排序的两端距离，最小距离需要遍历数组

**代码**
```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def nodesBetweenCriticalPoints(self, head: Optional[ListNode]) -> List[int]:
        ans = []
        if head == None:
            return [-1, -1]
        pre_node = head
        head = head.next
        idx = 1
        while head != None:
            if head.next and head.val > pre_node.val and head.val > head.next.val:
                ans.append(idx)
            if head.next and head.val < pre_node.val and head.val < head.next.val:
                ans.append(idx)
            idx += 1
            pre_node = head
            head = head.next
        ans.sort()
        if len(ans) <= 1:
            return [-1, -1]
        mi = 0x7fffffff
        pre = ans[0]
        for x in ans[1:]:
            mi = min(mi, x - pre)
            pre = x
        return [mi, ans[-1]-ans[0]]
```

### 5916. [转化数字的最小运算数](https://leetcode-cn.com/problems/minimum-operations-to-convert-number/)

**思路**  使用bfs直接遍历，因为范围在0~1000中，注意如果范围不在0~1000中则直接判断是否是目标值，否则不需要存储到队列进行遍历

**代码**
```
class Solution:
    def minimumOperations(self, nums: List[int], start: int, goal: int) -> int:
        from collections import deque
        
        que = deque([(0, start)])
        visit = {start,}
        
        while que:
            step, i = que.popleft()
            if i == goal:
                return step
            for j in nums:
                for tmp in [i+j, i-j, i^j]:
                    if tmp == goal:
                        return step + 1
                    if 0 <= tmp <= 1000 and tmp not in visit:
                        visit.add(tmp)
                        que.append((step+1, tmp))
        return -1
```
