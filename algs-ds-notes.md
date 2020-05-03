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

## 198 Houser Robber
* Key is to use dynamic programming:
* Base Case is 0
* Memory optimization available when you realize current value only dependent on the previous value

Solution 1:
`T: o(n), S: o(n)`
```python
class Solution:
  def rob(self, nums):
    if not nums: return 0
    dp = [float('-inf') for _ in range(len(nums)+1)]
    dp[0], dp[1] = 0, nums[0]
    for i in range(1, len(nums)):
      dp[i+1] = max(nums[i] + dp[i-1], dp[i])
    return dp[-1]
```

Solution 2:
`T: o(n), S: o(1)`
```python
class Solution:
  def rob(self, nums):
  last = now = 0
  for n in nums:
    now, last = max(last + n, now), now
  return now
```

## 102 Binary Tree Level Order Traversal
* Key idea is to go through level by level building a new queue out of the previous queue
Solution:
`T: o(n), S: o(n)`
```python
class Solution:
  def levelOrder(self, root):
    if not root: return []
    res, q = [], [root]
    while q:
      res.append([n.val for n in q])
      leaves = []
      for n in q:
        if n.left: leaves.append(n.left)
        if n.right: leaves.append(n.right)
      q = leaves
    return res
```
## 337 House Robber III
* This is a great problem. Let's walk through the solutions from naive -> optimal
* In the naive solution, we understand that if we choose to rob the root, then we must skip the kid nodes `root.left`, `root.right`, and rob the grandchildren nodes `root.left.left`, `root.left.right`, `root.right.left` and `root.right.right`. We consider both and choose the highest value operation.
* In the more optimal solution, we realize that in the naive solution there are overlaps in the operations we consider.
  * e.g. robHelper(root) considers robHelper(root.left.left), but when we call robHelper(root.left.left), it calls robHelper(root.left.left) again! We can use dynamic programming to remember the results of shared operatins
* In the most optimal solution as we backtrack back up through the tree in our recursive calls, we return the the value of the options to either rob the root or not.
  * return `res[0]` returns the maximum we can rob if we DON'T rob the root (skipRoot)
  * return `res[1]` returns the maximum we can rob if we rob the root (robRoot)

Solution 0 (Naive):
`T: o(n!), S: o(n)`
```python
class Solution:
  def rob(self, root):
    if not root: return 0
    val = 0
    if root.left:
      val += self.rob(root.left.left) + self.rob(root.left.right)
    if root.right:
      val += self.rob(root.right.left) + self.rob(root.right.right)

    return max(root.val + val, self.rob(root.left) + self.rob(root.right))
```

Solution 1 (More Optimal):
`T: o(n), S: o(n)`
```python
class Solution:
  def rob(self, root):
    return self.robHelper(root, {})

  def robHelper(self, root, seen):
    if not root: return 0
    if root in seen: return seen[root]
    val = 0 
    if root.left:
      val += self.rob(root.left.left, seen) + self.rob(root.left.right, seen)
    if root.right:
      val += self.rob(root.right.left, seen) + self.rob(root.right.right, seen)
    res = max(root.val + val, self.robHelper(root.left, seen) + self.robHelper(root.right, seen))
    seen[root] = res
    return res
```

Solution 2 (Most Optimal):
`T: o(n), S: o(1) (if you don't consider recursive stack calls)`
```python
class Solution:
  def rob(self, root):
    return max(self.robHelper(root))

  def robHelper(self, root):
    if not root: return [0, 0]
    left = self.robHelper(root.left)
    right = self.robHelper(root.right)
    skipRoot = max(left[0], left[1]) + max(right[0], right[1])
    robRoot = left[0] + right[0] + root.val
    return (skipRoot, robRoot)
```

## 581 Shorted Unsorted Continuous Subarray
* Key idea is that you loop through the array searching for the lefmost and rightmost numbers that break the sorted invariant
* Sorted Invariant: any number should be >= than the largest number to its left. any number should be =< than the smallest number to its right. 

**Edge Case**
* in the case that the array is sorted, then the initial values of `end`, `start` must bound a difference of 0.
  * Therefore `start, end = -1, -2`

Solution 1 (more readable):
`T: o(n), S: o(1)`
```python
class Solution:
  def findUnsortedSubarray(self, nums):
    smallest, largest = float('inf'), float('-inf')
    start, end = -1, -2
    for i in range(len(nums)):
      largest = max(largest, nums[i])
      if nums[i] < largest: end = i
    for i in range(len(nums)-1, -1, -1):
      smallest = min(smallest, nums[i])
      if nums[i] > smallest: start = i
    return end - start + 1
```

Solution 2 (optimal):
`T: o(n), S: o(1)`
```python
class Solution:
  def findUnsortedSubarray(self, nums):
    smallest, largest = float('inf'), float('-inf')
    start, end = -1, -2
    n = len(nums)
    for i in range(len(nums)):
      largest = max(largest, nums[i])
      smallest = min(smallest, nums[n-i-1])
      if nums[i] < largest: end = i
      if nums[n-i-1] > smallest : start = n-i-1
    return end - start + 1
```

### 494 Target Sum
* Use dynamic programming
* Key is to loop through numbers and sum the different ways we can make up the number if taken as a positive, or negative
  * if we take num x as negative: `step[y-x] += count[y]`
  * if we take num x as positive: `step[y+x] += count[y]`

**Base Case**
the base case is, with no numbers, we can make 0 one way (1)

`T: o(n^2), S: o(n^2), n:len(nums)`
```python
class Solution:
  def findTargetSumWays(self, nums, S):
    count = collections.defaultdict(int)
    count[0] = 1 # (1)
    for x in nums:
      step = collections.defaultdict(int)
      for y in count:
        step[y+x] += count[y]
        step[y-x] += count[y]
      count = step
    return count[S]
```

### 142 Linked List Cycle II
* use two pointers, a fast and a slow pointer
* faster pointer skips two nodes, slow pointer skips one.
* fast pointer either hits the end of the list, in which case there is no cycle, or fast == slow
* if fast == slow, we've hit the halfway point of the cycle
* restart the fast pointer to the start of the list, and have the two pointers move at the same pace to find the intersection

Solution
`T: o(n), S: o(1)`
```python
class Solution:
  def detectCycle(self, head):
    fast = slow = head
    while fast:
      fast, slow = fast.next.next, slow.next
      if fast == slow: break
    if not fast or not fast.next: return None # hit end of list w no cycle
    first, second = head, slow
    while first != second:
      first, second = first.next, second.next
    return first # or second doesn't matter
```