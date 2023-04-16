---
title: 动态规划算法详解
img: /medias/files/algorithm-dp.jpg
summary: 动态规划三要素：重叠子问题、最优子结构、状态转移方程
tags:
  - 博客
  - 算法
  - 动态规划
categories:
  - 随笔
  - 算法
top: false
cover: false
toc: true
mathjax: true
date: 2022-11-27 15:32:05
password:
---

动态规划的核心问题是穷举，因为要求最值，肯定要把所有可行的答案都穷举出来，然后再其中找最值。

首先，动态规划的问题存在**重叠子问题**，如果暴力穷举，效率会极其低下，所以需要“备忘录”或“DP table”来优化穷举过程。
其次，动态规划的问题一定会具备**最优子结构**，这样才能通过子问题的最值得到原问题的最值
最后，虽然动态规划的核心思想就是穷举求最值，但是问题可以千变万化，穷举所有可行解并不是一件容易的事情，只有列出正确的**状态转移方程**，才能正确的穷举。

核心套路：状态、选择、dp数组的定义

```
# 初始化base case
dp[0][0][...] = base case

# 进行状态转移
for 状态1 in 状态1的所有取值：
  for 状态2 in 状态2的所有取值：
      for ...
        dp[状态1][状态2][...] = 求最值(选择1， 选择2, ...)
```

# 经典题目

## 斐波那契数列

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项（即 F(N)）。斐波那契数列的定义如下：
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.

示例 1：
输入：n = 2
输出：1

示例 2：
输入：n = 5
输出：5

### 解答

#### 递归：定义dp[i]为f(i)

```go
func fib(n int) int {
    dp := make([]int, n+1)
    dp[0] = 0
    dp[1] = 1

    return hepler(dp, n)
}

func hepler(dp []int, n int)int {
    if n ==  0 {
        return 0
    }
    if dp[n] != 0 {
        return dp[n]
    }
    // 递归计算
    dp[n] = hepler(dp, n-1) + hepler(dp, n-2)
    return dp[n]
}
```

#### dp table：定义dp[i]为f(i)

```go
func fib(n int) int {
    if n == 0 {
        return 0
    }
    if n == 1{
        return 1
    }

    dp := make([]int, n+1)
    // 初始化base case
    dp[0] = 0
    dp[1] = 1

    // 状态转移
    for i := 2; i <= n; i++{
        dp[i] = dp[i-1] + dp[i-2]
    }

    return dp[n]
}
```

## 凑零钱

给你一个整数数组 coins ，表示不同面额的硬币；以及一个整数 amount ，表示总金额。
计算并返回可以凑成总金额所需的 最少的硬币个数 。如果没有任何一种硬币组合能组成总金额，返回 -1 。
你可以认为每种硬币的数量是无限的。

示例 1：
输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1

示例 2：
输入：coins = [2], amount = 3
输出：-1

示例 3：
输入：coins = [1], amount = 0
输出：0

### 解答

#### 递归，dp[i] = value  凑i需要最少value个硬币

```go
func coinChange(coins []int, amount int) int {
    // 初始化dp数组  dp[i] = value  凑i需要最少value个硬币
    dp := make([]int, amount + 1)
    dp[0] = 0

    return hepler(dp, coins, amount)
}

func hepler(dp, coins []int, amount int) int{
    // 是否已经计算过dp[amount]
    if amount == 0 {
        return 0
    }
    if amount < 0 {
        return -1
    }
    if dp[amount] != 0{
        return dp[amount]
    }

    // 递归计算 dp[n] = min(dp(n-coin) + 1)
    res := 100000
    for _, coin := range coins {
        subproblem := hepler(dp, coins, amount - coin)

        // 子问题无解 跳过
        if subproblem < 0 {
            continue
        }

        if subproblem < res {
            res = 1 + subproblem
        }
    }
    if res != 100000 {
        dp[amount] = res
    } else {
        dp[amount] = -1
    }

    return dp[amount]
}
```

#### dp table，dp[i] = value  凑i需要最少value个硬币

