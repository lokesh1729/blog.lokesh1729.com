# Kadane's algorithm

Hello everyone! in this post, we'll discuss kadane's algorithm. I'll try to keep it as simple as possible. The goal of this algorithm is to find a maximum value of the sub-array sum. Let's take an example to understand.

`arr = [7,2,-1,-3,9]`

the maximum sub array sum is = 7+2-1-3+9 = 14

How do we find this? well, the naive approach is to get all sub-array sums and find the maximum.

Let's say we have an array \[a1, a2, a3, a4\]

We'll start with a1 and find all the different sub-arrays

![](https://lokesh1729.com/static/1079a1a04fcc0f5c6cc9339a11253187/fd213/combinations.webp align="center")

Now, we get maximum of all i.e. max(a1+a2, a1+a2+a3, a1+a2+a3+a4, a2+a3, a2+a3+a3, a3+a4)

Well, this works but let's analyze the time complexity. The pseudo-code looks like this

```python
result = -infinity
for i 0...n
    for j in i+1...n
       result = max(result, a[i] + a[j])
return result
```

This will give O(n^2) time complexity.

Let's see how we can optimize this

![](https://lokesh1729.com/static/3eebf0499213171b6cd6fe5a9353c0f9/f8f9f/overlapping_subproblems.webp align="center")

From the above image, we see that there are repeated computations i.e. **overlapping sub-problems**. If we find the maximum of a1+a2 and a1+a2+a3 gives a solution for a sub-problem. We can prove the fact by induction that when combining solutions of all sub-problems will give the overall solution i.e. **optimal sub-structure**.

From these two facts, we can say that we're using a dynamic programming approach.

The idea is to find the maximum of all nested sub-problems eventually we'll reach the solution.

![](https://lokesh1729.com/static/3978a91813570c1cf63bb65f31d9e2ba/5733b/optimal_solution.webp align="center")

As depicted in the above image it is very clear that `max(a1+a2, a1+a2+a3)` gives a local maximum when we do this for all sub-problems we'll get the global maximum.

at every j, local\_max holds maximum sum A\[i\] + A\[i+1\] + .... A\[j-1\] for all i ∈ {1....j}. So, `local_max + A[j]` contains next local maximum. By the end we reach n it'll contain maximum of {1....n-1} elements which gives our result

Let's write code

```python
def find_max_sub_array_sum(arr):
  local_max = arr[0]
  global_max = float('-inf')
  for i in range(1,n):
    local_max = max(arr[i], arr[i] + local_max)
    global_max = max(local_max, global_max)
  return global_max
```

After learning kadane's algorithm, let's try to solve this problem [https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

From the problem description, we need to maximize the profit by finding optimal buy and sell prices. You cannot sell first and buy later.

Example : \[7, 1, 5, 3, 6, 4\]

Answer: we'll buy on day 2 at price 1 and sell on day 5 at price 6 with a total profit of $5.

When I first saw this problem and the above example, I thought of a solution to find the index of minimum elements. From that index, traverse the whole array and find the maximum which will give our answer. But, I was wrong. Let's see how

![](https://lokesh1729.com/static/49c9fa0d1eb720d38b48f04a2b41f713/4673f/chart.webp align="center")

From the above chart, we can see that the minimum value is -1. After -1, the maximum value is 6. The total profit we get is 7. But, there's another case prior to a minimum which is 2 and 11 which gives a profit of 9. This proves that the above approach is wrong.

We need to apply kadane's algorithm, but with a slight change. Here, we need to find the maximum sub-array sum of stock price differences. Let's see how this works.

let A be the list of stock prices where A\[i\] represents stock price on the day i.

let's assume A = \[a1, a2, a3, a4\]

our aim is to maximize profit which means maximizing the difference, let B = \[a1, a2-a1, a3-a2, a4-a3\]

let's say a2 and a4 give the maximum profit, so in B we need to find the sum of (a3-a2) + (a4-a3) = a4-a2. Hence, we need to find the largest sub-array sum of price differences

```python
def max_profit(prices):
    local_max = 0
    global_max = 0
    for i in range(1, len(prices)):
        local_max += (prices[i] - prices[i-1])
        local_max = max(0, local_max)
        global_max = max(local_max, global_max)
    return global_max
```

#### Code Explanation

We can do this in two ways. First, find another array with the difference in prices and find a maximum subarray. For that, we need two iterations and another array. (or) find the difference of prices in the same iteration (refer to line no. 5 in the above code). The rest of the code is self-explanatory.