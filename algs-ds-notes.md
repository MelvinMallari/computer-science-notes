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

## 494 Target Sum
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

## 142 Linked List Cycle II
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

## 72 Edit Distance
* dynamic programming
* key idea is that delete, insert and replace represent three different recurisve operations that we can check
  * We can cache or tabularize the returns of these three different recursive calls

Solution 1 (Naive, top-down solution):
Time complexity is hard to analyze, but exponential
```python
class Solution:
  def minDistance(self, word1, word2):
    if not word1 and not word2: return 0
    if not word1: return len(word2)
    if not word2: return len(word1)
    if word1[0] == word2[0]: return self.minDistance(word1[1:], word2[1:])
    insert = 1 + self.minDistance(word1, word2[1:])
    delete = 1 + self.minDistance(word1[1:], word2)
    replace = 1 + self.minDistance(word1[1:], word2[1:])
    return min(delete, insert, replace)
```

Solution 2 (Optimal, top-down solution):
`T: o(mn), S: o(mn), m: len(w1), n: len(w2)`
```python
class Solution:
  def minDistance(self, word1, word2, i=0, j=0, memo={}):
    if i == len(word1) and j == len(word2): return 0
    if i == len(word1): return len(word2) - j
    if j == len(word2): return len(word1) - i
    if (i, j) not in memo:
      if word1[i] == word2[j]:
        res = self.minDistance(word1, word2, i+1, j+1, memo)
      else:
        insert = 1 + self.minDistance(word1, word2, i, j+1, memo)
        delete = 1 + self.minDistance(word1, word2, i+1, j, memo)
        replace = 1 + self.minDistance(word1, word2, i+1, j+1, memo)
        res = min(insert, delete, replace)
      memo[(i, j)] = res
    return memo[(i, j)]
```

Solution 3 (Optimal, bottom-up solution):
**Base Case**
* Base case is if word to transform to is empty, then it would take length of word to edit into empty string

* from the perpective for word1 (which needs to be transformed to word2)
  * insert: represented by `1 + dp[i-1][j]`
  * delete: represented by `1 + dp[i][j-1]`
  * replace: represented by `1 + dp[i-1][j-1]`
  * match: represented by `dp[i-1][j-1]`

`T: o(mn), S: o(mn), m: len(w1), n: len(w2)`
```python
class Solution:
  def minDistance(self, word1, word2):
    m, n = len(word1), len(word2)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1):
      dp[i][0] = i
    for j in range(n+1):
      dp[0][j] = j
    for i in range(1, m+1):
      for j in range(1, n+1):
        if word1[i-1] == word2[j-1]:
          dp[i][j] = dp[i-1][j-1]
        else:
          dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[-1][-1]
```

## 338 Counting Bits
* dynamic programming
* Key thing to realize is that you want to fit the largest squares of 2's you can, cache the result for future reference
* we take offset and continually check if it'll finally fit into the current number we're at
* otherwise i can be made by the num of 1's it takes from the i-offset + 1

**Base Case**
* n = 0 requires 0 1's to represent

Solution:
`T: o(n), S: o(1), n: num`
```python
class Solution:
  def countBits(self, num):
    res, offset = [0], 1
    for i in range(1, num+1):
      if offset*2 == i: offset *= 2
      res.append(res[i-offset]+1)
    return res
```

## 14 Longest Common Prefix
* first, find the shortest word
* then compare each to every ch in the shortest word
* if there is a mismatch, return the matches up til the mismatches
* if not mismatches return the whole word
Solution:
`T: o(mn), S: o(m) m: length of shortest word, n: `
```python
class Solution:
  def longestCommonPrefix(self, strs):
    if not strs: return ""
    shortest = min(strs, key=len)
    for i, ch in enumerate(shortest):
      for s in strs:
        if s[i] != ch:
          return shortest[:i]
    return shortest
```

## 26 Remove Duplicates from Array
* Key algorithm is often involved with skipping duplicates in an array
  * `if i < len(nums) - 1 and nums[i] == nums[i+1]: continue`
  * `if i > 0 and nums[i] == nums[i-1]: continue`

