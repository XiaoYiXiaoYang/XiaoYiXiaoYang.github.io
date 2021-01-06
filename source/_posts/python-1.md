---
title: Python基础语法
date: 2020-07-13 22:24:57
tags: [Python]
categories: [学习笔记]
---



人生苦短，我学Python

<!--more-->



import this  展示 Python之禅



```python
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```



变量名

只能包含字母、数字、下划线，可以字母或下划线打头

不能包含空格、但可以使用下划线来分割其中的单词

不要将Python关键字和函数名用作变量名



### 字符串

单引号、双引号

name = "aabbC"

```
name.title()         //AabbC
name.upper()     	 //AABBC
name.lower()      	 //aabbc
name + "212"    	 //aabbC212
```



```
name = "  aa bb  "
name.lstrip()        //'aa bb  '
name.rstrip() 		 //'  aa bb'
name.strip()         //'aa bb'

a = 5
name + str(a)       //'  aa bb  5'
```



### 列表

nums = [1,2,3]



```
nums[0]        //1
nums[-1]       //3
```



**修改**

```
nums[0] = 0   //nums为 0 2 3
```



**添加**

```
nums.append(4)   //nums为0 2 3 4
nums.insert(1,1)  //nums为0 1 2 3 4
```



**删除**



```
del nums[0]     //nums为1 2 3 4
nums.pop()      //nums为1 2 3
nums.remove(1)  //nums为2 3
nums.pop(0)  //nums为3
```



**排序**

```
nums.sort()                //nums 为2 3
nums.sort(reverse = True)  //nums为3 2
sorted(nums)               //nums为2 3   保留列表元素原来的排列顺序，同时以特定的顺序呈现它们
nums.reverse()             //nums为3 2
len(nums)                  // 2    
```



使用列表时避免索引错误，如果索引错误，会报错



**操作列表**

```
for num in nums:
        print(num)
```



**创建列表**

``` 
list(range(1,6))   //[1,2,3,4,5]
list(range(1,5,2)) //[1,3]   2是步长

nums = [1,2,3,4,5]
min(nums)    //1
max(nums)    //5
sum(nums)    //15

squares = [value**2 for value in range(1,5)]    //squares为 [1,4,9,16]
```



**切片**

```
nums = [1,2,3,4,5]
nums[0:3]   //[1,2,3]
nums[:3]    //[1,2,3]
nums[1:]    //[2,3,4,5]
nums[-3:]   //[3,4,5]
for num in nums[:3]:
	...
```



**复制**

```
nums = [1,2,3]
new_nums = nums[:]  //new_nums为[1,2,3]
```



**赋值**

```
nums = [1,2,3]
new_nums = nums    //new_nums 和num都指向 [1,2,3]
```



### 元组

```
nums = (1,2,3)
nums[1]   //2   元组的值不可被修改

但是nums = (4,5,6)
```



### 字典



```python
ytt = {'color':'red','height':180}

ytt['color']    		//访问
ytt['color'] = 'green'	//修改
ytt['weight'] = 55      //添加
del ytt['color']       //删除
```



```python
for key,value in ytt.items():   //键值对不一定按顺序
	print(key,value)
    

for key in ytt.keys():
    print(key)

for key in ytt:
    print(key)
    
for value in ytt.values():
    print(key)
    
for value in ytt:
    print(key)
    
set(ytt.values())   //去重
```



```
ytts = [ytt1,ytt2,ytt3]         //列表嵌套字典
ytt = {'color':[red,green]}     //字典嵌套列表
users = {'ytt1':{'height':180},'ytt2':{'height':150}}   //字典嵌套字典
```



### IF语句



```
if car =='bmw':
	print("这是宝马")

and
or
in   //判断值是否在列表中

if
elif
else
```



### WHILE循环



Python 2.7  raw_input

```
name = input("输入名字")        //name以字符串读取
age = input("输入年龄")
age = int(age)
```



break

continue



