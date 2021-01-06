---
title: LeetCode-33 搜索旋转排序数组
date: 2019-12-30 20:00:35
tags: [LeetCode]
categories: [学习笔记]
---

剑指offer我记得有一道找旋转数组的最小数

<!--more-->

题目

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。

你可以假设数组中不存在重复的元素。

你的算法时间复杂度必须是 O(log n) 级别。



**算法**：时间复i杂度限制了大概一定只能是二分法了

**分析**：主要是在二分法的过程中判断在左边还是右边的问题



参考:https://blog.csdn.net/qq_39360985/article/details/83793352



画图讲的很清楚

一、nums[mid] < target

1. nums[mid] > nums[0] 

2. 

   (1).nums[mid] < nums[0]   target < nums[0]

   (2).nums[mid] < nums[0]   target > nums[0]



二、类比

1.

2.



```c++
int search(vector<int>& nums, int target) {
	int res = -1;

	int first = 0;
	int end = nums.size() - 1;

	while (first <= end)
	{
		int mid = (first + end) / 2;
		if (nums[mid] == target)
		{
			res = mid;
			break;
		}
		else if (nums[mid] < target)
		{
			if(nums[mid] >= nums[0]) /*中间值小于目标值  且 中间值大于第一个  肯定在前一个排序数组*/
			{
				first = mid + 1;
			}
			else
			{
				if (target < nums[0]) /*中间值小于目标值  中间值小于第一个  目标值小于第一个 在后一个排序数组*/
				{
					first = mid + 1;
				}
				else               /*中间值小于目标值  中间值小于第一个  目标值大于第一个  在前一个排序数组*/
				{
					end = mid - 1;
				}
			}
		}
		else
		{
			if (nums[mid] < nums[0]) /*中间值大于目标值  且 中间值小于第一个  在后一个排序数组*/
			{
				end = mid - 1;
			}
			else
			{
				if (target < nums[0]) /*中间值大于目标值  中间值大于第一个  目标值小于第一个 在后一个排序数组*/
				{
					first = mid + 1;
				}
				else               /*中间值大于目标值  中间值大于第一个  目标值大于第一个  在前一个排序数组*/
				{
					end = mid - 1;
				}
			}
		}

	}
	return res;
}
```

