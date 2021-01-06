---
title: 算法入门----（蛮力法优化）
date: 2019-12-15 20:01:09
tags: [算法]
categories: [学习笔记]
---

 蛮力法固然思路简单，但是容易超出范围，需要优化

<!--more-->

##### 使用蛮力法时要特别注意带求解问题的规模，稍不注意程序就很可能会超出规定的时间限制，导致报错。
- 常见的就是百鸡问题，升级为千鸡问题，则规模就会太大。 可通过问题的条件约束求解空间，缩小可能解集合的空间。

- 找密码 密码是一个5位数，密码是81和91的倍数，密码的中间一位是1
**分析：**ab1cd  可通过暴力搜索，a为1-9，剩下几个数0-9，满足81,91倍数即可。
**优化：** 暴力搜索还是繁琐，81,91的倍数转换为7371的倍数，直接遍历5位数以内7371的倍数是否中间位为1

```
int password;
int n =99999/7371;
for(int i =0;i < n;i ++){
	password = n*i;
	if(password/100%10 ==1){
		cout<<password<<endl;
	}
}
```
- 判断N个整数I的表，判断奇偶性， N为1-100，I为1到10的60次方
**分析：** 因为I数的范围大，所以考虑通过字符串，判断奇偶只通过末位数字即可

```
char a[65]; int n;
cin>>n；
while(n--){
	cin>>a;
	if((a[strlen(a)-1]-'0')%2 == 0)
		cout<<""<<endl;
	else
		cout<<""endl;
}
```
*只判断数组a的最后一位，并将字符运算减0，得到数字值*

- 最大连续子序列和
最大连续子序列是所有连续子序列总元素和最大的一个。
1. 暴力
直接遍历所有子数组，比较每一个子数组的和即得到最大的子数组和

```
long maxSubSum(const vector<int> &a){
	long maxSum=0;
	for(int i =0;i<a.size();i++){
		for(int j =0;j<a.size();j++){
			long thisSum=0;
			for(int k=i;k <= j;k ++){
				thisSum+=a[k];
			}
			if(thisSum > maxSum){
				maxSum = thisSum;
			}
		}
	}
	return maxSum;
}
```

上面的算法时间复杂度高达O(n^3)，但其实我们会发现，这其中i到j即可扫描所有子序列，k的循环每次又重新加了j，所以k的循环没必要

**优化**
如果序列为ABCD，上述找法为AB 、AC、AD、、BC、BD、CD
可以优化为AB、AC、AD、BC、BD、CD

```
long maxSubSum(const vector<int> &a){
	long maxSum=0;
	for(int i =0;i<a.size();i++){
			long thisSum=0;
			for(int j=i;j < a.size();j ++){
				thisSum+=a[j];
				if(thisSum > maxSum){
				maxSum = thisSum;
			}
			}
		}
	}
	return maxSum;
}
```

这时候复杂度已经优化到了O(n^2)
自然而然要想到还能优化到O(nlogn)
怎么做呢，使用分而治之的思想，将数组二分为左右两个子序列，递归的计算左右子序列的最大和以及跨越中间的子序列和，最后找出最大子序列和

```
//分而治之
```


更甚者，使用动态规划算法，还能将其优化到O(n)

```
动态规划
```

另辟蹊径，还有一个O(n)算法

```

```

https://www.cnblogs.com/conw/p/5896155.html


蛮力法还是比较好理解，但是在实际写算法时候，自习读题，当算法复杂度过大的时候，必要时可进行优化。

*2019-03-05 14:49:26 星期二
萧逸小杨*