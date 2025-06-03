+++
title = 'Python basic'
date = 2023-11-22T21:22:27+08:00
draft = false

tags = ["python","python-basic"]
categories = ["DevOps"]

+++

# Python 基础

## 一、基础语法

### 1、输出语句的使用

1）引号的使用

```python
>>> print ('hello world')
hello world
>>> print ("hello world")
hello world
>>>
```

2）三引号: 输出多行内容

```python
>>> print ("""abc
... cdc
... ddd
... ccc
... fff""")
abc
cdc
ddd
ccc
fff
```

注释符：# 

python2与python3的区别

* print语法不同，python3需要加()  
* python2默认使用的字符集ASCII码,  # encoding: utf8。python3默认使用的字符集Unicode



2）输出变量

```python
username = "root"
password = "redhat"
 
print(username)
 
print("my name is ", username, "my password is", password)
print("my name is " + username + "my password is " + password)      #只适用于字符串
    
```

3）格式化输出

```python
>>> username = "root"
>>> password = "redhat"
>>> print("my name is %s" % username )
my name is root
>>> print("my name is '%s'" % username)
my name is 'root'
>>> print("my name is %s, my password is %s" % (username, password))
my name is root, my password is redhat
```

常用的格式化字符

```python
%s    字符串   通用 
%d    数字，整数 
        
number_01 = 123
number_02 = "456"
number_03 = 3.9415
 
print("It is %d" % number_01)
print("It is %d" % number_03)
        
%f    浮点数，小数 
        
number_01 = 123
number_02 = 3.948915
number_03 = 3.9489151111111111
 
print("It is %f" % number_01)
print("It is %f" % number_02)
print("It is %f" % number_03)
 
print("It is %.2f" % number_02)
 
        
        
%%      输出%本身
number = 50
 
# This is 50%
print("This is %d%%" % number)  
 
 
```

```python
username = "root"
password = "redhat"
 
sql_01 = "select * from tb01 where username='%s' and password='%s'" % (username, password)
print(sql_01)
```

### 2、变量定义

变量名称规范：

* 字母、数字、下划线_ 
* 只能以字母、下划线_开头
* 不能与python关键字冲突。（`print`   `if`   ` for`  `while`  `else` ）


```python
#交互式变量赋值      
username = input("输入用户名： ")
        
#注意：
#返回的结果是字符串 
print("用户名： %s" % username)
```

删除变量

```python
name = "Martin"
print(name)
 
del name
print(name)
```

python变量与其他语言不同之处

1) 弱类型
2) 地址引用类型

```python
number_01 = 10
number_02 = 10
 
print(id(number_01))
print(id(number_02))
 
number_01 = 100
number_02 = number_01
        
print(id(number_01))
print(id(number_02))
```

内置函数

```python
id()        返回变量的内存地址 
type()      返回变量的类型 
```

内存使用机制 
		每个变量定义后，会在内存中开辟一段空间，这段空间对应存在一个引用计数器，变量被调用一次，引用计数器会自动增加；当python解释器检测到一段内存的引用计数器为0后，会自动清理该内存



### 3、变量的类型

* 数字
* 字符串
* 列表
* 元组
* 字典
* 集合
* Bytes



### 4、运算

```python
#数学运算符 
>>> a = 10
>>> b = 4
>>> a + b
14
>>> a - b
6
>>> a * b
40
>>> a ** b
10000
>>>
>>> a / b
2.5
>>> a // b
2
>>> a % b
2
 
>>> a = 10
>>> a = a + 1
>>> a
11
>>> a += 1
>>> a
12
>>>
 
#比较运算符  
==, !=, >, >=, <=, < 
    
逻辑运算符 
and, or, not  
 
>>> a = 10
>>>
    >>>
>>> a > 20 and 1 < 2
False
>>>
>>> a > 20 or 1 < 2
True
>>>
>>> not a > 20
True
>>>
 
数制转换 
>>> a = 10
>>> bin(a)
'0b1010'
>>> oct(a)
'0o12'
>>> hex(a)
'0xa'
>>>
```



```python
#生成随机数的模块  
>>> import random
>>> random.randint(0, 10)
7
>>> random.randint(0, 10)
9
>>> random.randint(0, 10)
2
>>> random.randint(0, 10)
3
>>> random.randint(0, 10)
6   
```

示例：四则运算

```python
number_01 = int(input("输入第1个数字： "))
number_02 = int(input("输入第2个数字： "))
 
print("%s + %s = %s" % (number_01, number_02, number_01 + number_02))
print("%s - %s = %s" % (number_01, number_02, number_01 - number_02))
print("%s * %s = %s" % (number_01, number_02, number_01 * number_02))
print("%s / %s = %s" % (number_01, number_02, number_01 / number_02))
print("%s // %s = %s" % (number_01, number_02, number_01 // number_02))
   
```

