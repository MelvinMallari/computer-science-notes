# Algorithms and Data Structure Notes

## 309 Best Time to Buy and Sell Stock with Cooldown

This can be solved with dynamic programming. The three different actions (buy, sell, rest) take us between 3 different states: s0, s1, s2.

**States**
s0 (rest -> s0) buy-> s1 (rest-> s1) sell -> s2 rest-> s0

s0: can either rest or buy. if rest, stay at s0. if buy go to s1
s1: can either rest or sell. if rest, stay at s1. if buy go to s2
s2: just game from s1, so just sold. have to rest, go to s0

**Initial Conditions**
* `s2[0] = s0[0] = 0` start off at 0  if you haven't bought
* `s1[0] = 0` start off at -prices[0] if you just bought

**Edge Cases**
* if `len(prices) < 2` return 0

Solution 1:
`T: o(n), S: o(n)`
```python
class Solution:
  def maxProfit(self, prices):
    if len(prices) < 2: return 0
    s0 = [0]*len(prices)
    s1 = [0]*len(prices)
    s2 = [0]*len(prices)

    s1[0] = -prices[0]
    for i in range(1, len(prices)):
      s0[i] = max(s0[i-1], s2[i-1])
      s1[i] = max(s1[i-1], s0[i-1] - prices[i])
      s2[i] = s1[i-1] + prices[i]
    return max(s0[-1], s2[-1])
```
Solution 1:
`T: o(n), S: o(1)`
* Notice that that all the states only depend on their previous value. We can reduce our memory consumption by using variables instead.
```python
class Solution:
  def maxProfit(self, prices):
    if len(prices) < 2: return 0
    s0, s1, s2 = 0, -prices[0], 0
    for price in (prices[1:]):
      s0, s1, s2 = max(s0, s2), max(s1, s0 - price), s1 + price
    return max(s0, s2)
```

## 128 Longest Consecutive Sequence
* The key idea is to use a set to reference each number in O(1). 
* Check that the number before the current number is not in the set. This is the start of the streak.
* Keep walking the streak while we keep finding the consecutive number in the set

Solution:
`T: o(n), S: o(n)`
```python
class Solution:
  def longestConsecutive(self, nums):
    numSet = set(nums)
    res = 0
    for n in nums:
      if n-1 not in numSet:
        m = n
        while m in numSet:
          m = m + 1
          res = max(res, m-n)

    return res
```

## 45 Jump Game II
* Greedy Algorithm
* At each number, evaluate the farthest jump that you could have taken
* If you exhaust your current jump, increase the number of jumps you need to take, set your current jump to the farthest jump you could have taken
* This minimizes the amount of jumps you need to take while maximizing the jump distance

**Edge Case**
* (1) If `len(nums) < 2` then no jumps need to be taken, return 0
* (2) We only consider the nums prior to the last number. Must consider the case where you reach the last number at the end of a jump.

Solution:
`T: o(n), S: o(1)`
```python
class Solution:
  def jump(self, nums):
    if len(nums) < 2: return 0 # (1)
    res = currJump = maxJump = 0
    for i, n in enumerate(nums[:-1]): # (2)
      maxJump = max(maxJump, i + n)
      if currJump == i:
        res += 1
        currJump = maxJump
    
    return res
```

## 32 Longest Valid Parentheses
* Use a stack
* Loop through string, if the parenthese is an opener, append index to the stack
* if closer, pop from the stack
  * If the last one was an opener, then we just matched a valid pair
  * If the stack is empty, append the invalid closer so we can reference it as a left boundary
  * If the stack is not empty, then we update the max valid we've seen so far
    * Key idea is any difference b/w curr index and last invalid index stored in stack represents a valid parentheses chunk. Record the longest chunk seen.

**Edge Case**
* (1) Consider the case where the whole parentheses is valid. Must instantiate the stack with -1 to account for the length of whole string

Solution:
`T: o(n), S: o(n)`
```python
class Solution:
  def longestValidParentheses(self, s):
    res, stack = 0, [-1] #(1)

    for i, ch in enumerate(s):
      if ch == '(':
        stack.append(i)
      else:
        stack.pop()
        if not stack:
          stack.append(ch)
        else:
          res = max(res, i - stack[-1])
    return res
```

## 279 Perfect Squares
* Key pattern is dynamic programming
* Create a dp table across possible squares and values up to n
* Base case is 0 
* `T: o(nm), S: o(nm)` solution exist, but was getting TLE so just optimal solution show 

Solution:
`T: o(nm), S: o(n)`
`n: input n`
`m: num perfect squares that fit in n`
```python
class Solution:
  def numSquares(self, n):
    squares = [i*i for i in range(1, math.floor(n**0.5)+1)]
    dp = [float('inf') for _ in range(n+1)]
    dp[0] = 0
    for sq in squares:
      for i in range(1, n+1):
        if i >= sq:
          dp[i] = min(dp[i], dp[i-sq]+1)
  return dp[-1]
```