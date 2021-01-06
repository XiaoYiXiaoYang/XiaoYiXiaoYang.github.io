---
title: 数据结构（排序）
date: 2019-12-25 11:21:38
tags: [数据结构]
categories: [学习笔记]
---

 从算法设计角度看，排序算法展示了算法设计的某些重要原则和技巧，体现了重要的算法设计方法

<!--more-->

#### 排序算法的稳定性

假定在待排序的记录序列中，存在多个具有相同关键码的记录，若经过排序，这些记录的相对次序保持不变，即在原序列中，ki = kj，且ri在rj之前，在排序后的序列中，ri仍在rj之前，则称这种排序算法稳定。

否则称为不稳定。

#### 排序分类

根据排序过程中待排序的所有记录是否全部被放置在内存中，可将排序方法分为**内排序**和**外排序**两大类。

内排序是指整个排序过程中，待排序的所有记录全部都被放在内存中，

外排序是指由于待排序的记录个数太多，不能同时放在内存，需要将一部分记录放在内存，另一部分放在外存，整个排序过程需要在内外存之间多次交换数据才能得到排序的结果。

### 插入排序

**直接插入排序**

依次将待排序序列的每一个记录插入到一个已排好序的序列中，直到全部记录都排好序。

```
void InsertSort(int r[],int n){
    int i,j;
    for(i=2;i<n;i++){
        r[0]=r[i];					//r[0]暂存被插入记录
        for(j=i-1;r[0]<r[j];j--){  //比较过的后移
            r[j+1]=r[j];
        }
        r[j+1]=r[0];            //找到插入位置
    }
}

void InsertSort2(int r[],int n){
    int i,j;
    for(i=1;i<n;i++){
        int temp = r[i];
        j=i-1;
        while(j>=0 && temp<r[j] ){
            r[j+1]=r[j];  //记录后移
            j--;
        }
        r[j+1]=temp;
    }
}

void InsertSort(vector<int> &vec)
{
	int i, j;
	for (i = 1; i < vec.size(); i++)
	{
		int temp = vec[i];  //暂存待插入元素，后面的会覆盖掉
		for (j = i - 1; j >= 0 && vec[j] > temp; j--)
		{
			vec[j + 1] = vec[j];
		}
		vec[j + 1] = temp;
	}
}
```

关键在于理解每一次比较的时候将记录后移，然后找到一个合适的位置插入新的数据。

复杂度分析

最好情况：正序排列，无需插入，比较n-1次，O(n)
最坏情况：逆序排列，每次都要移动元素再插入，O(n^2)
平均情况：O(n^2)


**希尔排序**

对直接插入排序的一种改进，先将整个待排序记录序列分割成若干个子序列，在子序列内分别进行直接插入排序，待整个序列基本有序时，再对全体进行一次直接插入排序

取整数d=n/2，将所有相距为d的记录构成一组，将序列分割成d个子序列，对每个子序列分别进行直接插入排序，然后在缩小间隔d，取d=d/2，重复分割分别进行插入排序，直到最后取d=1，则将所有记录放在一组内进行排序。

```
void shellSort(int r[],int n){ 
    int d,i,j;
    for(d=n/2;d>=1;d=d/2){
        for(i=d+1;i<n;i++){
            r[0]=r[i];				//r[0]暂存记录
            for(j=i-d;j>0 && r[0]<r[j];j=j-d)
                r[j+d]=r[j];
            r[j+d]=r[0];
        }
    }
}



void ShellSort(vector<int> &vec)
{
	int i,j,d;
	for (d=vec.size()/2; d>0;d = d/2)
	{
		for (i = d; i < vec.size(); i++)
		{
			int temp = vec[i];  //暂存
			for (j = i - d; j >= 0 && vec[j] > temp; j -= d)
			{
				vec[j + d] = vec[j];
			}
			vec[j + d] = temp;
		}
	}
}
```