Solution:
`T: o(n), s: o(1), n: len(nums)`
```python
class Solution:
  def removeDuplicates(self, nums):
    i = 0
    for j in range(len(nums)):
      if j < len(nums)-1 and nums[j] == nums[j+1]: continue
      nums[i] = nums[j]
      i += 1
    return i
```

## 88 Merge Sorted Array
* Variation of merge sort merging algorithm


Solution:
`T: o(n), s: o(1), n: len(nums)`
```python
class Solution:
  def merge(self, nums1, m, nums2,n):
    i, m, n = len(nums1)-1, m-1, n-1
    while m >= 0 and n >= 0:
      if nums1[m] > nums2[n]:
        nums1[i] = nums1[m]
        m -= 1
      else:
        nums1[i] = nums2[n]
        n -= 1
      i -= 1
    
    while m >= 0:
      nums1[i] = nums1[m]
      m -=1
      i -= 1
    
    while n >= 0:
      nums1[i] = nums2[n]
      n -= 1
      i -= 1
```

## 7 Reverse Integer
* Key idea is to modulo the number by 10 to get the 1's number
* add the modulo to the result
* multiply the result by 10 to shift the number's up to make place for the one's digit
* only shift the numbers up if there's more to modulo
* account for negative numbers by having a `sign` variable that's negative if the initial num negative


Solution
`T: o(n), S: o(1), n: num digits in number`
```python
class Solution:
  def reverse(self, x):
    sign = 1 if x > 0 else -1
    res, x = 0, abs(x)
    while x > 0:
      res += x % 10
      if x >= 10: res *= 10
      x //= 10
    return res*sign if -2**31 < res*sign < 2**31+1 else 0
```

## 36 Valid Sudoku
* Need to check that all rows and all cols and all 3x3 squares are valid
* we'll define "unit" as a row or col or square.
* a unit is valid when ther are no repeats in each unit


Solution:
`T:o(n) S:o(1), n: num elements in sudoku`
```python
class Solution:
  def isValidSudoku(self, board):
    return self.rowsValid(board) and self.colsValid(board) and self.squaresValid(board)

  def unitValid(self, unit):
    unit = [i for i in unit if i != '.']
    return len(set(unit)) == len(unit)

  def rowsValid(self, board):
    for row in board:
      if not self.unitValid(row): return False
    return True

  def colsValid(self, board):
    for cols in zip(*board):
      if not self.unitValid(cols): return False
    return True
  
  def squaresValid(self, board):
    for i in (0, 3, 6):
      for j in (0, 3, 6):
        square = [board[x][y] for x in range(i, i+3) for y in range(j, j+3)]
        if not self.unitValid(square): return False
    return True
```

## 44 Wildcard Matching
* dynamic programming
* Two key ideas for the state equation:
  * if `dp[i][j] == '.' or p[j-1] == s[i-1]` both represent a match
    * in this case `dp[i][j] = dp[i-1][j-1]`
  * if `p[j-1] == '*'` there are two possible cases that could be true
    * the word up til the wildcard operator was already a match. '*' doesn't not include the current character
      * this cases is represented by `dp[i][j-1]`
    * the current letter is represented by a previously called wild card operator. '*' does include the current character
      * this case is represented by `dp[i-1][j]`

`T: o(mn), S:o(mn). m: len(p) n:len(s)`
```python
class Solution:
  def isMatch(self, s, p):
    dp = [[False]*(len(p)+1) for _ in range(len(s)+1)]
    dp[0][0] = True
    for j in range(1, len(p)+1):
      if p[j-1] == '*': dp[0][j] = dp[0][j-1]
    for i in range(1, len(s)+1):
      for j in range(1, len(p)+1):
        if p[j-1] == '?' or s[i-1] == p[j-1]:
          dp[i][j] = dp[i-1][j-1]
        elif p[j-1] == '*':
          dp[i][j] = dp[i-1][j] or dp[i][j-1]
    return dp[-1][-1]
```

## 312 Burst Balloons
* dynamic programming
* we want to loop through all possible gap sizes from (2 -> n)
  * reason we pick 2 as a starting point is that's the min size of a non-inclusive borders 
  * e.g. 0, 1, 2 -> (0, 2 bound 1)
