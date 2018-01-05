# 前置:

闭包:
首先来复习下什么是闭包.
闭包是一类特殊的函数。如果一个函数定义在另一个函数的作用域中，并且函数中引用了外部函数的局部变量，那么这个函数就是一个闭包。下面的代码定义了一个闭包：
```
def f():
    n = 1
    def inner():
        print n
    inner()
    n = 'x'
    inner()
```
函数inner定义在f的作用域中，并且在inner中使用了f中的局部变量n，这就构成了一个闭包。闭包绑定了外部的变量，所以调用函数f的结果是打印1和'x'。这类似于普通的模块函数和模块中定义的全局变量的关系：修改外部变量能影响内部作用域中的值，而在内部作用域中定义同名变量则将遮蔽（隐藏）外部变量。

如果需要在函数中修改全局变量，可以使用关键字global修饰变量名。Python 2.x中没有关键字为在闭包中修改外部变量提供支持，在3.x中，关键字nonlocal可以做到这一点：
```
#Python 3.x supports `nonlocal'
def f():
    n = 1
    def inner():
        nonlocal n
        n = 'x'
    print(n)
    inner()
    print(n)
```
# 什么是函数式编程

特征

- 函数是一等公民
- 匿名函数(lambda) 
- 封装控制结构的内置模板函数 
- 闭包(closure) 
- 内置的不可变数据结构 
- 递归 

# 函数

## 从函数开始

### 1.定义一个函数
```
def add(x, y):
    return x + y
```
在python中,一切即对象,可以给函数定义一个别名,使用别名调用该函数
```
add_a_number_to_another_one_by_using_plus_operator = add
print add_a_number_to_another_one_by_using_plus_operator(1, 2)
```
既然函数可以被变量引用，那么将函数作为参数和返回值就是很寻常的做法了。

## 2.作为参数
使用上面定义的函数来写一段指令式风格的代码,完成数组的相加
```
lst = range(5) #[0, 1, 2, 3, 4]

amount = 0
for num in lst:
    amount = add(amount, num)
```
下面,使用函数式风格来重构下:
首先可以预见的是求和这个动作是非常常见的，如果我们把这个动作抽象成一个单独的函数，以后需要对另一个列表求和时，就不必再写一遍这个套路了：
```
def sum_(lst):
    amount = 0
    for num in lst:
        amount = add(amount, num)
    return amount
 
print sum_(lst)
```
还能继续。sum_函数定义了这样一种流程： 
1. 使用初始值与列表的第一个元素相加； 
2. 使用上一次相加的结果与列表的下一个元素相加； 
3. 重复第二步，直到列表中没有更多元素； 
4. 将最后一次相加的结果返回。

如果现在需要求乘积，我们可以写出类似的流程——只需要把相加换成相乘就可以了：
```
def multiply(lst):
    product = 1
    for num in lst:
        product = product * num
    return product
```
除了初始值换成了1以及函数add换成了乘法运算符，其他的代码全部都是冗余的。我们为什么不把这个流程抽象出来，而将加法、乘法或者其他的函数作为参数传入呢？
```
def reduce_(function, lst, initial):
    result = initial
    for num in lst:
        result = function(result, num)
    return result
 
print reduce_(add, lst, 0)
```
现在，想要算出乘积，可以这样做：
```
print reduce_(lambda x, y: x * y, lst, 1)
```
## 3.作为返回值
将函数返回通常需要与闭包一起使用（即返回一个闭包）才能发挥威力。我们先看一个函数的定义：
``` python
def map_(function, lst):
    result = []
    for item in lst:
        result.append(function(item))
    return result
```
函数map_封装了最常见的一种迭代：对列表中的每个元素调用一个函数。map_需要一个函数参数，并将每次调用的结果保存在一个列表中返回。这是指令式的做法，当你知道了列表解析(list comprehension)后，会有更好的实现。

对于上一节中的lst，你可能发现最后求乘积结果始终是0，因为lst中包含了0。为了让结果看起来足够大，我们来使用map_为lst中的每个元素加1：
```
lst = map_(lambda x: add(1, x), lst)
print reduce_(lambda x, y: x * y, lst, 1)
```
继续
```
lst = map_(lambda x: add(10, x), lst)
print reduce_(lambda x, y: x * y, lst, 1)
```
回头看看我们写的两个lambda表达式：相似度超过90%，绝对可以使用抄袭来形容。而问题不在于抄袭，在于多写了很多字符有木有？如果有一个函数，根据你指定的左操作数，能生成一个加法函数，用起来就像这样：
```
lst = map_(add_to(10), lst) #add_to(10)返回一个函数，这个函数接受一个参数并加上10后返回
```
写起来应该会舒服不少。下面是函数add_to的实现：
```
def add_to(n):
    return lambda x: add(n, x)
```
通过为已经存在的某个函数指定数个参数，生成一个新的函数，这个函数只需要传入剩余未指定的参数就能实现原函数的全部功能，这被称为偏函数。Python内置的functools模块提供了一个函数partial，可以为任意函数生成偏函数：
```
functools.partial(func[, *args][, **keywords])
```
你需要指定要生成偏函数的函数、并且指定数个参数或者命名参数，然后partial将返回这个偏函数；不过严格的说partial返回的不是函数，而是一个像函数一样可直接调用的对象，当然，这不会影响它的功能。