一个重要的思想就是前面的元素是有序的，后面的元素需要和前面的元素一一比较，满足往前插入的条件的话就要把当前比较过的元素后移，直至找到正确的插入位置

复杂度分析
最好情况：O(nlogn)
最坏情况：O(n^2)
平均情况：O(n^1.3)

这是一个不稳定的排序方法

遗留问题:增量d如何取；增量d的序列恰好有序就很浪费

### 交换排序

**冒泡排序**

两两比较相邻记录的关键码，如果反序则交换，直到没有反序为止，需要设标志位，标志每趟循环是否有交换。

将待排序记录划分为有序区和无序区
对无序区从前到后依次将相邻记录的关键码进行比较，若反序则交换，从而使得关键码小的记录向前移，关键码大的记录向后冒泡
重复上述

```
void BubleSort(int r[],int n){
    int exchange=n;   //标志是否发生交换
    while(exchange!=0){
        int  bound = exchange;
        exchange=0;
        for(int i=0;i<bound;i++){
            if(r[i]>r[i+1]){
                int t = r[i];
                r[i]=r[i+1];
                r[i+1]=t;
                exchange = i;   //下一趟循环【1,i】即可
            }
        }
    }
}


void BubleSort(vector<int> &vec)
{
	int exchange = vec.size()-1;
	while (exchange != 0)
	{
		int bound = exchange;  exchange = 0;
		for (int j = 0; j < bound; j++)
		{
			if (vec[j] > vec[j + 1])
			{
				int temp = vec[j];
				vec[j] = vec[j + 1];
				vec[j + 1] = temp;
				exchange = j;
			}
		}
	}
}
```

复杂度分析

最好情况：待排序序列为正序，算法只执行一趟，exchange=0，退出while，复杂  O(n)
最坏情况：待排序列为逆序，算法执行n-1趟，第i趟执行n-i次关键码的比较和n-i次交换，复杂度O(n^2)
平均情况：O(n^2)

需要一个记录的辅助空间，(临时变量t)，暂存交换单元
冒泡排序是一个稳定的排序方法


**快速排序**

快速排序是对冒泡排序的一种改进，着眼点是冒泡排序每次交换相邻两个位置的记录，总的比较次数和交换次数较多，快速排序中，记录的比较和移动是从两端到中间进行，记录移动距离较远。

选一个轴值，将待排序记录划分成独立的两部分，左侧记录的关键码小于或等于轴值，右侧记录的关键码大于或等于轴值，然后分别对两部分重复这个过程，直到整个序列有序。

一次划分：选取第一个记录为轴值，设置两个参数i，j，分别用来指示将要与轴值记录比较的左侧记录位置和右侧记录位置
右侧开始扫描，将轴值与下标为j的记录比较，如果下标j的记录大，则继续向前移动，如果下标j的记录小，则交换下标j的记录和轴值，进行左侧扫描
左侧扫描，交换之后开始比较下标i的记录和轴值，如果下标i的记录小，则继续向后移动，如果下标i的记录大，则交换下标i的记录和轴值，进行右侧扫描
继续扫描，直到i，j指向同一位置，一次划分结束


```
int Partition(vector<int> &vec, int first, int end)
{
	int pivot = first; //轴值就是第一个记录
	while (first < end)
	{
		while (first < end && vec[first] < vec[end])
		{
			end--;
		}
		if (first < end)
		{
			int temp = vec[first];
			vec[first] = vec[end];
			vec[end] = temp;
			first++;
		}

		while (end > first && vec[first] < vec[end])
		{
			first++;
		}
		if (first < end)
		{
			int temp = vec[first];
			vec[first] = vec[end];
			vec[end] = temp;
			end--;
		}
	}
	pivot = first;
	return pivot;
}

void QuickSort(vector<int> & vec,int first,int end)
{
	if (first < end)
	{
		int pivot = Partition(vec, first, end);
		QuickSort(vec,first,pivot-1);  //左侧
		QuickSort(vec,pivot+1,end);//右侧
	}
}
```


复杂度分析