```
cars = ['a','b','c']

while cars:
	cars.pop()

```



```
dic = {}
flag = True
while flag:
	key = input("")
	value = input("")
	dic[key] = value
```



### 函数



**位置实参**

```python
def hello(a,b)

hello(1,2)
```



**关键字实参**

```python
def hello(a,b)

hello(a = 1,b = 2)
```



**默认值**

```
def hello(a, b = 2)

hello(a = 1)
```



**返回值**

```
def build_person(first_name, last_name):
	person = build_person('first':first_name,'last_name',last_name)
	return person
```



**传递列表**

```
//在函数中修改列表是永久的
def greet_users(names):
	names.append("修改")

//禁止函数修改列表，可以切片传递副本
//调用方式
function_name(list_name[:])

```



**传递任意数量的实参**

```python
def make(*items)    			//函数将实参封装到元组中
def make(size,*items) 			//
def people(first,last,**items)	//
```



**导入模块**

模块是扩展名为.py的文件

pizza.py文件有make_pizza函数



在新文件中

```python
import pizza

//调用
pizza.make_pizza
```



**as指定别名**

```
from pizza import make_pizza as mp
import pizza as p
from pizza import*
```



注意：给形参默认值，等号两边不能有空格



**类**

```
class Dog():
	def __init__(self,name,age):
		self.name = name
		self.age = age
	
	def sit():
		print(self.name.title() + "")
	def roll_over():
		print(self.name.title() + "")
```



属性要有初始值

```
class Dog():
	def __init__(self,name,age,size):
		self.name = name
		self.age = age
		self.size = 0
```



修改初始值：

1.通过实例直接修改

```
dog = Dog('zz',2,5)
dog.size = 10
```



2.通过方法进行设置

```
class Dog():
	#略
	
	def set_size(newsize):
		self.size = newsize

dog = Dog('zz',2,5)
dog.set_size(10)
```



3.通过方法进行递增

```
class Dog():
	#略
	
	def add_size_year():
		self.size +=1 

dog = Dog('zz',2,5)
dog.set_size()
```



**继承**

1.子类的方法__init()__

子类首先要完成的任务是给父类所有属性赋值

```
class Car():
	def __init__(self,make,model,year):
		self.make = make
		self.model = model
		self.year = year
		self.size = 0
	#略	

class ElectriCar(Car):  #定义子类时，必须在括号内指定父类名称
	def __init__(self,make,model,year):
		#初始化父类的属性
		super().init(make,model,year)
```



```
#Python2.7中的继承
class Car():
	def __init__(self,make,model,year):
		#略	

class ElectriCar(Car):  
	def __init__(self,make,model,year):
		#初始化父类的属性
		super(ElectriCar,self).init(make,model,year)   #需要指定子类名和对象self
```



```
#给子类定义属性和方法
class Car():
	def __init__(self,make,model,year):
		self.make = make
		self.model = model
		self.year = year
		self.size = 0
	#略	

class ElectriCar(Car):  #定义子类时，必须在括号内指定父类名称
	def __init__(self,make,model,year):
		#初始化父类的属性
		super().init(make,model,year)
		self.battery = 0
```



```
#重写父类的方法
class Car():
	def __init__(self,make,model,year):
		self.make = make
		self.model = model
		self.year = year
		self.size = 0
	def read_size():
		return size

class ElectriCar(Car): 
	def __init__(self,make,model,year):
		super().init(make,model,year)
		self.battery = 0
	def read_size(self):  #这里重写需要加参数self吗？？
		return battery

```



```
#实例用作属性 (就是类成员变量)
class Car():
	def __init__(self,make,model,year):
		self.make = make
		self.model = model
		self.year = year
		self.size = 0
	def read_size():
		return size

class Battery():
	def __init__(self,battery = 0):
		self.battery = battery
		
class ElectriCar(Car): 
	def __init__(self,make,model,year):
		super().init(make,model,year)
		self.battery = Battery()     #实例用作属性
```



