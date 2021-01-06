---
title: LeetCode-1 两数之和
date: 2019-12-30 19:51:09
tags: [LeetCode]
categories: [学习笔记]
---



<!-- LeetCode是什么？我不知道 第一道题目-->



英文版   https://leetcode.com/problemset/all/

中文版   https://leetcode-cn.com/problemset/all/


有github账户  先同步一下自己的账户

#### Two sum



给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]



```
   vector<int> twoSum(vector<int>& nums, int target) {

        vector<int> res;
        for(int i=0;i<nums.size()-1;i++){
            for(int j=i+1;j<nums.size();j++){
                if(nums[i]+nums[j]==target){
                   res.push_back(i);
                   res.push_back(j);
                    return res;
                }
            }
        }
        return res;
    }

输入：[0,2,7,8]  target=9   
返回：[1,2]    //数组下标
```

注意了注意了 
i循环从 0 ---- nums.size()-1
j循环从 i+1----nums.size()


分析一下其他情况

**1.**

i循环从 0 ---- nums.size()
j循环从 0----nums.size()

```
错误答案
输入[3,2,4]  target=6
返回 [0,0]   //3+3确实等于6
```

从上面就可以看出，只要i定了，j循环的时候不能等于j

顺便发现其他有趣思路

```
1.
return {i,j};  //列表初始化返回vector
```

优化：

map

hash表