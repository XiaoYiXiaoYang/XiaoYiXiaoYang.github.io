---
title: LeetCode-34 在排序数组中查找第一个和最后一个位置
date: 2019-12-30 20:14:23
tags: [LeetCode]
categories: [学习笔记]
---

依然是二分法思想

<!--more-->

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

你的算法时间复杂度必须是 O(log n) 级别。

如果数组中不存在目标值，返回 [-1, -1]。



**分析**：通过二分法查找，在命中一个target后，顺序向前和向后扫描target





```c++
vector<int> searchRange(vector<int>& nums, int target) 
{
	vector<int> res(2,-1);  //两个结果
	
	int first = 0;
	int end = nums.size() - 1;

	while (first <= end)
	{
		int mid = (first + end) / 2;  //取中间
		if (nums[mid] == target) /*命中*/
		{
			int i = mid;
			while (i < nums.size() && nums[i] == target) /*i范围的判断放左边*/
			{
				i++;  /*找到target右边界*/
			}
			res[1] = i - 1;

			i = mid;
			while (i >= 0 && nums[i] == target)			/*i范围的判断放左边*/
			{
				i--;/*找到target左边界*/
			}
			res[0] = i + 1;
			break; /*全都找到 返回*/
		}
		else if(nums[mid] < target)
		{
			first = mid + 1;
		}
		else if (nums[mid] > target)
		{
			end = mid - 1;
		}
		else 
		{

		}
	}
	return res;
}
```