### 5、逻辑控制语句 

同级代码要有相同缩进，默认4个空格

条件判断 --- if

1) if 条件:
    操作语句
    操作语句 

```python
if 1 < 2:
    print("AAA")
    print("BBB")
 
# 只要数字不等于0，条件为真
 
number = 10
if number:
   print("AAA")
   print("BBB")
 
number = 10
if not number:
    print("AAA")
    print("BBB")

# True, False 首字母全是大写
if True:
    print("AAA")
```

2) if .... else 

```python
number = 100
 
if number < 20:
    print("AAAA")
else:
    print("BBBB")
```

3) if ... elif ... elif .... else

```python
number = int(input("Enter number: "))
 
if number > 10:
    print("AAA")
elif number > 30:
    print("BBBB")
elif number > 40:
    print("CCCC")
else:
    print("DDDD")
```

4) 嵌套if

```python
age = int(input("输入你的年龄： "))

if age <= 18:
    gender = input("输入你的性别： ")
    if gender == "M":
        print("准备入队")
    else:
        print("睡吧")
else:
    print("回家洗洗睡吧")
```

循环

* for 
* while




for循环：

```python
for 变量 in 取值:
    操作语句
    操作语句
```

```python
for i in range(5):
    print("第%s次循环开始" % i)
    print("第%s次循环结束" % i)
    print("-----------------")
```

中断循环：

```python
#break       中断整体循环
 
for i in range(5):
    print("第%s次循环开始" % i)
    if i == 3:
        break
    print("第%s次循环结束" % i)
    print("-----------------")
```


continue    中断本次循环   

```python
for i in range(5):
    print("第%s次循环开始" % i)
    if i == 3:
        continue
    print("第%s次循环结束" % i)
    print("-----------------")
```

示例：斐波那契数列

```python
length=int(input("input length:"))
i=0
j=1
tmp=1
for n in range(length):
    print( tmp," ",end=)
    tmp = i + j
    i = j
    j = tmp
    
print()
```



```python
while循环
 
while 条件:
    操作语句
    操作语句
        
i = 1

while i <= 4:
    print("第%s次循环开始" % i)
    print("第%s次循环结束" % i)
    print("-----------------")
    i += 1
 
while True:
    操作语句
    操作语句
```

示例： 实现数制转换 

```python
import sys
 
number = int(input("输入数字： "))
 
menu = """
1、二进制
2、八进制
3、十六进制
4、退出
 
输入你的选择：d
"""
 
'''
    循环判断用户的选择，根据不同的选择做不同的响应 
'''
 
while True:
    choice = int(input(menu))
 
    if choice == 1:
        print("数字%s的二进制形式：%s" % (number, bin(number)))
    elif choice == 2:
        print("数字%s的八进制形式：%s" % (number, oct(number)))
    elif choice == 3:
        print("数字%s的十六进制形式：%s" % (number, hex(number)))
    else:
        print("谢谢")
        sys.exit()
```

pass: 占位符

```python
#!/usr/bin/python
import time

for i in range(5):
    print i
    if i == 1:
        pass    ---- 代码桩 if之间不能空代码可以用pass占位
    if i == 2:
        continue
    if i == 3:
        break
    print "#"*10
else:
    print "END"

for j in range(2):
    print "--->",j
[root@server python]# python dic.py 
0
##########
1
##########
2
3
---> 0
---> 1
```

结束执行

```python
[root@server python]# cat dic.py 
#!/usr/bin/python

for i in range(10):
    print i
    if i == 5:
        exit()
[root@server python]# python dic.py 
0
1
2
3
4
5
```



switch语句

* switch语句用于编写多分支结构的程序,类似与if... elif... else语句
* switch语句表达的分支结构比if... elif... else语句表达的更清晰,代码可读性更高，但是python并没有提供switch语句
* python可以通过字典实现switch语句的功能
  实现方法分为两步
* 首先：定义一个字典，其次,调用字典的get()获取相应的表达式xxxxxxxxxx 

```python
[root@www python]# cat test.py 
#!/usr/bin/bash
#coding:utf8
from __future__ import division
def jia(x,y):
	return x+y

def jian(x,y):
	return x-y

def cheng(x,y):
	return x*y

def chu(x,y):
	return x/y

def operator(x,o,y):
	if o == "+":
		print jia(x,y)
	elif o == "-":
		print jian(x,y)
	elif o == "*":
		print cheng(x,y)
	elif o == "/":
		print chu(x,y)
	else:
		pass
operator(2,"/",4)
```