最好情况：每次划分之后，该记录的左侧子序列和右侧子序列的长度相同，时间复杂度是O(nlogn)
最坏情况：待排序序列为正序或逆序，每次划分只得到一个比上一次划分少一个记录的子序列，另一个子序列为空，此时必须经过n-1次递归调用才能把所有记录定位，其时间复杂度O(n^2)
平均情况：O(nlogn)

快速排序是一种不稳定的排序方法，
快速排序适用于待排序记录个数很大且原始记录随机排列的情况，快速排序的平均性能是迄今为止所有内排序算法中最好的一种



### 选择排序

**简单选择排序**

在第i趟排序中，选取关键码最小的记录，并且和第i个有序记录交换作为有序序列的第i个记录

初始时有序区为空，无序区满，然后从无序区中寻找最小的关键码和第i个有序的进行交换

```
void SelectSort(vector<int> &vec)
{
	for (int i = 0; i < vec.size(); i++)
	{
		int index = -1;
		int minNum = 9999;
		for (int j = i; j < vec.size();j++)
		{
			if (vec[j] < minNum)
			{
				index = j;
				minNum = vec[j];
			}
		}
		if (index!=-1)
		{
			int temp = vec[i];
			vec[i] = vec[index];
			vec[index] = temp;
		}
	}
}
```

复杂度分析

最好情况：关键码比较次数不变O(n^2)
最坏情况：O(n^2)
平均情况：O(n^2)

选择排序是一种不稳定的排序方法，

在简单选择排序过程中，只需要一个用来作为记录的暂存单元


**堆排序**

是简单选择排序的一种改进，减少关键码的比较次数，将每趟比较的结果存下来，免得每趟都要比较

堆是完全二叉树

小根堆定义：每个节点的值都小于或者等于其左右孩子结点的值

在调整堆的过程中，总是将根节点与左右孩子结点进行比较，若不满足堆的条件，则将根节点与左右孩子的较大者进行交换，这个调整过程一直进行到所有子树均为对或将被调整结点交换到叶子为止。

```
void Sift(vector<int> &vec,int k,int size)
{
	int leftchild = 2 * k+1, rightchild = 2*k +2,largest = leftchild;
	while (leftchild < size)  /*筛选还没到叶子结点*/
	{
		if (rightchild < size && vec[leftchild] < vec[rightchild])
		{
			largest = rightchild;
		}
		if(vec[k] > vec[largest])
		{
			break;  /*根节点已经大于左右孩子中的较大者*/
		}
		else 
		{
			int temp = vec[k];
			vec[k] = vec[largest];
			vec[largest] = temp;
			k = largest;
			leftchild = 2 * k+1;
			rightchild = 2 * k + 2;
			largest = leftchild;

		}
	}
}

void heapSort(vector<int> &vec)
{
	for (int i = vec.size() / 2; i >= 1; i--)
	{
		Sift(vec, i-1,vec.size());
	}


	for (int i = 0; i < vec.size(); i++)
	{
		int temp = vec[0];
		vec[0] = vec[vec.size() - 1 - i];
		vec[vec.size() - 1 - i] = temp;
		Sift(vec, 0,vec.size()-1-i);   /*终止的叶子大小在变化*/
	}
}

```


复杂度分析

最好情况：堆排序的运行时间主要消耗在初始建堆和重建堆进行的反复筛选上，初始建堆需要O(n)时间，第i次取堆顶记录重建堆需要用O(logi)时间，并且需要取n-1次堆顶记录，总得到时间复杂度O(nlogn)
最坏情况：O(nlogn)
平均情况：O(nlogn)

堆排序是一种不稳定的排序方法，

在堆排序过程中，只需要一个用来作为记录的暂存单元


### 归并排序

**二路归并**

将若干个有序序列两两归并，最终归并成为一个有序序列

将n个待排序记录看成n个长度为1的有序序列

归并一次变成 n/2个 长度为2的有序序列

再归并一次变成 n/2/2个 长度为4的有序序列...

