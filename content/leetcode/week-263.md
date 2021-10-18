---
title: "LeetCode 周赛 263"
date: 2021-10-18T19:11:38+08:00
---

## LeetCode 263场周赛题解

### 1. 检查句子中的数字是否递增
句子是由若干 token 组成的一个列表，token 间用 单个 空格分隔，句子没有前导或尾随空格。每个 token 要么是一个由数字 0-9 组成的不含前导零的 正整数 ，要么是一个由小写英文字母组成的 单词 。

示例，"a puppy has 2 eyes 4 legs" 是一个由 7 个 token 组成的句子："2" 和 "4" 是数字，其他像 "puppy" 这样的 tokens 属于单词。
给你一个表示句子的字符串 s ，你需要检查 s 中的 全部 数字是否从左到右严格递增（即，除了最后一个数字，s 中的 每个 数字都严格小于它 右侧 的数字）。

如果满足题目要求，返回 true ，否则，返回 false 。

输入：s = "1 box has 3 blue 4 red 6 green and 12 yellow marbles"
输出：true
解释：句子中的数字是：1, 3, 4, 6, 12 。
这些数字是按从左到右严格递增的 1 < 3 < 4 < 6 < 12 。

**思路** 直接取句子中是数字的放入数组，然后判断是否严格递增

**代码**
```
class Solution:
    def areNumbersAscending(self, s: str) -> bool:
        nums = []
        for token in s.split(' '):
            if token.isnumeric():
                nums.append(int(token))
        if len(nums) <= 1:
            return True
        for i in range(len(nums)):
            if i == 0:
                continue
            if nums[i] <= nums[i-1]:
                return False
        return True
```

### 2. 简易银行系统
你的任务是为一个很受欢迎的银行设计一款程序，以自动化执行所有传入的交易（转账，存款和取款）。银行共有 n 个账户，编号从 1 到 n 。每个账号的初始余额存储在一个下标从 0 开始的整数数组 balance 中，其中第 (i + 1) 个账户的初始余额是 balance[i] 。

请你执行所有有效的交易。如果满足下面全部条件，则交易有效 ：

指定的账户数量在 1 和 n 之间，且
取款或者转账需要的钱的总数小于或者等于账户余额。   
实现 Bank 类：

**Bank(long[] balance)** 使用下标从 0 开始的整数数组 balance 初始化该对象。   
**boolean transfer(int account1, int account2, long money)** 从编号为 account1 的账户向编号为 account2 的账户转帐 money 美元。如果交易成功，返回 true ，否则，返回 false 。  
**boolean deposit(int account, long money)** 向编号为 account 的账户存款 money 美元。如果交易成功，返回 true ；否则，返回 false 。  
**boolean withdraw(int account, long money)** 从编号为 account 的账户取款 money 美元。如果交易成功，返回 true ；否则，返回 false 。  

输入：  
["Bank", "withdraw", "transfer", "deposit", "transfer", "withdraw"]  
[[[10, 100, 20, 50, 30]], [3, 10], [5, 1, 20], [5, 20], [3, 4, 15], [10, 50]]  
输出：  
[null, true, true, true, false, false]  

解释：  
```
Bank bank = new Bank([10, 100, 20, 50, 30]);    
bank.withdraw(3, 10);    // 返回 true ，账户 3 的余额是 $20 ，所以可以取款 $10 。  
                         // 账户 3 余额为 $20 - $10 = $10 。  
bank.transfer(5, 1, 20); // 返回 true ，账户 5 的余额是 $30 ，所以可以转账 $20 。  
                         // 账户 5 的余额为 $30 - $20 = $10 ，账户 1 的余额为 $10 + $20 = $30 。  
bank.deposit(5, 20);     // 返回 true ，可以向账户 5 存款 $20 。  
                         // 账户 5 的余额为 $10 + $20 = $30 。  
bank.transfer(3, 4, 15); // 返回 false ，账户 3 的当前余额是 $10 。  
                         // 所以无法转账 $15 。  
bank.withdraw(10, 50);   // 返回 false ，交易无效，因为账户 10 并不存在。  
```

**思路** 直接模拟交易，不需要考虑并发安全
**代码**
```
class Bank:

    def __init__(self, balance: List[int]):
        self._balances = balance
        self._nums = len(balance)

    def check_valid_account(self, account: int) -> bool:
        if account < 1 or account > self._nums:
            return False
        return True

    def transfer(self, account1: int, account2: int, money: int) -> bool:
        if self.check_valid_account(account1) == False:
            return False
        if self.check_valid_account(account2) == False:
            return False
        if self._balances[account1-1] < money:
            return False
        self._balances[account1-1] -= money
        self._balances[account2-1] += money
        return True


    def deposit(self, account: int, money: int) -> bool:
        if self.check_valid_account(account) == False:
            return False
        self._balances[account-1] += money
        return True


    def withdraw(self, account: int, money: int) -> bool:
        if self.check_valid_account(account) == False:
            return False
        if self._balances[account-1] < money:
            return False
        self._balances[account-1] -= money
        return True



# Your Bank object will be instantiated and called as such:
# obj = Bank(balance)
# param_1 = obj.transfer(account1,account2,money)
# param_2 = obj.deposit(account,money)
# param_3 = obj.withdraw(account,money)
```

### 3. 统计按位或能得到最大值的子集数目

给你一个整数数组 nums ，请你找出 nums 子集 按位或 可能得到的 最大值 ，并返回按位或能得到最大值的 不同非空子集的数目 。

如果数组 a 可以由数组 b 删除一些元素（或不删除）得到，则认为数组 a 是数组 b 的一个 子集 。如果选中的元素下标位置不一样，则认为两个子集 不同 。

对数组 a 执行 按位或 ，结果等于 a[0] OR a[1] OR ... OR a[a.length - 1]（下标从 0 开始）。

输入：nums = [3,1]
输出：2
解释：子集按位或能得到的最大值是 3 。有 2 个子集按位或可以得到 3 ：
- [3]
- [3,1]

**思路** 按位取出是1的位，因为最大值肯定是所有包含1的位，因此可以先统计出最大值；然后遍历各个集合，看集合按位或结果是否为最大值
**代码**
```
class Solution:
	def countMaxOrSubsets(self, nums: List[int]) -> int:
		bits = [0 for i in range(20)]
		for x in nums:
			i = 0
			while x != 0:
				if x % 2 == 1:
					bits[i] += 1
				i += 1
				x = int(x / 2)

		ma = 0
		idx = 0
		for i in bits:
			if i != 0:
				ma |= (1<<idx)
			idx += 1

		import itertools
		ans = 0

		for c in range(2**len(nums)):
			tmp = 0
			idx = 0
			while c != 0:
				if c & 1:
					tmp |= nums[idx]
				idx += 1
				c >>= 1
			if tmp == ma:
				ans += 1
		return ans
```
