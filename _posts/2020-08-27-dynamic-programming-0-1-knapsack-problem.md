---
title: "Dynamic Programming: 0-1 Knapsack problem"
date: 2020-08-27 10:00:00 +07:00
tags: [programming, algorithms, computer science]
description: How to solve the 0-1 knapsack problem using dynamic programming
---

The knapsack problem is a combinatorial problem that can be optimized by using dynamic programming. 
<b>Problem</b>: given a set of `n` items with set of `n` cost, `n` weights for each item. You're given a knapsack that can carry a fixed value of weight find the combination of items that maximizes the cost of items to put in the knapsack that the total weight does not surpass the maximum capacity of the knapsack.

## Problem
So here's an example of the problem given above. Take a scenario where you're given a backpack that can carry about `100 kg` of items and you are given five items to sell with price in dollars `120, 200, 150, 350, 100, 90` and weights for each item in kg as `15, 20, 40, 50, 20, 10` respectively. We want to know the maximum price we can get by selecting some items without surpassing the capacity. So our data structure will look like this.
```python
values = [120, 200, 150, 350, 100, 90]
weights = [15, 20, 40, 50, 20, 10]
capacity = 100
n = len(values)
```

## Naive Method
We are first going to solve this step by step from the naive approach then build-up to the bottom-up approach. So in the naive approach, we solve it with recursion. For each item, if the weight of the current item is less than or equal to the current capacity we take two combinations, first when we don't put the item in the knapsack and the second when we put the item and get the maximum of the two outcomes and return it.
```python
def knapsack_naive(values, weights, capacity, n):
    if n == -1 or capacity == 0:
        return 0
    if weights[n] <= capacity:
        no_insert = knapsack_naive(values, weights, capacity, n-1)
        insert = values[n] + knapsack_naive(values, weights, capacity - weights[n], n-1)
        max_sack = max(no_insert, insert)
        return  max_sack
    else:
        return knapsack_naive(values, weights, capacity, n-1) 

print(knapsack_naive(values, weights, capacity, n-1))

```
### Output
```
760
```
So as shown above our solution works, because if we analyze the items by our self we see that the maximum value we can get without surpassing the knapsack limit is the combination of items of index `0, 1, 3, 5` which corresponds with values `120, 200, 350, 90` respectively and sums up to `760`, while the sum of the weight `15, 20, 50, 10` adds up to `95` which does not surpass our knapsack capacity `100`. So we've got our solution but there's a problem our solution time complexity increases by O(2^n) which is not good. We have to find a way to reduce the time complexity, that's where dynamic programming comes in play.

## Memoization Method
In this method we reduce computation by saving computed values and reusing them in the future, instead of computing them over and over again. We create an array of `n` rows where `n` is the size of the items and `capacity+1` columns to hold our computed value for each `n` and `capacity` step. 
```python
memo = [[None for _ in range(capacity+1)] for _ in range(n)]
def knapsack_memorize(values, weights, memo, capacity, n):
    if n == -1 or capacity == 0:
        return 0

    if memo[n][capacity] != None:
        return memo[n][capacity]

    if weights[n] <= capacity: 
        no_insert = knapsack_memorize(values, weights, memo, capacity, n-1)
        insert = values[n] + knapsack_memorize(values, weights, memo, capacity - weights[n], n-1)
        max_sack = max(no_insert, insert)
        memo[n][capacity] = max_sack
        return  max_sack
    else:
        memo[n][capacity] = knapsack_memorize(values, weights, memo, capacity, n-1)
        return memo[n][capacity]
```
Here we only change the code a little by creating a 2d array called memo to hold our computed values, after our break case we check if the value at the point `n` and `capacity` have been computed and if true we return its value and after we have computed a value we store it in the array so that it can be reused without going through the computation again. The time complexity of this method is O(n * capacity) so this is better than our previous O(2^n). If we take a case where we have `20` items and a capacity of `100 kg` using the naive method, in the worst case our function will be executed `1048576` times while using the memorization method it will be executed `2000` times which is by far better than the previous.

## Bottom-Up Method
The bottom-up approach is used to save memory cost that comes with recursion and bypass the recursion max stack. It does this by building up the array from the top to the bottom. And solving each sub-problem.
```python
def knapsack_bottom_up(values, weights, capacity):
    k = [[0 for _ in range(capacity+1)] for _ in range(len(weights)+1)]

    for i in range(len(weights)+1):
        for w in range(capacity+1):
            if i == 0 or w == 0:
                k[i][w] = 0
            elif weights[i-1] <= w:
                k[i][w] = max(values[i-1] + k[i-1][w - weights[i-1]], k[i-1][w])
            else:
                k[i][w] = k[i-1][w]
    return k[len(weights)][capacity]

print(knapsack_bottom_up(values, weights, capacity))
```
So lets break down everything above. First we create an array of capacity columns and weights length rows, this is will be the array we would build on as our knapsack. When the item index i is at 0 means we have 0 item in to put in the knapsack and the cost of 0 item is 0, so we set the score for it to 0 and also when our capacity is 0 which means we've maxed out the capacity and there's no space, we set the score for that position to 0. Then we check the weight at index `i-1`  we subtract 1 because list indexing start at 0 so for the first item 1 we subtract 1 to get index 0, but you can pad the `weights` and `values` list with an arbitrary value to match the indexing, a padding would look like this `[None,120, 200, 150, 350, 100, 90]`. We compare each weight with the value from 0 to capacity inclusive, and if the weight is less than the current capacity value we set the max cost so far at that point to the maximum of the sum of the value and the max cost of the previous value at cost weight - current cost with the previous item weight. If the weight is greater than the current cost we just set the max so far as the previous item weight. We then return the max cost so far at point `len(weights)` and `capacity`.