直至得到 1个 长度为n的有序序列


```
void Merge(vector<int> &vec, int begin, int mid, int end)
{
	vector<int> vecDest;
	int i = begin,j =mid + 1;  
	while (i <= mid && j <= end)
	{
		if (vec[i] <= vec[j])
		{
			vecDest.push_back(vec[i]);
			i++;
		}
		else
		{
			vecDest.push_back(vec[j]);
			j++;
		}
	}
	if (i <= mid)  /*第一个子序列没有处理完*/
	{
		while (i <= mid)
		{
			vecDest.push_back(vec[i]);
			i++;
		}
	}
	else
	{
		while (j <= end)
		{
			vecDest.push_back(vec[j]);
			j++;
		}
	}

	for (int i = 0; i < vecDest.size(); i++)
	{
		vec[begin + i] = vecDest[i];
	}

}


void MergeSort(vector<int> &vec,int begin,int end)
{
	if (vec.empty() || begin >= end)
	{
		return;
	}

	int mid = (begin + end) / 2;
	MergeSort(vec, begin, mid);
	MergeSort(vec,mid+1, end);
	Merge(vec, begin, mid,end);  
}

```

复杂度分析

最好情况：一趟归并排序需要将待排序序列扫描一遍，时间性能O(n),整个归并排序logn趟，则总的时间复杂度O(nlogn)
最坏情况：O(nlogn)
平均情况：O(nlogn)

归并排序是一种稳定的排序方法，

在归并排序过程中，需要与待排序序列同样数量的存储空间，以便存放归并结果，其空间复杂度O(n)

**q一个点，书上给的代码执行会失败的 36,30,40,18**

```
分裂下去 
MergeSort  vec2  36
MergeSort  vec2  30
Merge      vec   30 36

MergeSort  vec2  40
MergeSort  vec2  18
Merge      vec   18 40

MergeSort  vec2  36 30
MergeSort  vec2  40 18
Merge      vec   结果错误
```


```
	MergeSort(vec, vec2,begin, mid);
	MergeSort(vec,vec2,mid+1, end);
	Merge(vec2, vec,begin, mid,end); 
```

### 分配排序

**桶排序**

假设待排序记录的值都在0~m-1之间，设置m个桶，首先将值为i的记录分配到第i个桶中，然后再将桶中的记录依次收集起来。

书上：

每个桶可能有有相同的记录，为了保证排序的稳定性，可以用m个链队列表示
可以采用静态链表存储桶

简化：
直接用数组存储，有一个数就把数组对应下标处+1
输出的时候，对应数组值是多少，就把数组下标输出几次

分配：把每个元素分配到对应的桶里
收集：把所有元素收集起来作为排序结果

```
void BucketSort(vector<int> &vec)
{
	int maxSize = *max_element(vec.begin(), vec.end());
	vector<int> vecTmp(maxSize+1);

	for (int i = 0; i < vec.size(); i++)
	{
		vecTmp[vec[i]]++;
	}

	
	for (int i = 0,j=0; i < vecTmp.size(); i++) /*对m个桶*/
	{
		for (int k = 0; k < vecTmp[i]; k++)  /*每个桶不止一个元素*/
		{
			vec[j] = i;
			j++;
		}
	}
}
```

复杂度分析

最好情况：桶排序第一个循环初始化桶，进行分配O(n)，进行收集O(n+m)，则总的时间复杂度O(n+m)

最坏情况：O(n+m)
平均情况：O(n+m)

桶排序如果采用队列作为桶内的存储结构，则是一种稳定的排序方法，
而上述使用了数组下标存储相同键值的个数，没有先后次序，所以是不稳定的排序方法

在桶排序过程中，需要m个桶存储，其空间复杂度O(m)



**基数排序**

将关键码看成由若干个关键码复合而成，然后借助由分配和收集操作完成排序

如：扑克牌
花色：红桃 < 梅花 < 黑桃 < 方块
点数：1 < 2 < 3 <4 < 5... < K

