---
title: "LeetCode 周赛 264"
date: 2021-10-24T20:31:38+08:00
---

## LeetCode 264场周赛题解

### 2047. [句子中的有效单词数](https://leetcode-cn.com/problems/number-of-valid-words-in-a-sentence/)

**思路** 直接模拟检查，注意这里的'-'符号两边是指该符号的左右两边，检查有点繁琐  

**代码**
```
class Solution:
    def is_valid(self, token: str) -> int:
        for c in token:
            if c >= '0' and c <= '9':
                return 0
            if c >= 'A' and c <= 'Z':
                return 0
        if token[0] == '-' or token[len(token)-1] == '-':
            return 0
        num_her = 0
        num_other = 0
        for c in token:
            if c == '-':
                num_her += 1
            elif c < 'a' or c > 'z':
                num_other += 1
        if num_her > 1:
            return 0
        elif num_her == 1:
            for i in range(len(token)):
                if token[i] == '-':
                    if i > 0:
                        if token[i-1] < 'a' or token[i-1] > 'z':
                            return 0
                    if i < len(token) -1:
                        if token[i+1] < 'a' or token[i+1] > 'z':
                            return 0
                    break
        if num_other > 1:
            return 0
        if num_other:
            if token[-1] < 'a' or token[-1] > 'z':
                return 1
            else:
                return 0
        return 1
        
    def countValidWords(self, sentence: str) -> int:
        ret = 0
        for token in sentence.split(" "):
            if len(token) == 0:
                continue
            if self.is_valid(token):
                ret += 1
        return ret
```

### 2048. [下一个更大的数值平衡数](https://leetcode-cn.com/problems/next-greater-numerically-balanced-number/)

如果整数  x 满足：对于每个数位 d ，这个数位 恰好 在 x 中出现 d 次。那么整数 x 就是一个 数值平衡数 。  
给你一个整数 n ，请你返回 严格大于 n 的 最小数值平衡数 。  

**思路**  

**代码**
```
class Solution: 
    def is_valid(self, n: int) -> int:
        bt = [0 for i in range(10)]
        while n:
            x = int(n % 10)
            bt[x] += 1
            n = int(n / 10)
        idx = 0
        for x in bt:
            if x and x != idx:
                return 0
            idx += 1
        return 1
        
        
    def nextBeautifulNumber(self, n: int) -> int:
        n += 1
        while True:
            if self.is_valid(n):
                return n
            n += 1
```

### 2049. [统计最高分的节点数目](https://leetcode-cn.com/problems/count-nodes-with-the-highest-score/)

**思路**   

**代码**
```
class Node:
    def __init__(self, val):
        self.child = []
        self.val = val
        self.nums = 0
    
    def add_child(self, child):
        self.child.append(child)

class Solution:
    def dfs(self, node: Node):
        tmp = 0
        for c in node.child:
            tmp += self.dfs(c)
        node.nums = tmp + 1
        return node.nums
        
    def countHighestScoreNodes(self, parents: list[int]) -> int:
        root = None
        graph = {}
        for i, parent in enumerate(parents):
            graph[i] = Node(parent)
        for i, p in enumerate(parents):
            if p == -1:
                root = graph[i]
            else:
                parent = graph[p]
                parent.add_child(graph[i])
        self.dfs(root)
        ans = []
        for k in graph:
            node = graph[k]
            if node == root:
                tmp = 1
                for ch in root.child:
                    tmp *= ch.nums
                ans.append(tmp)
            else:
                tmp = 1
                for ch in node.child:
                    tmp *= ch.nums
                ans.append(tmp * (len(parents) - node.nums))
        ans.sort()
        ret = 0
        for x in ans:
            if x == ans[-1]:
                ret += 1
        return ret
```
