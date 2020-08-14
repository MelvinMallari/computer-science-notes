# Algorithms and Data Structure Notes

## Resources

[leetcode resources](https://leetcode.com/discuss/general-discussion/665604/important-and-useful-links-from-all-over-the-leetcode)
[facebook q's](https://leetcode.com/discuss/general-discussion/675445/facebook-interview-experiences-all-combined-from-lc-till-date-07-jun-2020)
[amazon online assessments](https://leetcode.com/discuss/interview-question/344650/Amazon-Online-Assessment-Questions)

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
Solution 2:
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

## 581 Shortest Unsorted Continuous Subarray
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
	* `if 10 <= int(s[i-2:i]) <= 26: tmpCurr += prev` represents a match of number within [10, 26]

	* `prev = curr = 1`
	* `tmpCurr = 0`
	* `if 0 < int(s[i-1:i]): tmpCurr += curr` represents a match of number within [1, 9]
	* `if 10 <= int(s[i-2:i]) <= 26: tmpCurr += prev` represents a match of number within [10, 26]

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

## 122 Best time to buy and sell stock ii
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

## 38 Count and Say
* Loop through n-1 and count it up

class Solution:
`T: o(n^2?), s: o(1) hard to reason`
```python
class Solution:
	def countAndSay(self, n):
		s = '1'
		for _ in range(n-1):
			curr, tmp,  count = s[0], '', 1
			for i in range(1, len(s)):
				if s[i] == curr:
					count += 1
				else:
					tmp += str(count) + curr
					curr = s[i]
					count = 1
			tmp += str(count) + curr
			s = tmp
		return s
```

## 66 Plus One
* Loop through the array backwards, turn the 9's into zeros. If we reach the end of the array, append a 1 to the beginning

Solution:
`T: o(n), s: o(1) n: len(digits)`
```python
class Solution:
	def plusOne(self, digits):
		for i in range(len(digits)-1, -1, -1):
			if digits[i] != 9:
				digits[i] += 1
				return digits
			digits[i] = 0
		return [1] + digits
```

## 144 Binary Tree Preorder Traversal
`T: o(n), s: o(n) n: num nodes`
```python
class Solution:
	def preorderTraversal(self, root):
		if not root: return
		res, stack = [], [root]
		while stack:
			node = stack.pop()
			res.append(node.val)
			if node.right: stack.append(node.right)
			if node.left: stack.append(node.left)
		return res
```

## 103 Binary Tree Zig Zag Level Order Traversal
Solution I came up with:
`T: o(n), s: o(n) n: num nodes`
```python
class Solution:
	def zigzagLevelOrder(self, root):
		if not root: return []
		res, level, counter = [], [root], 0
		while level:
			leaves = []
			if counter % 2 == 0:
				res.append(n.val for n in level)
			else:
				res.append(n.val for n in level[::-1])
			for n in level:
				if n.left: leaves.append(n.left)
				if n.right: leaves.append(n.right)
			level = leaves
			counter += 1
		return res
```
Fancy Solution I did not come up with:
```python
class Solution:
	def zigzagLevelOrder(self, root):
		if not root: return []
		res, level, direction = [], [root], 1
		while level:
			res.append(n.val for n in level[::direction])
			leaves = [kid for n in level for kid in (n.left, n.right) if kid]
			level = leaves
			direction *= -1
		return res
```

## 116 Populating Next Right Pointers in Each Node
* We want to traverse through starting through the left most nodes and make our way to the right

Solution:
`T:o(n), s:o(1). n: num nodes in tree`
```python
class Solution:
	def connect(self, root):
		curr = root
		while curr and curr.left:
			tmp = curr
			while tmp:
				tmp.left.next = tmp.right
				if tmp.next: tmp.right.next = tmp.next.left
				tmp = tmp.next
			curr = curr.left
		return root
```

## 680 Valid Palindrome II
* Use two pointers moving towards each other. If we find a mismatch, delete either and check if you can make a palindrome.
* You can just letters between left and right pointer, because you've confirmed matches outside of them
Solution:
`T:o(n), s:o(1). n: num nodes in tree`
``
```python
class Solution:
	def validPalindrome(self, s):
		l, r = 0, len(s)-1
		while l < r:
			if s[l] != s[r]:
				dL, dR = s[l:r], s[l+1:r+1]
				return dl == dl[::-1] or dR == dR[::-1]
			l += 1
			r -= 1
		return True
```
## 10 Regular Expression Matching
* if `p[j-1] (current char in string p) != '*'`
	* if `s[i-1] == p[j-1] or p[j-1] == '.'` if match, or match operator
		* `dp[i][j] = dp[i-1][j-1]`
* else
	* if `ch == '*'`
		* wildcard operator can either propogate horizontally or vertically
			* horizontal propogation: `dp[i][j] = dp[i][j-2]`
			* vertical propogation: `dp[i][j] |= dp[i-1][j]`
				* only check vertical propogation if there's a char match or match operator prior to wildcard operator
				* |= because the horizontal propogation might have returned true.

Solution:
`T:o(mn), s:o(mn). m: len(s) n: len(p)`
```python
class Solution:
	def isMatch(self, s, p):
		dp = [[False]*(len(p)+1) for _ in range(len(s)+1)]
		dp[0][0] = True
for j in range(2, len(p)+1):
			if p[j-1] == '*': dp[0][j] = dp[0][j-2]

		for i in range(1, len(s)+1):
			for j in range(1, len(p)+1):
				if p[j-1] != '*':
					dp[i][j] = (s[i-1] == p[j-1] or p[j-1] == '.') and dp[i-1][j-1]
				else:
					dp[i][j] = dp[i][j-2]
					if p[j-2] == s[i-1] or p[j-2] == '.': dp[i][j] |= dp[i-1][j]
		return dp[-1][-1]
```

## 118 Pascal's Triangle
Solution:
`T:o(n^2), s:o(1) n: numRows`
```python
class Solution:
		def generate(self, numRows: int) -> List[List[int]]:
				res = [[1]*(i+1) for i in range(numRows)]
				for i in range(2, numRows):
						for j in range(1, i):
								res[i][j] = res[i-1][j-1] + res[i-1][j]
				return res
```

## 127 Word Ladder
* key idea is to explore minimum conversion distance with a BFS
* key idea is that you can compare the words with other words in the word list by using a map of lists

Solution
`T:o(n^2), s:o(1) n: numRows`
```python
def constructWD(wordList):
		d = collections.defaultdict(list)
		for w in wordList:
				for i in range(len(w)):
						s = w[:i] + '_' + w[i+1:]
						d[s].append(w)
		return d
	```

Solution:
```python
class Solution(object):
		def ladderLength(self, beginWord, endWord, wordList):
				def constructWD(wordList):
						d = collections.defaultdict(list)
						for w in wordList:
								for i in range(len(w)):
										s = w[:i] + '_' + w[i+1:]
										d[s].append(w)
						return d

				def bfs(begin, end, wordDict):
						q, visited = collections.deque([(begin, 1)]), set()
						while q:
								word, count = q.popleft()
								if word == end: return count
								for i in range(len(word)):
										s = word[:i] + '_' + word[i+1:]
										nbrs = wordDict[s]
										for nbr in nbrs:
												if nbr not in visited:
														visited.add(nbr)
														q.append((nbr, count+1))
						return 0

				d = constructWD(set(wordList))
				return bfs(beginWord, endWord, d)
```

## 171 Excel Sheet Column Number
* Each number is a power of 26, kind like a hex number

Solution:
`T:o(n), S:o(1) n: len(s)`
```python
class Solution:
	def titleToNumber(self, s):
		res = 0
		for l in s:
			res = 26*res + ord(l) - ord('A') + 1
		return res
```

## 130 Surrounded Regions
* dfs all the `O`'s around the edges convert that to some character, call it `#`
* dfs from the inside and the `O`'s from the inside convert to `X`
* convert all the `#`'s to `O`

Solution:
`T:o(mn) S:o(1) m: len(board), n: len(board[0])`
```python
class Solution:
	def solve(self, board):
		if not board and not board[0]: return
		m, n = len(board), len(board[0])
		if m < 3 or n < 3: return board

		for i in range(m):
			if board[i][0] == 'O': self.dfs(board, i, 0, '#')
			if board[i][n-1] == 'O': self.dfs(board, i, n-1, '#')

		for j in range(1, n):
			if board[0][j] == 'O': self.dfs(board, 0, j, '#')
			if board[m-1][j] == 'O': self.dfs(board, m-1, j, '#')

		for i in range(1, m-1):
			for j in range(1, n-1):
				if board[i][j] == 'O': self.dfs(board, i, j, 'X')

		for i in range(m):
			for j in range(n):
				if board[i][j] == '#': self.dfs(board, i, j, 'O')

	def dfs(self, board, i, j, mark):
		if i < 0 or i >= len(board) or j < 0 or j >= len(board[0]) or board[i][j] == 'X': return
		board[i][j] = mark
		self.dfs(board, i+1, j, mark)
		self.dfs(board, i-1, j, mark)
		self.dfs(board, i, j-1, mark)
		self.dfs(board, i, j+1, mark)
```
## 131 Palindrome Partitioning
* dfs + backtracking
* How to recognize backtracking?
	* a string can be partitioned in a different _combination_ of ways
	* string partitions do _not_ have to be the same length.
		* if any part of the string is valid palindrome, then explore the rest of the string to confirm that it is a palindrome

Solution:
`T:o(n!), S: o(n!) n: len(s)`
```python
class Solution:
	def partition(self, s):
		res = []
		self.dfs(res, [], s)
		return res

	def dfs(self, res, path, s):
		if not s:
			res.append(path)
			return
		for i in range(1, len(s)+1):
			if self.isPal(s[:i]):
				self.dfs(res, path+[s[:i]], s[i:])

	def isPal(self, w):
		return w == w[::-1]

```

## 134 Gas Station
* Two key ideas:
	* if a car starts at A and cannot reach point B. Any station between A and B cannot reach B. `(1)`
		* Similar to the idea in Kadane's algorithm. Idea is that you start a station with >= 0 gas, otherwise you wouldn't have been able to reach it
	* if total gas available > total cost, then there exists a solution: This one makes intuitive sense, with enough gas you can reach any station. `(2)`

Solution:
`T:o(n), S:o(1) n: len(gas)`
```python
class Solution:
	def canCompleteCircuit(self, gas, cost):
		totalGas = totalCost = tank = start = 0
		for i in range(len(gas)):
			totalGas += gas[i]
			totalCost += cost[i]
			tank += gas[i] - cost[i]
			if tank < 0:
				start = i + 1 # key idea (1)
				tank = 0
		return start if totalGas >= totalCost else -1 # key idea (2)
```

## 162 Peak Element
* Realize that a peak element is just the largest number in the sorted portion of an array
* therefore we can use binary search, because we are guaranteed a peak number and therefore a sorted portion

Solution:
`T: o(logn) S: o(1). n: len(nums)`
```python
class Solution:
	def findPeakElement(self, nums):
		i, j = 0, len(nums)-1
		while i < j:
			mid = (i + j) // 2
			if nums[mid+1] > nums[mid]:
				i = mid + 1
			else:
				j = mid
		return i
```

## 189 Rotate Array
* have to mod k with the len(nums) in case k > len(nums)
* learned that nums[:] modifies original array
* way to do this in o(1) space to reverse the entire array, then reverse the two portions

Solution 1:
`T: o(n), s: o(n), n: len(nums)`
```python
class Solution:
	def rotate(self, nums, k):
		k = k % len(nums)
		nums[:] = nums[-k:] + nums[:-k]
```

Solution 2:
`T: o(n), s: o(1), n: len(nums)`
```python
class Solution:
	def rotate(self, nums, k):
		def reverse(i, j):
			while i < j:
				nums[i], nums[j] = nums[j], nums[i]
				i += 1
				j -= 1
		k, n = k % len(nums), len(nums)
		if k: # not necessary speeds up some test cases
			reverse(0, n-1)
			reverse(0, k-1)
			reverse(k, n-1)
```

## 210 Course Schedule II
* topological sort

Solution 1 (recursive dfs):
`T: o(v + e), s: o(n), n: numCourses, v: num vertices, e: num edges`
```python
class Solution:
	def findOrder(self, numCourses, prereqs):
		graph = collections.defaultdict(set)
		visited = [0 for _ in range(numCourses)]
		for x, y in prereqs:
			graph[x].add(y)
		res = []
		for i in range(numCourses):
			if not self.dfs(i, graph, visited, res): return []
		return res

	def dfs(self, course, graph, visited, res):
		if visited[course] == -1: return False
		if visited[course] == 1: return True
		visited[course] = -1
		for nbr in graph[course]:
			if not self.dfs(nbr, graph, visited, res): return False
		visited[course] = 1
		res.append(course)
		return True
```

Solution 2 (iterative dfs):
`T: o(v + e), s: o(n), n: numCourses, v: num vertices, e: num edges`
```python
class Solution:
	def findOrder(self, numCourses, prereqs):
		graph = collections.defaultdict(set)
		indegree = collections.defaultdict(int)
		for x, y in prereqs:
			indegree[x] += 1 # indegree in this case is how many prereqs a course has {course: {prereqs....}}
			graph[y].add(x) #{prereqs: {courses...}}
		stack, res = [c for c in range(numCourses) if indegree[c] == 0], []
		while stack:
			course = stack.pop()
			res.append(course)
			for nbr in graph[course]:
				indegree[nbr] -= 1
				if indegree[nbr] == 0: stack.append(nbr)
			del graph[course]
		return res if len(res) == numCourses else []
```

Solution 2 (bfs):
`T: o(v + e), s: o(n), n: numCourses, v: num vertices, e: num edges`
```python
class Solution:
	def findOrder(self, numCourses, prereqs):
		graph = collections.defaultdict(set)
		indegree = collections.defaultdict(int)
		for x, y in prereqs:
			indegree[x] += 1
			graph[y].add(x)
		q = collections.deque([c for c in range(numCourses) if indegree[c] == 0])
		res = []
		while q:
			course = q.popleft()
			res.append(course)
			for nbr in graph[course]:
				indegree[nbr] -= 1
				if indegree[nbr] == 0: q.append(nbr)
			del graph[course]
		return res if len(res) == numCourses else []
```
## 140 Word Break II
* dynamic programming
* recursively explore rest of the word after word match, if letters left in s to explore
* if match, then append the word with a space to all the matches
	* as we backtrack up the recursive calls, then we append matched words a long the way (with breaks/spaces)
* as backtrack up we will memoize the result

**Base Cases**
* if not s, return [] - no string, no result
* if s starts with word, and len(word) == len(s) - append to result without space, this is end of word break

Solution:
`T: o(mn) S:o(mn) m: len(s), n: len(wordDict)`
```python
class Solution:
	def wordBreak(self, s, wordDict):
		return self.helper(s, wordDict, {})

	def helper(self, s, wordDict, memo):
		if s in memo: return memo[s]
		if not s: return []
		res = []
		for word in wordDict:
			if not s.startswith(word): continue
			if len(word) == len(s):
				res.append(word)
			else:
				rest = self.helper(s[len(word):], wordDict, memo)
				for item in rest:
					item = word + ' ' + item
					res.append(item)
		memo[s] = res
		return res
```

## 150 Reverse Polish Notation
* use a stack
* loop through tokens:
	* if current token in set of operators
		* pop top two values and apply relevant operator
	* else
		* append to stack

Solution:
`t: o(n), s:o(n) n: len(token)`
```python
class Solution:
	def evalRPN(self, tokens):
		stack = []
		for token in tokens:
			if token not in '+-*/':
				stack.append(int(token))
			else:
				y, x = stack.pop(), stack.pop()
				if token == '+':
					stack.append(x+y)
				elif token == '-':
					stack.append(x-y)
				elif token == '*':
					stack.append(x*y)
				else:
					stack.append(int(x/y))
		return stack[-1]
```

## 454 4Sum ii
* invariant: (i, j, k, l) such that A[i] + B[j] + C[k] + D[l] = 0
* can be rewritten as A[i] + B[j] = -(C[k] + D[l])
* use a hasmap to store all combinations of a + b sums - find all c + d sums that are complements to a+b

`T: o(n^2), S: o(n^2) n: len(A or B or C or D)`
```python
class Solution:
	def fourSumCount(self, A, B, C, D):
		hashMap = collections.defaultdict(int)
		count = 0
		for a in A:
			for b in B:
				hashMap[a+b] += 1
		for c in C:
			for d in D:
				count += hashMap[-c-d]
		return count
```

## 395 Longest Substring with at least K repeating characters
* substring -> sliding window
* if h represents the num unique characters, k min repetitions for every character
	* want to find the max length of the candidate solutions that fulfill the h and k requirements

Solution:
`T: o(n) S: o(n) n: len(s)`
```python
class Solution:
	def longestSubstring(self, s, k):
		res = 0
		for i in range(1, 27):
			res = max(res, self.helper(s, k, i))
		return res

	def helper(self, s, k, numUniqCharsTarget):
		start = end = numUniqChars = numNoLessThanK = res = 0
		chMap = collections.defaultdict(int)

		while end < len(s):
			if chMap[s[end]] == 0: numUniqChars += 1
			chMap[s[end]] += 1
			if chMap[s[end]] == k: numNoLessThanK += 1
			end += 1

			while numUniqChars > numUniqCharsTarget:
				if chMap[s[start]] == k: numNoLessThanK -= 1
				chMap[s[start]] -= 1
				if chMap[s[start]] == 0: numUniqChars -= 1
				start += 1

			if numUniqChars == numNoLessThanK: res = max(res, end - start)

		return res
```

## 76 Minimum Window Substring
* sliding window
* keep count of the characters within the window range.
* expand to the right while we have relevant characters still to include in the substring
* close the window while non-relevant characters are included

Solution:
`T: o(n), S: o(n) n: len(s)`
```python
class Solution:
	def minWindow(self, s, t):
		counter = collections.Counter(t)
		start = end = i = 0
		missing = len(t)
		for j, ch in enumerate(s, 1):
			if counter[ch] > 0: missing -= 1
			counter[ch] -= 1
			if missing == 0:
				while i < j and counter[s[i]] < 0:
					counter[s[i]] += 1
					i += 1

				if end == 0 or j - i < end - start:
					start, end = i, j

				counter[s[i]] += 1
				i += 1
				missing += 1
		return s[start:end]
```

## 3 Longest Substring without repeating characters
* sliding window

Solution 1:
`T: o(n), S: o(n) n: len(s)`
```python
class Solution:
	def lengthOfLongestSubstring(self, s):
		seen = collections.defaultdict(int)
		i = res = 0
		for j, ch in enumerate(s):
			if ch in seen and i <= seen[ch]:
				i = seen[ch] + 1
			else:
				res = max(res, j - i + 1)
			seen[ch] = j
		return res
```

## 384 Shuffle an Array

Solution:
`T: o(n), S: o(1). n: len(nums)`
```python
class Solution:
	def __init__(self, nums):
		self.nums = nums

	def reset(self):
		return self.nums

	def shuffle(self):
		res = self.nums[:]
		for i in range(len(res)):
			j = random.randrange(i+1)
			res[i], res[j] = res[j], res[i]
		return res
```

## 380 Insert Delete GetRandom O(1)

Solution:
`T: o(1), S: o(n). n: len(nums)`

```python
class RandomizedSet:
	def __init__(self):
		self.array, self.hashMap = [], {}

	def insert(self, val):
		if val not in self.hashMap:
			self.hashMap[val] = len(self.array)
			self.array.append(val)
			return True
		return False

	def delete(self, val):
		if val in self.hashMap:
			i = self.hashMap[val]
			last = self.array[-1]
			self.array[i] = last
			self.hashMap[last] = i
			self.array.pop()
			del self.hashMap[val]
			return True
		return False

	def getRandom(self):
		return self.array[random.randrange(len(self.array))]
```


## 378 Kth Smallest in a Sorted Matrix
Solution 1:
`t: o(min(k,n)*klog(n)) s: min(k, n). n: len(matrix)`
```python
from heapq import *
class Solution:
	def kthSmallest(self, matrix, k):
		minHeap = []
		for i in range(min(k, len(matrix))):
			heappush(minHeap, (matrix[i][0], i, 0))
		count = 0
		while minHeap:
			n, row, col = heappop(minHeap)
			count += 1
			if count == k: return n
			if col + 1 < len(matrix[row]): heappush(minHeap, (matrix[row][col+1], row, col+1))
		return None
```

Solution 2:
```python
class Solution:
	def kthSmallest(self, matrix, k):
		start, end = matrix[0][0], matrix[-1][-1]
		while start < end:
			mid = (start + end) // 2
			smaller, larger = matrix[0][0], matrix[-1][-1]
			count, smaller, larger = self.helper(matrix, mid, smaller, larger)
			if count == k: return smaller
			if count < k:
				start = larger
			else:
				end = smaller
		return start

	def helper(self, matrix, mid, smaller, larger):
		count, n = 0, len(matrix)
		row, col = n - 1, 0
		while row >= 0 and col < n:
			if matrix[row][col] > mid:
				larger = min(larger, matrix[row][col])
				row -= 1
			else:
				smaller = max(smaller, matrix[row][col])
				count += row + 1
				col += 1
		return count, smaller, larger
```

## 328 Odd Even Linked List
* two pointers, weaving through linked list
* point the odd list to the even list
```python
class Solution:
	def oddEvenList(self, head):
		if not head or not head.next: return head
		oddH, evenH = head, head.next
		currO, currE = oddH, evenH
		while currO.next and currO.next.next:
			currO.next, currE.next = currO.next.next, currE.next.next
			currO, currE = currO.next, currE.next
		if currE: currE.next = None
		currO.next = evenH
		return oddH
```

## 350 Intersection of Two Arrays II
Solution 1:
`t: o(n), s: o(m). n: len(nums1), m: len(nums2)`
```python
class Solution:
	def intersect(self, nums1, nums2):
		count, res = collections.Counter(nums1), []
		for n in nums2:
			if count[n] > 0:
				res.append(n)
				count[n] -= 1
		return res
```

Solution 2:
`t: o(max(nlogn, mlogm): s: o(1) n: len(nums1), m: len(nums2)`
```python
class Solution:
	def intersect(self, nums1, nums2):
		n1, n2, res = sorted(nums1), sorted(nums2), []
		p1 = p2 = 0
		while p1 < len(nums1) and p2 < len(nums2):
			if n1[p1] < n2[p2]:
				p1 += 1
			elif n2[p2] < n1[p1]:
				p2 += 1
			else:
				res.append(n1[p1])
				p1 += 1
				p2 += 1
		return res
```

## 341 Flatten Nested Iterator

Solution 1:
`t: o(max(m, n)) s: o(n) n: len(list) m: largest amount of nesting`
```python
#class NestedInteger:
#    def isInteger(self) -> bool:
#        """
#        @return True if this NestedInteger holds a single integer, rather than a nested list.
#        """
#
#    def getInteger(self) -> int:
#        """
#        @return the single integer that this NestedInteger holds, if it holds a single integer
#        Return None if this NestedInteger holds a nested list
#        """
#
#    def getList(self) -> [NestedInteger]:
#        """
#        @return the nested list that this NestedInteger holds, if it holds a nested list
#        Return None if this NestedInteger holds a single integer
#        """
class NestedIterator:
	def __init__(self, nestedList: [NestedInteger]):
		self.stack = nestedList[::-1]

	def next(self) -> int:
		return self.stack.pop().getInteger()

	def hasNext(self) -> bool:
		while self.stack:
			top = self.stack[-1]
			if top.isInteger(): return True
			self.stack = self.stack[:-1] + top.getList()[::-1]
		return False
```

Solution 2:
`t: o(max(m, n)) s: o(n) n: len(list) m: largest amount of nesting`
```python
class NestedIterator:
	def __init__(self, nestedList: [NestedInteger]):
		self.dq = collections.deque(nestedList)

	def next(self) -> int:
		return self.dq.popleft().getInteger()

	def hasNext(self) -> bool:
		while self.dq:
			if self.dq[0].isInteger(): return True
			first = self.dq.popleft()
			self.dq.extendleft(first.getList()[::-1])
		return False
```

## 326 Power of Three
`t: o(logn), s: o(1) n: how many times original n can be divided by 3`
```python
class Solution:
	def isPowerOfThree(self, n):
		if n == 0: return True
		while n:
			if n == 1: return True
			if n % 3 != 0: return False
			n /= 3
		return False
```

## 334 Increasing Triplet Subsequence
* first is always updated such that it can only go downwards
* second is only updated if first is already update
* e.g. i: [1, 3, 0, 5]
	first: 1, -> 0
	second: 3

Solution:
`T: o(n), S: o(1). n: len(nums)`
```python
class Solution:
	def increasingTriplet(self, nums):
		first = second = float('inf')
		for n in nums:
			if n <= first:
				first = n
			elif n <= second:
				second = n
			else:
				return True
		return False
```
## 237 Delete Node in a Linked List

Solution:
`T: o(1), S: o(1)`
```python
class Solution:
	def deleteNode(self, node):
		node.val = node.next.val
		node.next = node.next.next
```

## 204 Count Primes

```python
class Solution:
	def countPrimes(self, n):
		if n < 3: return 0
		primes = [1]*n
		primes[0] = primes[1] = 0
		for i in range(2, int(n**0.5) + 1):
			if primes[i]:
				for j in range(i*i, n, i):
					primes[j] = 0
		return sum(primes)
```

## 224 Basic Calculator

`T: o(n), S: o(n). n: len(s)`
```python
class Solution:
	def calculate(self, s):
		res, num, sign, stack = 0, 0, 1, []
		for ss in s:
			if ss.isdigit():
				num = num*10 + int(ss)
			elif ss in '-+':
				res += num*sign
				num = 0
				sign = 1 if ss == '+' else -1
			elif ss == '(':
				stack.append(res)
				stack.append(sign)
				res, sign = 0, 1
			elif ss == ')':
				res += num*sign
				res *= stack.pop() # account for previous sign stored in stack
				res += stack.pop() # account for previous result stored in stack
				num = 0
		return res + num*sign
```

## 227 Basic Calculator II
* stack

Solution:
`T: o(n), S: o(n). n: len(s)`
```python
class Solution:
	def calculate(self, s):
		num, prevSign, stack = 0, '+', []
		for i, ss in enumerate(s):
			if ss.isdigit():
				num = num*10 + int(ss)
			if ss in '-+*/' or i == len(s) - 1:
				if prevSign == '-':
					stack.append(-num)
				elif prevSign == '+':
					stack.append(num)
				elif prevSign == '*':
					stack.append(stack.pop()*num)
				else:
					stack.append(int(stack.pop()/num))
				num, prevSign = 0, ss
		return sum(stack)
```

## 172 Factorial Trailing Zeroes

```python
class Solution:
	def trailingZeroes(self, n):
		res = 0
		while n:
			res += n // 5
			n //= 5
		return res
```

## 324 Wiggle Sort II
* sort the array
* odd positions to be the reverse of the first half of array
* even positions to be the reverse of the second half of array

```python
class Solution:
	def wiggleSort(self, nums):
		nums.sort()
		half = len(nums[::2])
		nums[::2], nums[1::2] = nums[:half][::-1], nums[half:][::-1]
```

## 146 LRU Cache
* use hashtable and doubly linked list
* Hashtable {key: Node}
* self.head side represents the Least Recently Used
* self.tail side repesents the Most Recently Used
* Node {key, val}

```python
class Node:
	def __init__(self, key, val):
		self.key = key
		self.val = val
		self.next = None
		self.prev = None

class LRUCache:
	def __init__(self, capacity):
		self.capacity = capacity
		self.d = dict()
		self.head = Node(0, 0)
		self.tail = Node(0, 0)
		self.head.next = self.tail
		self.tail.prev = self.head

	def get(self, key):
		if key in self.d:
			n = self.d[key]
			self._remove(n)
			self._add(n)
			return n.val
		return -1

	def put(self, key, val):
		if key in self.d: self._remove(self.d[key])
		n = Node(key, val)
		self._add(n)
		self.d[key] = n
		if len(self.d) > self.capacity:
			lru = self.head.next
			self._remove(lru)
			del self.d[lru.key]

	def _add(self, n):
		prev = self.tail.prev
		prev.next = n
		n.prev = prev
		n.next = self.tail
		self.tail.prev = n

	def _remove(self, n):
		n.prev.next = n.next
		n.next.prev = n.prev
```

## 56 Merge Intervals
* merge all overlapping intervals given array of intervals, `intervals`
```python
class Solution:
	def merge(self, intervals):
		if not intervals: return
		res = []
		intervals = sorted(intervals, key = lambda x: x[0])
		least, greatest = intervals[0]
		for start, end in intervals[1:]:
			if start <= greatest:
				greatest = max(greatest, end)
			else:
				res.append([least, greatest])
				least, greatest = start, end
		res.append([least, greatest])
		return res
```

## 57 Insert Intervals
```python
class Solution:
	def insert(self, intervals, newInterval):
		i, res = 0, []

		while i < len(intervals) and intervals[i][1] < newInterval[0]:
			res.append(intervals[i])
			i += 1

		while i < len(intervals) and intervals[i][0] <= newInterval[1]:
			newInterval = min(newInterval[0], intervals[i][0]), max(newInterval[1], intervals[i][1])
			i += 1

		res.append(newInterval)

		while i < len(intervals):
			res.append(intervals[i])
			i += 1

		return res
```

## 540 Single Element in a Sorted Array

```python
class Solution:
	def singleNonDuplicate(self, nums):
		l, r = 0, len(nums) - 1
		while l < r:
			m = (l + r) // 2
			if m % 2 == 0 and nums[m] == nums[m+1] or m % 2 == 1 and nums[m] == nums[m-1]:
				l = m + 1
			else:
				r = m
		return nums[l]
```

## How to deal with Buy and Sell stock problems
* Base Cases
```
T[-1][k][0] = T[i][0][0] = 0
T[-1][k][1] = T[i][0][1] = -Infinity
```

* General Recurrence Relationship
```
T[i][k][0] = max(T[i][k][0], T[i-1][k][1] + prices[i])
T[i][k][1] = max(T[i][k][1], T[i-1][k-1][0] - prices[i])
```

**Notation**
* `i`: `ith` day
* `k`: at most `k` transactions
* `prices[i]`: price of stock at `ith` day
* `[0]`: holding 0 stock end of day
* `[1]`: holding 1 stock end of day
* `T[i][k][0]`: Most profit on `ith` day, with at most `k` transactions, holding `0` stock EOD
* `T[i][k][1]`: Most profit on `ith` day, with at most `k` transactions, holding `1` stock EOD

**Explanation**
* Base Cases:
	* `T[-1][k][0] = T[i][0][0] = 0`: because holding 0 stocks in the beginning means 0 profit
	* `T[-1][k][1] = T[i][0][1] = -Infinity`: because having 1 stock to start off with is impossible

* We want a relationship between the `T[i][k][0/1]` and its sub-problems `T[i-1][k][0/1], T[i][k-1][0/1], T[i-1][k-1][0/1]...`
* If we have this relationship we can solve the problem with dynamic programming (since the relationship is recursive)

### 121 Best Time to Buy and Sell Stock
`Case 1: k = 1`

* Recurrence Relationships
```
	T[i][1][0] = max(T[i][1][0], T[i][1][1] + prices[i])
	T[i][1][1] = max(T[i][1][1], T[i][0][0] - prices[i]) = max(T[1][1][0], -prices[i])
```

Solution:`t: o(n), s: o(1). n: len(prices)`
```python
class Solution:
	def maxProfit(self, prices):
		t_i10, t_i11 = 0, float('-inf')
		for p in prices:
			t_i10, t_i11 = max(t_i10, t_i11+p), max(t_i11, -p)
		return t_i10
```

### 122 Best Time to Buy and Sell Stock II
`Case 2: k = Infinity`

* Recurrence Relationships
```
	T[i][k][0] = max(T[i][k[0], T[i][k][1] + prices[i])
	T[i][k][1] = max(T[i][k][1], T[i][k-1][0] - prices[i]) = max(T[i][k][1], T[i][k][0] - prices[i])
```
* Note:
	* if `k = Infinity`: `T[i][k-1][0] = T[i][k][0]`
	* this is because as lim(k) approaching infinity is the same as lim(k-1) approaching infinityA

Solution:`t: o(n), s: o(1). n: len(prices)`
```python
class Solution:
	def maxProfit(self, prices):
		t_ik0, t_ik1 = 0, float('-inf')
		for p in prices:
			t_ik0, t_ik1 = max(t_ik0, t_ik1 + p), max(t_ik1, t_ik0-p)
		return t_ik0
```

### 123 Best Time to Buy and Sell Stock III
`Case 3: k = 2`

* Recurrence Relationships
```
	T[i][2][0] = max(T[i][2][0], T[i][2][1] + prices[i])
	T[i][2][1] = max(T[i][2][1], T[i][1][1] - prices[i])
	T[i][1][0] = max(T[i][1][0], T[i][1][1] + prices[i])
	T[i][1][1] = max(T[i][1][1], T[i][0][1] - prices[i]) = max(T[i][1][1], -prices[i])
```

Solution:`t: o(n), s: o(1). n: len(prices)`
```python
class Solution:
	def maxProfit(self, prices):
		t_i20 = t_i10 = 0
		t_i21 = t_i11 = float('-inf')
		for p in prices:
			t_i20 = max(t_i20, t_i21 + p)
			t_i21 = max(t_i21, t_i10 - p)
			t_i10 = max(t_i10, t_i11 + p)
			t_i11 = max(t_i11, -p)
		return t_i20
```

### 188 Best Time to Buy and Sell Stock IV
`Case 4: k is arbitrary`
* given that there are n stocks (len(prices))
	* there can only be at most n/2 profitable transactions
	* if k >= n/2, this scenario models exactly like case II (infinite transactions possible)
* otherwise we have to consider the states of possible transactions leading up to k

`t:o(kn) s:o(k), k:k, n: len(prices)`
```python
class Solution:
	def maxProfit(self, prices, k):
		if k >= len(prices) // 2:
			tik0, tik1 = 0, float('-inf')
			for p in prices:
				tik0, tik1 = max(tik0, tik1 + p), max(tik1, tik0 - p)
			return tik0
		tik0, tik1 = [0]*(k+1), [float('-inf')]*(k+1)
		for p in prices:
			for j in range(k, 0, -1):
				tik0[j] = max(tik0[j], tik1[j] + p)
				tik1[j] = max(tik1[j], tik0[j-1] - p)
		return tik0[k]
```

### 309 Best Time to Buy and Sell Stock with Cooldown
`Case 5: k is Infinite but with cooldown`

* same as case II but we have consider `i-2`th day instead of `i-1`th date when we consider a transaction (during a buy)

Solution: `t:o(n) s:o(1), n: len(prices)`
```python
class Solution:
	def maxProfit(self, prices):
		tik0Pre, tik0, tik1 = 0, 0, float('-inf')
		for p in prices:
			tik0Old = tik0
			tik0, tik1 = max(tik0, tik1 + p), max(tik1, tik0Pre - p)
			tik0Pre = tik0Old
		return tik0
```

### 714 Best Time to Buy and Sell Stock with Transaction Fee
`Case 6: k is Infinite but with fee`
* similar to case II, but with fee

Solution: `t:o(n) s:o(1), n: len(prices)`
```python
class Solution:
	def maxProfit(self, prices, fee):
		tik0, tik1 = 0, float('-inf')
		for p in prices:
			tik0, tik1 = max(tik0, tik1 + p), max(tik1, tik0 - p - fee)
		return tik0
```

## 278 First Bad Version
Solution: `t:o(n), s:o(1). n: n`
```python
class Solution:
	def firstBadVersion(self, n):
		i, j = 0, n
		while i < j:
			m = (i + j) // 2
			while i < j:
				if isBadVersion(m):
					j = m
				else:
					i = m + 1
		return i
```


## 38 Combination Sum
* we're given a set of candidates with no duplicates (we don't have to sort)
* (1) can reuse the numbers

```python
class Solution:
	def combinationSum(self, candidates, target):
		res = []
		self.backtrack(res, candidates, target, [], 0)
		return res

	def backtrack(self, res, candidates, target, path, index):
		if target < 0: return
		if target == 0:
			res.append(path)
			return
		for i in range(index, len(candidates)):
			self.backtrack(res, candidates, target-candidates[i], path+[candidates[i]], i) # (1)
```

## 39 Combination Sum 2
* we are given duplicates
* only allowed to use numbers once

```python
class Solution:
	def combinationSum2(self, candidates, target):
		res = []
		self.backtrack(res, sorted(candidates), target, [], 0)
		return res

	def backtrack(self, res, candidates, target, path, index):
		if target < 0: return
		if target == 0:
			res.append(path)
			return
		for i in range(index, len(candidates)):
			if i > index and candidates[i] == candidates[i-1]: continue
			self.backtrack(res, candidates, target-candidates[i], path+[candidates[i]], i+1)
```

## 77 Combinations
```python
class Solution:
	def combine(self, n, k):
		res = []
		self.backtrack(res, n, k, [],  1)
		return res

	def backtrack(self, res, n, k, path, index):
		if len(path) == k:
			res.append(path)
			return
		for i in range(index, n+1):
			self.backtrack(res, n, k, path+[i], i+1)
```

## 78 Subsets
```python
class Solution:
	def subsets(self, nums):
		res = []
		self.backtrack(res, nums, [], 0)
		return res

	def backtrack(self, res, nums, path, index):
		res.append(path)
		for i in range(index, len(nums)):
			self.backtrack(res, nums, path+[nums[i]], i+1)
```

## 90 Subsets II
```python
class Solution:
	def subsetsWithDup(self, nums: List[int]) -> List[List[int]]:
		res = []
		self.backtrack(sorted(nums), res, [], 0)
		return res

	def backtrack(self, nums, res, path, index):
		res.append(path)
		for i in range(index, len(nums)):
			if i > index and nums[i] == nums[i-1]: continue
			self.backtrack(nums, res, path+[nums[i]], i+1))
```

## 46 Permutations
```python
class Solution:
	def permute(self, nums):
		res = []
		self.backtrack(res, nums, [])
		return res

	def backtrack(self, res, nums, path):
		if not nums:
			res.append(path)
			return
		for i in range(len(nums)):
			self.backtrack(res, nums[:i]+nums[i+1:], path+[nums[i]])
```

## 47 Permutations II
```python
class Solution:
	def permuteUnique(self, nums):
		res = []
		self.backtrack(res, sorted(nums), [], [False]*len(nums))
		return res

	def backtrack(self, res, nums, path, used):
		if len(path) == len(nums):
			res.append(path)
			return
		for i in range(len(nums)):
			if used[i] or (i > 0 and nums[i] == nums[i-1] and not used[i-1]): continue
			used[i] = True
			self.backtrack(res, nums, path+[nums[i]], used)
			used[i] = False
```

## 60 Permutations Sequence

* given n and k, we want to give the kth permutation
* you can build the kth permutation without building all permutations

Solution:
`t:o(n) s:o(n). n: given n`
```python
class Solution:
	def getPermutation(self, n, k):
		nums = [str(i) for i in range(1, n+1)]
		fact = [1]*n
		for i in range(1, n):
			fact[i] = i*fact[i-1]
		k -= 1
		res = []
		for i in range(n, 0, -1):
			j = k // fact[i-1]
			k %= fact[i-1]
			res.append(nums[j])
			nums.pop(j)
		return ''.join(res)
```

## 367 Valid Perfect Squares
```python
class Solution:
	def isPerfectSquare(self, num):
		l, r = 0, num
		while l <= r:
			m = (l+r)//2
			if m*m == num: return True
			if m*m > num:
				r = m - 1
			else:
				l = m + 1
		return False
```

## 383 Ransom Notes
```python
class Solution:
	def canConstruct(self, ransomNote, magazine):
		count = collections.Counter(magazine)
		for l in ransomeNote:
			if count[l] == 0: return False
			count[l] -= 1
		return True
```

## 402 Remove K Digits
* one way to think about this is if we have a set of sorted digits "123456789"
	* to remove k digits and get the smallest number, start removing k digits from the right
* if we don't have a sorted set of digits, then prioritize the removal numbers that don't fulfill that criteria

```python
class Solution:
	def removeKdigits(self, num, k):
		stack = []
		for n in num:
			while k > 0 and len(stack) > 0 and stack[-1] > n:
				stack.pop()
				k -= 1
			stack.append(n)
		if k > 0: stack = stack[:-k]
		return "".join(stack).lstrip("0") or "0"
```

## 525 Contiguous Array
```python
class Solution:
	def findMaxLength(self, nums):
		count = maxLength = 0
		seen = {count:-1}
		for i, n in enumerate(nums):
			count += 1 if n == 1 else -1
			if count in seen:
				maxLength = max(maxLength, i-seen[count])
			else:
				seen[count] = i
		return maxLength
```

## 451 Sort Characters by Frequency
```python
class Solution:
	def frequencySort(self, s):
		freq = sorted(collections.Counter(s).items(), key=lambda x: x[1], reverse=True)
		return "".join(l*f for l, f in freq)
```

## 1008 Construct Binary Search Tree from Preorder Traversal
Solution 1 (iterative):
`t:o(n^2), s:o(n)`
```python
class Solution:
	def bstFromPreorder(self, preorder):
		root = TreeNode(preorder[0])
		stack = [root]
		for val in preorder[1:]:
			if val < stack[-1].val:
				stack[-1].left = TreeNode(val)
				stack.append(stack[-1].left)
			else:
				while stack and stack[-1].val < val:
					last = stack.pop()
				last.right = TreeNode(val)
				stack.append(last.right)
		return root
```
Solution 2 (recursive):
`t:o(n^2), s:o(n)`
```python
class Solution:
	def bstFromPreorder(self, preorder):
		if not preorder: return
		root, i = TreeNode(preorder[0]), 1
		while i < len(preorder) and preorder[i] < root.val: i += 1
		root.left = self.bstFromPreorder(preorder[1:i])
		root.right = self.bstFromPreorder(preorder[i:])
		return root
```

## 993 Cousins in Binary Tree
* all nodes have a unique integer val
* BFS
* we check if children both exists and are x and y -> return False
* if vals x and y exist in the same level and are sibilings -> return True
```python
class Solution:
	def isCousins(self, root, x, y):
		level = [root]
		while level:
			leaves, vals = [], set()
			for n in level:
				if self.sibs(n, x, y): return False
				if n.left:
					leaves += n.left,
					vals.add(n.left.val)
				if n.right:
					leaves += n.right,
					vals.add(n.right.val)
			if x in vals and y in vals: return True
			level = leaves
		return False

	def sibs(self, n, x, y):
		return n.left and n.right and n.left.val in (x, y) and n.right.val in (x, y)
```

## 560 Subarray Sums Equals K
* keep track of complements
```python
class Solution:
	def subarraySum(self, nums: List[int], k: int) -> int:
		currSum = res = 0
		preSum = {0:1}
		for n in nums:
			currSum += n
			res += preSum.get(currSum-k, 0)
			preSum[currSum] = preSum.get(currSum, 0) + 1
		return res
```
## 1232 Check if it is a Straight Line
```python
class Solution:
	def checkStraightLine(self, coordinates):
		slope = self.slope(coordinates[0], coordinates[1])
		for i in range(1, len(coordinates)-1):
			if self.slope(coordinates[i], coordinates[i+1]) != slope: return False
		return True

	def slope(self, p1, p2):
		if p2[0] - p1[0] == 0: return float('inf')
		return (p2[1] - p1[1]) / (p2[0] - p1[0])
```

## 218 The Skyline Problem
`t: o(nlogn), s:`
```python
from heapq import heappop, heappush
class Solution:
	def getSkyline(self, buildings: List[List[int]]) -> List[List[int]]:
		events = [(l, -h, r) for l, r, h in buildings] # starting events
		events += list(set((r, 0, 0) for _, r, _ in buildings)) # ending events
		events.sort() # sort events based on occurence

		res = [[0,0]] #[x, height]
		live = [(0, float('inf'))] #(height, endPos)
		for pos, negH, r in events:
			while live[0][1] <= pos: heappop(live)
			if negH: heappush(live, (negH, r))
			if res[-1][1] != -live[0][0]: res += [pos, -live[0][0]],
		return res[1:]
```
## 437. Path Sum III
```python
class Solution:
	def pathSum(self, root: TreeNode, sum: int) -> int:
		if not root: return 0
		self.res = 0
		self.dfs(root, {0:1}, 0, sum)
		return self.res

	def dfs(self, node, preSum, currSum, target):
		if not node: return
		self.res += preSum.get(currSum - target, 0)
		preSum[currSum] = preSum.get(currSum, 0) + 1
		self.dfs(node.left, preSum, currSum, target)
		self.dfs(node.right, preSum, currSum, target)
		preSum[currSum] -= 1
```

## 73 Daily Temperatures
* stack
```python
class Solution:
	def dailyTemperatures(self, T):
		res, stack = [0]*len(T), []
		for i, t in enumerate(T):
			while stack and T[stack[-1]] < t:
				curr = stack.pop()
				res[curr] = i - curr
			stack += i,
		return res
```

## 901 Online Stock Span
* stack
```python
class StockSpanner:
	def __init__(self):
		self.stack = []

	def next(self, price):
		res = 1
		while self.stack and self.stack[-1][0] <= price: res += self.stack.pop()[1]
		self.stack.append((price, res))
		return res
```

## 84 Largest Rectangle in Histogram
```python
class Solution:
	def largestRectangleArea(self, heights):
		heights += 0,
		stack, res = [-1], 0
		for i, height in enumerate(heights):
			while height < heights[stack[-1]]:
				h = heights[stack.pop()]
				w = i-1 - stack[-1]
				res = max(res, w*h)
			stack += i,
		return res
```


## 406 Queue Reconstruction By Height
```python
class Solution:
	def reconstructQueue(self, people):
		if not people: return people
		res, heights, pplDict = [], [], collections.defaultdict(list)
		for h, k in people:
			if not pplDict[h]: heights += h,
			pplDict[h] += k,
		for h in sorted(heights, reverse=True):
			for k in sorted(pplDict[h]):
				res.insert(k, [h, k])
		return res
```


## 621 Task Scheduler
```python
from heapq import heapify, heappop, heappush
class Solution:
	def leastInterval(self, tasks, n):
		heap, intervals = [-v for v in collections.Counter(tasks).values()], 0
		heapify(heap)
		while heap:
			steps, tmp = 0, []
			while steps <= n:
				intervals += 1
				steps += 1
				if heap:
					freq = heappop(heap)
					if freq < -1: tmp += freq + 1,
				if not heap and not tmp: break
			for i in tmp: heappush(heap, i)
		return intervals
```

## 918 Maximum Sum Circular Subarray
```python
class Solution:
	def maxSubarraySumCircular(self, A):
		total, currMin, minSum, currMax, maxSum = 0, 0, A[0], 0, A[0]
		for a in A:
			currMax = max(currMax + a, a)
			maxSum = max(maxSum, currMax)
			currMin = min(currMin+a, a)
			minSum = min(minSum, currMin)
			total += a
		return max(maxSum, total - minSum) if maxSum > 0 else maxSum
```

## 1035 Uncrossed Lines
* dp problem
Solution 1:
`t:o(nm), s:o(nm) n: len(A), m: len(B)`
```python
class Solution:
	def maxUncrossedLines(self, A, B):
		m, n = len(A), len(B)
		dp = [[0]*(n+1) for _ in range(m+1)]
		for i in range(1, m+1):
			for j in range(1, n+1):
				if A[i-1] == B[j-1]:
					dp[i][j] = 1 + dp[i-1][j-1]
				else:
					dp[i][j] = max(dp[i-1][j], dp[i][j-1])
		return dp[-1][-1]
```

## 329 Longest Increasing Path in a Matrix
```python
class Solution:
	def longestIncreasingPath(self, matrix):
		if not matrix or not matrix[0]: return 0
		m, n = len(matrix), len(matrix[0])
		dp = [[0]*n for _ in range(m)]
		res = 0
		for i in range(m):
			for j in range(n):
				res = max(res, self.dfs(matrix, i, j, m, n, dp))
		return res

	def dfs(self, matrix, i, j, m, n, dp):
		if dp[i][j]: return dp[i][j]
		dp[i][j] = 1
		for di, dj in ((-1, 0), (1, 0), (0, -1), (0, 1)):
			x, y = i + di, j + dj
			if not 0 <= x < m or not 0 <= y < n or matrix[x][y] <= matrix[i][j]: continue
			dp[i][j] = max(dp[i][j], 1 + self.dfs(matrix, x, y, m, n, dp))
		return dp[i][j]
```

## 54 Spiral Matrix
* we have l, r, u, d variables to keep track of the current perimeter
* increment and decrement those as necessary as we move the considered perimeter towards center
* our while condition is `l <= r and u <= d` because `r = len(matrix[0])-1 and d = len(matrix) - 1`
`t:o(mn), s:o(1) m:len(matrix), n:len(matrix[0])`
```python
class Solution:
	def spiralOrder(self, matrix):
		if not matrix or not matrix[0]: return []
		m, n = len(matrix), len(matrix[0])
		l, r, u, d = 0, n-1, 0, m-1
		res = []
		while l <= r and u <= d:
			for j in range(l, r+1):
				if len(res) < m*n: res += matrix[u][j],
			for i in range(u+1, d):
				if len(res) < m*n: res += matrix[i][r],
			for j in range(r, l-1, -1):
				if len(res) < m*n: res += matrix[d][j],
			for i in range(d-1, u, -1):
				if len(res) < m*n: res += matrix[i][l],
			l += 1
			r -= 1
			u += 1
			d -= 1
		return res
```

### 498 Diagonal Traverse
`t: o(mn), s: o(1) n: len(matrix), m: len(matrix[0])`
```python
class Solution:
	def findDiagonalOrder(self, matrix):
		if not matrix or not matrix[0]: return []
		m, n = len(matrix), len(matrix[0])
		row, col, d = 0, 0, 1
		res = []
		for _ in range(m*n):
			res += matrix[row][col],
			row -= d
			col += d

			if row >= m: row, col, d = m-1, col + 2, -d
			if col >= n: row, col, d = row + 2, n-1, -d
			if row < 0: row, d = 0, -d
			if col < 0: col, d = 0, -d
		return res
```

## 935 Knight Dialer
`t: o(n), s: o(1)`
```python
class Solution:
	def knightDialer(self, N):
		moves = {1:[6, 8], 2:[7, 9], 3:[4, 8], 4:[0, 3, 9], 5:[], 6:[0, 1, 7], 7:[2, 6], 8:[1, 3], 9:[2, 4], 0:[4, 6]}
		dp = [1]*10
		for _ in range(N-1):
			tmp = [0]*10
			for i in range(10):
				for j in moves[i]:
					tmp[i] += dp[j]
			dp = tmp
		return sum(dp)%(10**9 + 7)
```

## 23 Merge K Sorted Lists
`t: o(nklogk) s:o(k) n: avg length of lists, k: num sorted lists`
```python
from heapq import heapify, heappush, heappop
class Solution:
	def mergeKLists(self, lists):
		heap = [(l.val, i, l) for i, l in enumerate(lists) if l]
		heapify(heap)
		count = len(lists)
		dummy = curr = ListNode()
		while heap:
			curr.next = heappop(heap)[2]
			curr = curr.next
			if curr.next:
				count += 1
				heappush(heap, (curr.next.val, count, curr.next))
		return dummy.next
```

## 33 Search in a Rotated Sorted Array
* at each point figure out which halfs are sorted and unsorted
* check if the target fits in the sorted portion, binary search through that if so
* if it doesn't fit, then repeat the algorithm on unsorted portion
```python
class Solution:
	def search(self, nums, target):
		if not nums: return -1
		l, r = 0, len(nums) - 1
		while l <= r:
			m = (l+r) // 2
			if nums[m] == target: return m
			if nums[m] > nums[r]:
				if nums[l] <= target <= nums[m]:
					r = m - 1
				else:
					l = m + 1
			else:
				if nums[m] <= target <= nums[r]:
					l = m + 1
				else:
					r = m - 1
		return -1
```

## 303 Range Sum Query Mutable

```python
class NumArray:
	def __init__(self, nums):
		self.accu = [0]
		for n in nums:
			self.accu += self.accu[-1] + n,

	def sumRange(self, i, j):
		return self.accu[j+1]-self.accu[i]
```

### 435 Non-Overlapping Intervals
```python
class Solution:
	def eraseOverlapIntervals(self, intervals):
		if not intervals: return 0
		currEnd = float('-inf')
		res = 0
		for start, end in sorted(intervals, key=lambda x: x[1])
			if currEnd <= start:
				currEnd = end
			else:
				res += 1
		return res
```

## 1277 Count Square Submatrices with All Ones
```python
class Solution:
	def countSquares(self, matrix):
		if not matrix or not matrix[0]: return 0
		m, n = len(matrix), len(matrix[0])
		dp = [row[:] for row in matrix]
		for i in range(1, m):
			for j in range(1, n):
				if dp[i][j]:
					dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
		return sum(sum(row) for row in dp)
```
## 567 Permutaiton in String
* sliding window
```python
from collections import Counter
class Solution:
	def checkInclusion(self, s1, s2):
		c1, c2 = Counter(s1), Counter(s2[:len(s1)-1])
		for i in range(len(s1)-1, len(s2)):
			c2[s2[i]] += 1
			if c1 == c2: return True
			start = i + 1 - len(s1)
			c2[s2[start]] -= 1
			if c2[s2[start]] == 0: del c2[s2[start]]
		return False
```

## 886 Possible Bipartition
* dfs
```python
class Solution:
	def possibleBipartition(self, N: int, dislikes: List[List[int]]) -> bool:
		NOT_COLORED, BLUE, GREEN = 0, 1, -1
		def helper(personID, color):
			colorTable[personID] = color
			for other in dislikeTable[personID]:
				if colorTable[other] == color:
					return False
				if colorTable[other] == NOT_COLORED and not helper(other, -color):
					return False
					return True
		colorTable = [NOT_COLORED]*(N+1)
		dislikeTable = collections.defaultdict(list)
		for a, b, in dislikes:
			dislikeTable[a] += b,
			dislikeTable[b]
		for person in range(1, N+1):
			if colorTable[person] == NOT_COLORED and not helper(person, BLUE):
				return False
		return True
```

### 297 Serialize and Deserialize Binary Tree
* preorder traversal
```python
class Codec:
		def serialize(self, root):
				def doit(node):
						if node:
								vals.append(str(node.val))
								doit(node.left)
								doit(node.right)
						else:
								vals.append('#')
				vals = []
				doit(root)
				return ' '.join(vals)

		def deserialize(self, data):
				vals = iter(data.split())
				def doit():
						val = next(vals)
						if val == '#':
								return None
						node = TreeNode(int(val))
						node.left = doit()
						node.right = doit()
						return node
				return doit()
```

## 543 Diameter of Binary Tree
Solution 1:
`t: o(v + e), s: o(v+e) v: num vertices, e: num edges`
```python
class Solution:
		def diameterOfBinaryTree(self, root: TreeNode) -> int:
				self.res = 0
				def traverse(node):
					if not node: return 0
					lh, rh = traverse(node.left), traverse(node.right)
					self.res = max(self.res, lh + rh)
					return 1 + max(lh, rh)
				traverse(root)
				return self.resa
```

### 687 Longest Univalue Path
```python
class Solution:
	def longestUnivaluePath(self, root: TreeNode) -> int:
		self.res = 0
		def traverse(node):
			if not node: return 0
			lh, rh = traverse(node.left), traverse(node.right)
			l = r = 0
			if node.left and node.val == node.left.val: l = lh + 1
			if node.right and node.val == node.right.val: r = rh + 1
			self.res = max(self.res, l + r)
			return max(l, r)
		traverse(root)
		return self.res
```

### 889 Construct Binary Tree from Preorder and Postorder Traversal
```python
class Solution:
	def constructFromPrePost(self, pre: List[int], post: List[int]) -> TreeNode:
		if not pre or not post: return None
		node = TreeNode(pre[0])
		if len(pre) == 1: return node
		i = pre.index(post[-2])
		node.left = self.constructFromPrePost(pre[1:i], post[:(i-1)])
		node.right = self.constructFromPrePost(pre[i:], post[(i-1):-1])
		return node

```

### 787 Cheapest Flights Within K Stops
```python
class Solution:
	def findCheapestPrice(self, n: int, flights: List[List[int]], src: int, dst: int, K: int) -> int:
		visited = {}
		graph = collections.defaultdict(list)
		for s, d, p in flights:
			graph[s] += (d, p),
		heap = [(0, 0, src)]
		while heap:
			dist, moves, node = heapq.heappop(heap)
			if node == dst: return dist
			if node not in visited or moves < visited[node]:
				visited[node] = moves
				if moves + 1 <= K+1:
					for nbr, weight in graph[node]:
						heapq.heappush(heap, (dist+weight, moves+1, nbr))
		return -1
```

## 35 Search Insert Position
* given a sorted array and a target value, return index if target is found

```python
class Solution:
	def searchInsert(self, nums, target):
		if not nums: return -1
		l, r = 0, len(nums)-1
		while l <= r:
			m = (l + r) // 2
			if nums[m] == target: return m
			if nums[m] < target:
				l = m + 1
			else:
				r = m - 1
		return -1
```

## 1249 Minimum Remove To Make Valid Parentheses
```python
class Solution:
	def minRemoveToMakeValid(self, s: str) -> str:
		stack, curr = [], ''
		for c in s:
			if c == '(':
				stack.append(curr)
				curr = ''
			elif c == ')':
				if stack: curr = stack.pop() + '(' + curr + ')'
			else:
				curr += c
		while stack:
			curr = stack.pop() + curr
		return curr
```

## 348 Design Tic-Tac-Toe
```python
class TicTacToe:
		def __init__(self, n: int):
			self.rows, self.cols = [0]*n, [0]*n
			self.d1 = self.d2 =  0
			self.n = n

		def move(self, row: int, col: int, player: int) -> int:
			mark = 1 if player == 1 else -1
			self.rows[row] += mark
			self.cols[col] += mark
			if row == col: self.d1 += mark
			if row + col == self.n-1: self.d2 += mark
			if self.won(row, col): return player
			return 0

		def won(self, row, col):
			n = self.n
			return abs(self.rows[row]) == n or abs(self.cols[col]) == n or abs(self.d1) == n or abs(self.d2) == n
```

## 763 Partition Labels
```python
class Solution:
	def partitionLabels(self, S):
		last = {c:i for i, c in enumerate(S)}
		res = []
		end = count = 0
		for i, c in enumerate(S):
			end = max(end, last[c])
			count += 1
			if i == end:
				res.append(count)
				count = 0
		return res
```

## 953 Verify an Alien Dictionary
`t: o(nm) s: o(1) n:length(words) m:avg length of word`
```python
class Solution:
	def isAlienSorted(self, words, order):
		index = {c:i for i, c in enumerate(order)}
		for w1, w2 in zip(words, words[1:]):
			if len(w1) > len(w2) and w1[:len(w2)] == w2:
				return False
			for c1, c2 in zip(w1, w2):
				if c1 != c2:
					if index[c1] < index[c2]:
						break
					else:
						return False
		return True
```

## 25 Reverse Nodes in k-group
`t: o(n), s:o(1)`
```python
class Solution:
	def reverseKGroup(self, head, k):
		dummy = jump = ListNode(-1)
		dummy.next = l = r = head
		while True:
			count = 0
			while r and count < k:
				r = r.next
				count += 1
			if count == k:
				curr, pre = l, r
				for _ in range(k): # keep reversing, k times
					curr.next, curr, pre = pre, curr.next, curr
				jump.next, jump, l = pre, l, r
			else:
				return dummy.next
```

## 994 Rotting Oranges
```python
class Solution:
  def orangesRotting(self, grid: List[List[int]]) -> int:
    if not grid or not grid[0]: return 0
    m, n = len(grid), len(grid[0])
    minutes = fresh = 0
    dq = collections.deque()
    for i in range(m):
      for j in range(n):
        if grid[i][j] == 2:
          dq.append((i, j))
        elif grid[i][j] == 1:
          fresh += 1

    while dq and fresh > 0:
      minutes += 1
      for _ in range(len(dq)):
        i, j = dq.popleft()
        for di, dj in ((-1, 0), (1, 0), (0, -1), (0, 1)):
          x, y = i + di, j + dj
          if not (0 <= x < m) or not (0 <= y < n) or grid[x][y] != 1: continue
          grid[x][y] = 2
          fresh -= 1
          dq.append((x, y))
    return minutes if fresh == 0 else -1
```
* there is the edge case for the last iteration where all the oranges are now rotten
  * can handle by checking if fresh oranges are still in play in the while loop:
    ```python
    while dq and fresh > 0:
      ...
    ```
  * can also handle by subtracting 1 from the result
    * also have to handle the edge case where there are no rotten fruit (can't have -1 minutes)
    ```python
      return max(0, minutes -1) if fresh == 0 else -1
    ```