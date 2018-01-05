
 Python教程太过繁杂，针对的前提的没怎么接触过编程的人士，最近在学Python，特列出与C程序不同之处，方便C程序员快速入门。

# 编译执行
python本身是解释性语言，不需要编译，可以直接使用python执行

```
python xxx.py
```

如果你嫌麻烦,可以在脚本的第一行指定解释器,即可把开头的python省略

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-   #这行中文用户一定要写
```

直接可以`xxx.py`执行


# 结构

- 没有括号，采用缩进代替语言块声明
- 语句结尾没有分号
- if,for,while和函数结尾带冒号":"

# 变量
变量赋值不需要声明类型，同一个变量可以反复赋值，而且可以是不同类型的变量

```
a =123   # a是整数
print a

a ='ABC'  # a变为字符串
print a
```

字符可乘

```
print ‘=’*10   #等于‘==========’
```

# 格式化输出
有两种方式,一种是类似于C语言的print函数,一种是2.7版本以后增加的format函数;
print函数和C类似,只是后面使用的是%,如`print "ID is %u and %u" % (num1,num2)`

另一种就是format函数,format函数使用占位符号" {} "表示第几个输入,":"做格式限定,而且有多种方式

通过位置:

```
In [1]: '{0},{1}'.format('kzc',18) 
Out[1]: 'kzc,18' 
In [2]: 'first is {}, second is {}'.format('kzc',18) 
Out[2]: 'first is kzc,second is 18' 
In [3]: '{1},{0},{1}'.format('kzc',18) 
Out[3]: '18,kzc,18'
```

通过关键字:

```
In [5]: '{name},{age}'.format(age=18,name='kzc') 
Out[5]: 'kzc,18'
```

通过对象属性:

```
class Person: 
  def __init__(self,name,age): 
    self.name,self.age = name,age 
  def __str__(self): 
    return 'This guy is {self.name},is {self.age} old'.format(self=self) 

In [2]: str(Person('kzc',18)) 
Out[2]: 'This guy is kzc,is 18 old'
```

通过数组下标:

```
In [7]: p=['kzc',18]
In [8]: '{0[0]},{0[1]}'.format(p)
Out[8]: 'kzc,18'
```

格式限定符:

```
In [54]: '{:b}'.format(17)  # b表示二进制,x表示十六进制,o表示八进制
Out[54]: '10001'

In [16]: '{:0>8}'.format('189') 
Out[16]: '00000189'   #^、<、>分别是居中、左对齐、右对齐,8为宽度,0表示填充位
```


# 字典
字典就是map

```
d = {'Michael':95,'Bob':75,'Tracy':85}
```

# 函数

- 如果想定义一个什么事也不做的空函数，可以用pass语句：

- 可以同时返回多个值  return x，y


# 切片

取一个list或tuple的部分元素，

```
L = ['Michael','Sarah','Tracy','Bob','Jack']
```

L[0:3]表示，从索引0开始取，直到索引3为止，但不包括索引3。即索引0，1，2，正好是3个元素。

如果第一个索引是0，还可以省略：  L[:3]，还支持负数 L[-2:-1]



# 迭代： 
只要是可迭代对象，无论有无下标，都可以迭代，通过collections模块的Iterable类型判断

```
>>>from collections import Iterable
>>>isinstance('abc', Iterable) # str是否可迭代
True

>>>d = {'a':1,'b':2,'c':3}

>>>for key in d: 

       print key
```


# 列表生成式

生成[1x1, 2x2, 3x3, ..., 10x10]

```
>>> [x * x for x in range(1, 11)]

[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

>>> [x * x for x in range(1, 11) if x % 2 == 0]

[4, 16, 36, 64, 100]

>>> [m + n for m in 'ABC' for n in 'XYZ']

['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

# 装饰符

@property

# namedtuple

首先回忆一下tuple的概念，即不可修改的数组，一般情况下的 tuple 是 (item1, item2, item3,...)，所有的 item 都只能按照 index 访问，有时候容易搞错。

namedtuple顾名思义就是命名的tuple，类似于C的结构体，事先把这些 item 命名，以后可以方便访问

```
from collections import namedtuple

# 定义一个namedtuple类型User，并包含name，sex和age属性。
User = namedtuple('User', ['name', 'sex', 'age'])

# 创建一个User对象
user = User(name='kongxx', sex='male', age=21)

# 也可以通过一个list来创建一个User对象，这里注意需要使用"_make"方法
user = User._make(['kongxx', 'male', 21])

print user
# User(name='user1', sex='male', age=21)

# 获取用户的属性
print user.name
print user.sex
print user.age

# 修改对象属性，注意要使用"_replace"方法
user = user._replace(age=22)
print user
# User(name='user1', sex='male', age=21)

# 将User对象转换成字典，注意要使用"_asdict"
print user._asdict()
# OrderedDict([('name', 'kongxx'), ('sex', 'male'), ('age', 22)])
```
