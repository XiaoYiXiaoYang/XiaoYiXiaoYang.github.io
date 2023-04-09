---
title: 链表相关
img: /medias/files/linked-list.png
summary: 链表算法的技巧
tags:
  - 博客
  - 算法
  - 链表
categories:
  - 随笔
  - 算法
top: false
cover: false
toc: true
mathjax: true
date: 2023-04-09 11:39:47
password:
---

# 经典题目

| 题目                                                                               |
|:-------------------------------------------------------------------------------- |
| [合并两个有序列表](https://leetcode.cn/problems/merge-two-sorted-lists/)                 |
| [合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)                 |
| [分割链表](https://leetcode.cn/problems/partition-list/)                             |
| [删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/) |
| [链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)               |
| [环形链表](https://leetcode.cn/problems/linked-list-cycle/)                          |
| [相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)           |

### 合并两个有序链表

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    // 虚拟头节点
    head := &ListNode{}
    p := head
    for ;list1 != nil && list2 != nil; {
        // 比较值的大小，并赋值到新的链表
        if list1.Val < list2.Val {
            p.Next = list1
            list1 = list1.Next
            // 不用断开list1 和 list1.Next,因为后续p.Next 指向谁就是修改谁的作用域
        } else {
            p.Next = list2
            list2 = list2.Next
        }

        // p 不断前进
        p = p.Next
    }

    // list1 还没遍历完
    if list1 != nil {
        p.Next = list1
    }
    // list2 还没遍历完
    if list2 != nil {
        p.Next = list2
    }

    return head.Next
}
```

### 合并 K 个升序链表

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeKLists(lists []*ListNode) *ListNode {
    // 和合并两个有序数组类似，本次使用方法1
    // 方法1 类似归并排序，两两合并，最后合并成一个
    // 方法2 顺序合并，n-1此合并
    var merge func(lists []*ListNode, left, right int) *ListNode
    merge = func(lists []*ListNode, left, right int) *ListNode{
        // 终止条件
        if left == right {
            return lists[left]
        }
        if left > right {
            return nil
        }

        mid := (left + right) / 2
        l1 := merge(lists, left, mid)
        l2 := merge(lists, mid+1, right)

        return mergeTwoLists(l1, l2)
    }

    return merge(lists, 0, len(lists)-1)
}

func mergeTwoLists(l1, l2 *ListNode)*ListNode {
    head := &ListNode{}
    p := head
    for ;l1 != nil && l2 != nil; {
        if l1.Val < l2.Val {
            p.Next = l1
            l1 = l1.Next
        } else {
            p.Next = l2
            l2 = l2.Next
        }

        // p向前进
        p = p.Next
    }

    // 当l1还没结束
    if l1 != nil {
        p.Next = l1
    }
    // 当l2还没结束
    if l2 != nil {
        p.Next = l2
    }

    return head.Next
}
```

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeKLists(lists []*ListNode) *ListNode {
    // 和合并两个有序数组类似，本次使用方法1
    // 方法1 类似归并排序，两两合并，最后合并成一个
    // 方法2 顺序合并，n-1此合并
    var merge func(lists []*ListNode, left, right int) *ListNode
    merge = func(lists []*ListNode, left, right int) *ListNode{
        // 终止条件
        if left == right {
            return lists[left]
        }
        if left > right {
            return nil
        }

        mid := (left + right) / 2
        l1 := merge(lists, left, mid)
        l2 := merge(lists, mid+1, right)

        return mergeTwoLists(l1, l2)
    }

    return merge(lists, 0, len(lists)-1)
}

func mergeTwoLists(l1, l2 *ListNode)*ListNode {
    head := &ListNode{}
    p := head
    for ;l1 != nil && l2 != nil; {
        if l1.Val < l2.Val {
            p.Next = l1
            l1 = l1.Next
        } else {
            p.Next = l2
            l2 = l2.Next
        }

        // p向前进
        p = p.Next
    }

    // 当l1还没结束
    if l1 != nil {
        p.Next = l1
    }
    // 当l2还没结束
    if l2 != nil {
        p.Next = l2
    }

    return head.Next
}
```

### 分割链表

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func partition(head *ListNode, x int) *ListNode {
    // 分割成两个链表，一个链表小于x 一个链表大于x
    head1 := &ListNode{}
    p1 := head1
    head2 := &ListNode{}
    p2 := head2

    for ;head != nil; {
        if head.Val < x {
            p1.Next = head
            p1 = p1.Next
        } else {
            p2.Next = head
            p2 = p2.Next
        }

        // 断开head head.Next两个节点的连接
        // 因为p1 p2是新构造出来的节点，所以需要断开head和head.Next
        tmp := head.Next
        head.Next = nil
        head = tmp
    }

    // 两个链表接起来
    p1.Next = head2.Next
    return head1.Next
}
```

### 删除链表的倒数第 N 个结点

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    // 找到倒数第n + 1个节点
    // 防止删掉头节点时越界操作，要虚拟出来一个节点
    newHead := &ListNode{}
    newHead.Next = head
    p1 := newHead
    p2 := newHead
    for ; p1 != nil && n >= 0; {
        p1 = p1.Next
        n--
    }

    for ; p1 != nil; {
        p1 = p1.Next
        p2 = p2.Next
    }

    // 删p2.Next
    tmp := p2.Next
    p2.Next = p2.Next.Next
    tmp.Next = nil

    return newHead.Next
}
```

### 链表的中间结点

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func middleNode(head *ListNode) *ListNode {
    // 双指针技巧 第一个指针走2步 第二个指针走1步
    p1 := head
    p2 := head
    for ; p1 != nil && p1.Next != nil; {
        p1 = p1.Next.Next
        p2 = p2.Next
    }

    return p2
}
```

### 环形链表

```go
    /**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func hasCycle(head *ListNode) bool {
    // 用hash法做一下
    nodes := map[*ListNode]bool{}
    for head != nil {
        if _, ok := nodes[head]; !ok {
            nodes[head] = true
            head = head.Next
        } else {
            return true
        }
    }

    return false
}
```

### 相交链表

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    // 双指针法，让两个链表接起来，则相遇的时候就是交点
    p1 := headA
    p2 := headB
    for ;p1 != p2; {
        if p1 == nil {
            p1 = headB
        } else {
            p1 = p1.Next
        }

        if p2 == nil {
            p2 = headA
        } else {
            p2 = p2.Next
        }
    }

    // 相交点
    return p1
}
```