```
#导入类
#car.py
class Car():
	#xxx
	

#my_car.py
from car import Car    #一个模块导入一个类
xxx


from car import car,EletricCar   #一个模块导入多个类

import car           #导入整个模块

from car import *   #导入模块中所有类
```



**文件和异常**



```
#读文件
with open('a.txt') as object:
	contents = object.read()   #read到达文件末尾的时返回一个空字符串
	print(contents)
```



```
#逐行读取
with open('a.txt') as object:
	for line in object    #读取每行时返回一个空字符串
	print(line)
```



```
#使用with时，open()返回的文件对象只能在with代码块内可用，如果要在代码块外访问，可将其存储至列表中
with open('a.txt') as object:
	lines = object.readlines()    #读取每行存储在列表中
	
for line in lines：
	print(line)
```





```
#写入文件
with write('a.txt','w') as object:
	object.write('yes ok')

#打开模式  读取r  写入w  附加a   读取和写入r+
```



```
#写入多行
with write('a.txt','w') as object:
	object.write('yes ok\n')
	object.write('no')
```



```
#追加到文件 用附加模式打开
with write('a.txt','a') as object:
	object.write('yes ok\n')
	object.write('no')
```



**异常**



```
#try-except 
try:
	print(5/0)
except ZeroDicisionError:
	print("zero error")
```



```
#else 代码块
try:
	ans = 5/0
except ZeroDicisionError:
	print("zero error")
else:
	print(ans)
```



```
#处理FileNotFoundError

try:
	with open('b.txt') as obj:
except FileNotFoundError:
	print('file not found')
```



```
#捕获异常 什么都不做

try:
	with open('b.txt') as obj:
except FileNotFoundError:
	pass
```



```
#存储数据
import json

nums = [2,3,4,5,6,7]

filename = 'nums.json'
with open(filename,'w') as obj:
	json.dump(nums.obj)     #将数字列表存储到文件中
```





```
#读取json
import json

filename = 'nums.json'
with open(filename) as obj:
	nums = json.load(nums.obj)     #将数字列表读取
	
print(nums)
```



**测试代码**



```python
#name_function.py
def get_name(first,last):
	full_name = first + ' ' +last
	return full_name
```



```
#test_name_function.py

import unittest
from name_function import get_name

class NamesTestCase(unittest.TestCase):
	def test_first_last_name(self):
		name = get_name('y','tt')
		self.assertEqual(name,'y tt')
 

unittest.main()   #让Python运行这个文件中的测试
```



*断言方法*





```
#survey.py

class AnonymousSurvey():
	
	def __init__(self,question):
		self.question = question
		self.response =[]
	
	def show_question(self):
		print(self.question)

	def store_response(self,new_response):
		self.response.append(new_response)
	
	def show_results(self):
		print("result")
		for response in self.response:
			print('-' + response)
			
```



```
#test_survey.py

import unittest
from survey import AnonymousSurvey

class TestAnonymousSurvey(unittest.TestCase):
	#测试单个答案会被妥善的存储
	def test_score_single_response(self):
		question = 'What ....'
		my_survey = AnonymousSurvey(question)
		my_survey.store_response('English')
		
        self.assertIn('English',my_survey.response)
        
 unittest.main()
```



*SetUp方法*

```
#如果你在TestCase类中包含了方法setUp，Python将先运行它，再运行各个以test_打头的方法，这样，在你编写的每个测试方法中都可以使用在方法setUp中创建的对象了


import unittest
from survey import AnonymousSurvey

class TestAnonymousSurvey(unittest.TestCase):
	
	def setUp(self):
		question = 'What ....'
		self.my_survey = AnonymousSurvey(question)
		self.responses = ['dsadsa','dad','sa']
	
	def test_store_single_response(self):
		self.my_survey.store_response(self.responses[0])
        self.assertIn(self.response[0],self.my_survey.response)
        
 unittest.main()

```





