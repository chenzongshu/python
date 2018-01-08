
Python内置了一些非常有趣但非常有用的函数，充分体现了Python的语言魅力！

# filter
原型：filter(function, sequence)
对sequence中的item依次执行function(item)，将执行结果为True的item组成一个List/String/Tuple（取决于sequence的类型）返回
```python
>>>def f(x): return x % 2 != 0 and x % 3 != 0 
>>>filter(f, range(2, 25)) 
```
结果为： [5, 7, 11, 13, 17, 19, 23]
```
>>>def f(x): return x != 'a' 
>>>filter(f, "abcdef") 
```
结果为：'bcdef'

**function的返回值只能是True或False**
特殊情况如sequence是string或tuple，则返回值按照sequence的类型,如：
```
filter(lambda x:len(x)!=0,'hello') #返回'hello'
filter(lambda x:len(x)==0,'hello') #返回''
```

# map
原型：map(function, sequence) 
对sequence中的item依次执行function(item)，见执行结果组成一个List返回
```python
>>> def cube(x): return x*x*x 
>>> map(cube, range(1, 11)) 
```
结果为： [1, 8, 27, 64, 125, 216, 343, 512, 729, 1000]
```
>>> def cube(x) : return x + x
>>> map(cube , "abcde") 
```
结果为：['aa', 'bb', 'cc', 'dd', 'ee']

另外map也支持多个sequence，这就要求function也支持相应数量的参数输入：
```
>>> def add(x, y): return x+y 
>>> map(add, range(8), range(8)) 
```
结果为： [0, 2, 4, 6, 8, 10, 12, 14]

# reduce
原型：reduce(function, sequence, starting_value)：
对sequence中的item顺序迭代调用function，如果有starting_value，还可以作为初始值调用，例如可以用来对List求和：
```
>>> def add(x,y): return x + y 
>>> reduce(add, range(1, 11)) 
```
55 （注：1+2+3+4+5+6+7+8+9+10）
```
>>> reduce(add, range(1, 11), 20) 
```
75 （注：1+2+3+4+5+6+7+8+9+10+20）

function接收的参数个数只能为2,先把sequence中第一个值和第二个值当参数传给function，再把function的返回值和第三个值当参数传给function，然后只返回一个结果。
第一个例子可以写成
```
reduce(lambda x,y:x+y,range(1,11))
```

# lambda
lambda：这是Python支持一种有趣的语法，它允许你快速定义单行的最小函数，类似与C语言中的宏，这些叫做lambda的函数，是从LISP借用来的，可以用在任何需要函数的地方： 
```
>>> g = lambda x: x * 2 
>>> g(3) 
```
6 
```
>>> (lambda x: x * 2)(3) 
```
6

map，reduce，filter中的function都可以用lambda表达式来生成！

如：求1*1,2*2,3*3,4*4
```
map(lambda x:x*x,range(1,5))
```
返回值是[1,4,9,16]