```go
func coinChange(coins []int, amount int) int {
    if amount == 0 {
        return 0
    }
    if amount < 0 {
        return -1
    }
    // 初始化dp数组  dp[i] = value  凑i需要最少value个硬币
    dp := make([]int, amount + 1)
    dp[0] = 0

    for i := 1; i <= amount; i ++ {
        // 初始化为很大
        dp[i] = 10001
        for _, coin := range coins {
            if i - coin < 0 {
                // 给的数组不一定是递增的，后面的还要测
                continue
            }
            dp[i] = min(dp[i], dp[i - coin] + 1)
        }
    }

    if dp[amount] == 10001 {
        return -1
    }

    return dp[amount]
}

func min(a, b int) int {
    if a < b {
        return a
    }

    return b
}
```

## 最长递增子序列

给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

示例 1：
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。

示例 2：
输入：nums = [0,1,0,3,2,3]
输出：4

示例 3：
输入：nums = [7,7,7,7,7,7,7]
输出：1

### 解答

#### dp table，dp[i]为表示以nums[i]为结尾的数组的最长递增子序列的长度

```go
func lengthOfLIS(nums []int) int {
    // dp[i] 表示以nums[i]为结尾的数组的最长递增子序列的长度
    // 那么 dp[i] = 遍历小于i的dp[i-1]，当nums[i] > nums[i-1]  就加1
    dp := make([]int, len(nums))
    // 初始化数组的最大子序列长度
    ret := -1
    for i := 0; i < len(nums); i ++ {
        // 初始化dp 每个位置 最长子序列至少是1
        dp[i] = 1
        // 计算dp[i]
        for j := 0; j < i; j ++ {
            if nums[i] > nums[j] {
                dp[i] = max(dp[i], dp[j] + 1)
            }
        }
        ret = max(ret, dp[i])
    }
    return ret
}

func max(a, b int) int{
    if a > b {
        return a
    }
    return b
}
```

## 最大子数组

给你一个整数数组 nums ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
子数组 是数组中的一个连续部分。

示例 1：
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。

示例 2：
输入：nums = [1]
输出：1

示例 3：
输入：nums = [5,4,-1,7,8]
输出：23

### 解答

#### dp table, dp[i]为以nums[i]结尾的最大子数组和

```go
func maxSubArray(nums []int) int {
    if len(nums) <= 0 {
        return -1
    }
    // 定义dp数组，dp[i]为以nums[i]结尾的最大子数组和
    dp := make([]int, len(nums))
    dp[0] = nums[0]
    ret := dp[0]

    // 状态转移
    for i := 1; i < len(nums); i ++ {
        dp[i] = max(nums[i], dp[i-1] + nums[i])
        ret = max(ret, dp[i])  
    }

    return ret
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

## 最长公共子序列

给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。
一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

示例 1：
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace" ，它的长度为 3 。

示例 2：
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc" ，它的长度为 3 。

示例 3：
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0 。

### 解答

#### dp table，dp[i][j]为str1[0:i]和str2[0:j]最长公共子序列的长度

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    if len(text1) == 0 || len(text2) == 0 {
        return 0
    }
    // dp[i][j] 为text1[0:i] 和 text2[0:j]的最长公共自序列
    // 默认初始化零
    dp := make([][]int, (len(text1)+1))
    for i := range dp {
        dp[i] = make([]int, (len(text2)+1))
    }

    ret := -1
    // 状态转移， 用dp[i+1][j+1]来替换dp[i][j]
    for i := 0; i < len(text1); i ++ {
        for j := 0; j < len(text2); j ++ {
            if text1[i] == text2[j] {
                // 相同 则等于text1[0:i] 和 text2[0:j]的公共子序列长度+1
                dp[i+1][j+1] = dp[i][j] + 1
            }else {
                // 不相同则等于text1[0:i-1] 和 text2[0:j]中的公共子序列长度
                // 与 text1[0:i] 和 text2[0:j-1]公共子序列长度较大的
                dp[i+1][j+1] = max(dp[i+1][j], dp[i][j+1])
            }
            ret = max(ret, dp[i+1][j+1])
       }
    }
    return ret
}

func max(a, b int) int{
    if a > b {
        return a
    }
    return b
}
```

## 编辑距离

## 以插入最小次数构造回文串