```python
#!/usr/bin/bash
#coding:utf8
from __future__ import division
def jia(x,y):
    return x+y

def jian(x,y):
    return x-y

def cheng(x,y):
    return x*y

def chu(x,y):
    return x/y

operator = {"+":jia,"-":jian,"*":cheng,"/":chu}

print operator ["/"](3,2)

```

示例：

写一个卖水果的菜单(有菜单),再将他转换为switch格式

```python
menu="""
1、apple
2、banala
3、orange
"""

fuirt={1:['apple',5],2:['banala',3],3:['orange',7]}
print(fuirt[1])
while True:
    print(menu)
    tmp=int(input("请输入要查询价格的水果:"))
    print("%s的价格是 %s 元/斤"%(fuirt[tmp][0],fuirt[tmp][1]))

```



### 六、函数

1、格式

```python
未定义返回值
>>> def fun():
...     print ("a")
       
>>> var=fun()
a
>>> print (var)
None

定义返回值
>>> def fun():
...     return "ok"
... 
... 
>>> var=fun()
>>> print (var)
ok
```

2、默认值

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print "Name: ", name
   print "Age ", age
   return
 
#调用printinfo函数
printinfo( age=50, name="miki" )
printinfo( name="miki" )
```

3、不定长参数

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
def printinfo( arg1, *vartuple ):
   "打印任何传入的参数"
   print "输出: "
   print arg1
   for var in vartuple:
      print var
   return
 
# 调用printinfo 函数
printinfo( 10 )
printinfo( 70, 60, 50 )
```

### 七、模块

搜索路径是一个解释器会先进行搜索的所有目录的列表。如想要导入模块 support.py，需要把命令放在脚本的顶端：

```python
# support.py
def print_func( par ):
   print "Hello : ", par
   return


# test.py
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 导入模块
import support
 
# 现在可以调用模块里包含的函数了
support.print_func("sugar")
```

Python 的 from 语句让你从模块中导入一个指定的部分到当前命名空间中。语法如下：

```
import module_a #导入整个模块功能
module_a.xxx #调用

from module import xx # 导入某个模块下的某个方法 or 子模块
from module.xx.xx import xx as rename #导入后一个方法后重命令
from module.xx.xx import * #导入一个模块下的所有方法，不建议使用
```





### 八、面向对象

定义类与实例化

```python
class Ren:

    name = "人"

    def run(self):
        print("跑步")

cmd = Ren()
print(cmd)
```



私有属性与方法

在属性或者方法前加`__`的，即表示该方法和属性为私有类外不可调用

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
class Employee:
   '所有员工的基类'
   empCount = 0
 
   def __init__(self, name, salary):
      self.name = name
      self.salary = salary
      Employee.empCount += 1
   
   def displayCount(self):
     print "Total Employee %d" % Employee.empCount
 
   def displayEmployee(self):
      print "Name : ", self.name,  ", Salary: ", self.salary
```



类的继承

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
class Parent:        # 定义父类
   parentAttr = 100
   def __init__(self):
      print "调用父类构造函数"
 
   def parentMethod(self):
      print '调用父类方法'
 
   def setAttr(self, attr):
      Parent.parentAttr = attr
 
   def getAttr(self):
      print "父类属性 :", Parent.parentAttr
 
class Child(Parent): # 定义子类
   def __init__(self):
      print "调用子类构造方法"
 
   def childMethod(self):
      print '调用子类方法'
 
c = Child()          # 实例化子类
c.childMethod()      # 调用子类的方法
c.parentMethod()     # 调用父类方法
c.setAttr(200)       # 再次调用父类的方法 - 设置属性值
c.getAttr()          # 再次调用父类的方法 - 获取属性值
```

方法重写

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
class Parent:        # 定义父类
   def myMethod(self):
      print '调用父类方法'
 
class Child(Parent): # 定义子类
   def myMethod(self):
      print '调用子类方法'
 
c = Child()          # 子类实例
c.myMethod()         # 子类调用重写方法
```





## 补充

```python
# 格式化字符串，两种方式

>>> year=2019
>>> month=6
>>> day=18
>>> f"Today is {year} {month} {day}"
'Today is 2019 6 18'

>>> "a{}{}".format("b","c")
'abc'
```

python 本地包导入

```python
#filepath
# addons/book/book.py
# python的包是以目录+文件名+文件名里的对象形式使用
from addons.book.book import book_bp
     
```



**装饰器**

装饰器是一种用于修改函数或方法行为的高级功能。装饰器本质上是一个接受函数作为参数的函数，并返回一个新的函数或方法。它们通常用于日志记录、访问控制、缓存结果、计时函数执行等场景。

```python
def my_decorator(func):
    def action():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return action

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

