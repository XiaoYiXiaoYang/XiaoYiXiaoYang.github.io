---
title: 双指针解决部分数组问题
img: /medias/files/array.png
summary: 主要汇总双指针解决数组问题的题目
tags:
  - 博客
categories:
  - 随笔
top: false
cover: false
toc: true
mathjax: true
date: 2023-04-16 11:45:38
password:
---

# 经典题目

双指针技巧是经常用到的，双指针技巧主要分为两类：**左右指针**和**快慢指针**。

| [删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)   |
| ---------------------------------------------------------------------------------- |
| [删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)   |
| [移除元素](https://leetcode.cn/problems/remove-element/)                               |
| [移动零](https://leetcode.cn/problems/move-zeroes/)                                   |
| [两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/) |
| [反转字符串](https://leetcode.cn/problems/reverse-string/)                              |
| [最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)              |

### [删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)

```go
func removeDuplicates(nums []int) int {
    // 快慢指针，遇到相同的就跳过，不同的就赋值
    if len(nums) <= 1 {
        return len(nums)
    }

    left, right := 0, 0
    for ;right <= len(nums) - 1; {
        if nums[left] != nums[right] {
            // 不相等则赋值并移动
            left ++
            nums[left] = nums[right]
            right ++  
        } else {
            // 相等则只移动右指针
            right ++
        }

    }

    // 数组长度要加1
    return left + 1
}

// 1 1 2
```

### [删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func deleteDuplicates(head *ListNode) *ListNode {
    // 快慢指针
    if head == nil {
        return head
    }

    slow, fast := head, head
    for ;fast != nil; {
        if fast.Val != slow.Val {
            // 不相等，则移动指针并赋值
            slow = slow.Next
            slow.Val = fast.Val
            fast = fast.Next
        } else {
            // 相等，则只移动快指针
            fast = fast.Next
        }
    }

    // 断开重复后面的
    slow.Next = nil
    return head
}
```

### [移除元素](https://leetcode.cn/problems/remove-element/)

```go
func removeElement(nums []int, val int) int {
        // 快慢指针
        slow , fast := 0, 0
        for ;fast <= len(nums) - 1; {
            if nums[fast] != val {
                // 不相等，则赋值再移动
                nums[slow] = nums[fast]
                slow ++
                fast ++
            } else {
                // 相等，只移动快指针
                fast ++

            }

        }

        return slow
}
```

### [移动零](https://leetcode.cn/problems/move-zeroes/)

```go
func moveZeroes(nums []int)  {
    // 双指针，由于不能改变元素顺序，不能只用左右指针，只用快慢指针
    slow, fast := 0, 0
    for ;fast < len(nums); {
        if nums[fast] != 0 {
            nums[slow] = nums[fast]
            slow ++
            fast ++
        } else {
            fast ++
        }
    }

    for ;slow < len(nums); {
        nums[slow] = 0
        slow ++
    }
}
```

### [两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)

```go
func twoSum(numbers []int, target int) []int {
    // 双指针
    ret := []int{0, 0}
    if len(numbers) < 2 {
        return ret
    }
    left, right := 0, len(numbers) - 1
    for ;left < right; {
        if numbers[left] + numbers[right] == target {
            ret[0], ret[1]= left + 1, right + 1
            break;
        } else if numbers[left] + numbers[right] < target {
            // 增加左边
            left ++
        } else if numbers[left] + numbers[right] > target {
            // 减小右边
            right --
        }
    }

    return ret
}
```

### [反转字符串](https://leetcode.cn/problems/reverse-string/)

```go
func reverseString(s []byte)  {
    left, right := 0, len(s) - 1
    for left < right {
        tmp := s[left]
        s[left] = s[right]
        s[right] = tmp

        left ++
        right --
    }
}
```

### [最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

```go
func longestPalindrome(s string) string {
    // 中心扩展法
    ret := ""
    for i, _ := range s {
        // 以s[i] 为中心的最长回文子串
        ret1 := palindrome(s, i, i)
        // 以s[i] 和s[i+1]为中心的最长回文子串
        ret2 := palindrome(s, i , i+1)

        if len(ret1) > len(ret) {
            ret = ret1
        }
        if len(ret2) > len(ret) {
            ret = ret2
        }
    }
    return ret
}

// 寻找以s[l] s[r]为中心的最长回文子串
// 适配子串是奇数或者偶数
func palindrome(s string, left, right int) string {
    for ;left >= 0 && right <= len(s) - 1 && s[left] == s[right]; {
        left --
        right ++
    }

    // left + 1 到 right
    return s[left + 1:right]
}
```
