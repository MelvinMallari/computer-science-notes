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