且花色优先级高，则对扑克牌排序时，可以先按花色分成四堆，每堆里具有相同花色的按点数排序，然后按花色将4堆整理在一块。也可以先按点数分成13堆，每堆里按花色排序，再收集。

最主位优先（MSD）：按照优先级最高的划分，再分别对下一优先级的划分排序

最次位优先（LSD）:按照优先级最低的划分，再分别对上一优先级的划分排序


36,30,18,40,32,45,22,50 

对个位进行排序:30,40,50,22,32,45,36,18
对十位进行排序:18,22,30,32,36,40,45,50

```
void RadixSort(vector<int> &vec)
{
	vector<int> vecCount(10);  /*个位0-9 存储个位数的次数*/
	vector<int> vecTmp(vec.size());

	int bits = maxbits(vec);
	int radix = 1;
	for (int i = 0; i < bits; i++)
	{
		/*每次清零*/
		for (int j = 0; j < vecCount.size(); j++)
		{
			vecCount[j] = 0;
		}

		/*每次分配*/
		for (int j = 0; j < vec.size(); j++)
		{
			int k =(vec[j]/radix)% 10;
			vecCount[k]++;
		}

		/*计算累加频数*/
		for (int j = 1; j < 10; j++)
		{
			vecCount[j] = vecCount[j - 1] + vecCount[j];
		}

		/*每次收集*/
		for (int j = vec.size() - 1; j >= 0; j--) 
		{
			int k = (vec[j] / radix) % 10;
			vecTmp[vecCount[k] - 1] = vec[j];
			vecCount[k]--;
		}

		/*赋值回原数组*/
		for (int j = 0; j < vecTmp.size(); j++)
		{
			vec[j] = vecTmp[j];
		}
		radix *= 10;
	}
}
```


书上的不好懂，还是用了别的简单的算法

复杂度分析

最好情况：假设待排序序列由d个子关键码组成，每个关键码组成为m个，每一趟分配的时间复杂度O(n)，每一趟收集的时间复杂度为O(n+m)，整个排序需要执行d趟，时间复杂度为O(d(n+m))

最坏情况：O(d(n+m))
平均情况：O(d(n+m))

基数排序不改变关键码前后次序，所以是一种稳定的排序方法

在基数排序过程中，需要m个桶存储，其空间复杂度O(m)



| 算法         | 稳定/不稳定 |
| ------------ | ----------- |
| 直接插入排序 | 稳定        |
| 希尔排序     | 不稳定      |
| 冒泡排序     | 稳定        |
| 快速排序     | 不稳定      |
| 选择排序     | 不稳定      |
| 堆排序       | 不稳定      |
| 归并排序     | 稳定        |
| 基数排序     | 稳定        |


综上：
稳定：直接插入排序、冒泡排序、归并排序、基数排序
不稳定：希尔排序、快速排序、选择排序、堆排序

| 排序方法     | 最好情况  | 最坏情况  | 平均情况          | 空间复杂度   |
| ------------ | --------- | --------- | ----------------- | ------------ |
| 直接插入排序 | O(n)      | O(n^2)    | O(n^2)            | O(1)         |
| 希尔排序     | O(n^1.3)  | O(n^2)    | O(nlogn) ~ O(n^2) | O(1)         |
| 冒泡排序     | O(n)      | O(n^2)    | O(n^2)            | O(1)         |
| 快速排序     | O(nlogn)  | O(n^2)    | O(nlogn)          | o(logn~O(n)) |
| 选择排序     | O(n^2)    | O(n^2)    | O(n^2)            | O(1)         |
| 堆排序       | O(nlogn)  | O(nlogn)  | O(nlogn)          | O(1)         |
| 归并排序     | O(nlogn)  | O(nlogn)  | O(nlogn)          | O(n)         |
| 基数排序     | O(d(n+m)) | O(d(n+m)) | O(d(n+m))         | O(m)         |





2019-10-19 22:51:05 星期六