* as we loop through the gap sizes we cache the optimal solution for that particular gap and slice of array
* state equation:
  ```python
    dp[l][r] = max(dp[l][r], dp[l][i] + nums[l]*nums[i]*nums[r] + dp[i][r])
  ```
  * we do `nums[l]*nums[i]*nums[r]` because we've cached the optimal solution from l -> i and i -> r
  * in this case we've popped the balloons in the middle and now want to cache the result from l -> r


Solution 1 (Top Down):
`T: o(n^3), S: o(n^3)`
```python
class Solution:
  def maxCoins(self, nums):
    nums, memo = [1] + nums + [1], {}
    def dp(l, r):
      if l + 1 == r: return 0
      if (l, r) not in memo:
        memo[(l, r)] = max(dp(l, i) + nums[l]*nums[i]*nums[r] + dp(i, r) for i in range(l+1, r))
      return memo[(l, r)]
    return dp(0, len(nums)-1)
```

Solution 2 (Bottom Up):
```python
class Solution:
  def maxCoins(self, nums):
    nums, n = [1] + nums + [1], len(nums) + 2
    dp = [[0]*n for _ in range(n)]
    for gap in range(2, n):
      for l in range(0, n-gap):
        r = l + gap
        for i in range(l+1, r):
          dp[l][r] = max(dp[l][r], dp[l][i] + nums[l]*nums[i]*nums[r] + dp[i][r])
    return dp[0][n-1]
```
## 28 Implement strStr()
* loop through and check for substrs to see if they match the needle

**edge case**
* if the needle matches the last n letters in the haystack, this must be accounted for in the loop
  * off by 1 error

Solution:
`T:o(n) S:o(1)`
```python
class Solution:
  def strStr(self, haystack, needle):
    if not needle: return 0
    m, n = len(haystack), len(needle)
    for i in range(m-n+1):
      if haystack[i:i+n] == needle: return i
    return -1
```

## 13 Roman to Integer
* Idea is to loop through the array backwards, if we run into a value that's less than previous we subtract rather than add
Solution:
`T:o(n) S:o(1)`
```python
class Solution:
  def romanToInt(self, s):
    mapping = {'I':1,'V':5,'X':10,'L':50,'C':100,'D':500,'M':1000} 
    res = prev = 0
    for ch in s[::-1]:
      curr = mapping[ch]
      if curr >= prev:
        res += curr
      else:
        res -= curr
      prev = curr
    return res

```

## 91 Decode Ways
* dynamic programming
* state equations:
  * `if 0 < int(s[i-1:i]): dp[i] += dp[i-1]` represents a match of number within [1, 9]
  * `if 10 <= int(s[i-2:i]) <= 26: tmpCurr += prev` represents a match of number within [10, 16]

**Base Case**
* `dp[0] = 1` case where there is a single match gives one possibility

**Edge Case**
* when `s == '' or s[0] == 0`  then there are no possible matches

Solution 1 (non-optimal space):
`T: o(n) S: o(n) n: len(nums)`
```python
class Solution:
  def numDecodings(self, s):
    if not s or s[0] == '0': return 0
    dp = [0]*(len(s)+1)
    dp[0:2] = [1,1]
    for i in range(2, len(s)+1):
      if 0 < int(s[i-1:i]): dp[i] += dp[i-1]
      if 10 <= int(s[i-2:i]) <= 26: dp[i] += dp[i-2]
    return dp[-1]
```

Solution 2 (optimal space/time):
`T: o(n) S: o(1) n: len(nums)`
```python
class Solution:
  def numDecodings(self, s):
    if not s or s[0] == '0': return 0
    prev = curr = 1
    for i in range(2, len(s)+1):
      tmpCurr = 0
      if 0 < int(s[i-1:i]): tmpCurr += curr
      if 10 <= int(s[i-2:i]) <= 26: tmpCurr += prev
      curr, prev = tmpCurr, curr
    return curr
```

### 122 Best time to buy and sell stock ii
* peak and valley algorithm

class Solutiion:
`T: o(n), s: o(1) n = len(prices)`
```python
class Solution:
  def maxProfit(self, prices):
    res = 0
    for i in range(len(prices)-1):
      if prices[i+1] > prices[i]: res += prices[i+1]-prices[i]
    return res
```