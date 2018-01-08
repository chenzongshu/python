
# 引子
在python程序中，老看到函数上面一行加了@xxx，这是个神马东东？经过和度娘很不友好的交流，特别不容易的理解了这个概念：装饰器
这下子，我们就能愉快的写出很装B的程序啦

不过，装饰器是个什么意思呢，打个形象的比方
> 内裤可以用来遮羞，但是到了冬天它没法为我们防风御寒，聪明的人们发明了长裤，有了长裤后宝宝再也不冷了，装饰器就像我们这里说的长裤，在不影响内裤作用的前提下，给我们的身子提供了保暖的功效。

再回到我们的主题,装饰器本质上是一个Python函数，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象

# 装饰器入门

## 为什么需要装饰器
举个例子来说明下

我们的新入职小菜鸟写了个无聊的函数
```
def foo():
    print 'in foo()'
 
foo()
```
有一天，领导叫你统计下函数执行的时间，so easy，于是你简单的加了点代码
```
import time
def foo():
    start = time.clock()
    print 'in foo()'
    end = time.clock()
    print 'used:', end - start
 
foo()
```
但是，第二天，领导忽然叫你把所有函数都统计下时间，你在一万只羊驼从心里轰隆隆的跑过之后，开始想想怎么修修补补
苦想了很久之后，想到一个方法：定义一个函数timeit，将foo的引用传递给他，然后在timeit中调用foo并进行计时，这样，我们就达到了不改动每个函数定义的目的
```
import time
 
def foo():
    print 'in foo()'
 
def timeit(func):
    start = time.clock()
    func()
    end =time.clock()
    print 'used:', end - start
 
timeit(foo)
```
等等，这样是不是意味着调用foo()等的地方，都要改成timeit(foo)，虽然进了一步，但是这样很麻烦，万一某些地方漏了，也不易发现。

冥思苦想之后（咳咳，主要是问过度娘之后），发现python本身提供了一个高级功能，装饰器,于是改写代码如下
```
import time
 
def timeit(func):
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
 
@timeit
def foo():
    print 'in foo()'
 
foo()
```
其中，@就代表了装饰器符号,这样,我们只需要在函数定义前增加@timeit就行了

所以,总体来说,**装饰器其实也就是一个函数，一个用来包装函数的函数，返回一个修改之后的函数对象，将其重新赋值原来的标识符，并永久丧失对原始函数对象的访问。**

# 装饰器语法

## 无参装饰器

```
def deco(func):  
    print func  
    return func  
@deco  
def foo():pass  
foo()  
```
第一个函数deco是装饰函数，它的参数就是被装饰的函数对象。我们可以在deco函数内对传入的函数对象做一番“装饰”，然后返回这个对象（**记住一定要返回** ，不然外面调用foo的地方将会无函数可用。实际上此时foo=deco(foo)）

## 有参数装饰器
```
def decomaker(arg):  
    '通常对arg会有一定的要求'  
    """由于有参数的decorator函数在调用时只会使用应用时的参数  
       而不接收被装饰的函数做为参数，所以必须在其内部再创建  
       一个函数  
    """  
    def newDeco(func):    #定义一个新的decorator函数  
        print func, arg  
        return func  
    return newDeco  
@decomaker(deco_args)  
def foo():pass  
foo()  
```
第一个函数decomaker是装饰函数，它的参数是用来加强“加强装饰”的。由于此函数并非被装饰的函数对象，所以在内部必须至少创建一个接受被装饰函数的函数，然后返回这个对象（实际上此时foo=decomaker(arg)(foo)）

# 闭包
我们在使用装饰器的时候，实际上已经使用到了python的一个概念：闭包
下面来介绍下闭包

## 函数调用VS函数对象
假设我们写了一个函数 func(),
```
def func():
    return "hello,world"
```
对它进行两次赋值,看看有什么区别吗?
```
ref1 = func    # 将函数对象赋值给ref1
ref2 = func()  # 调用函数，将函数的返回值("hello,world"字符串)赋值给ref2
```
ref1引用了函数对象本身，而ref2则引用了函数的返回值。通过内建的callable函数，可以进一步验证ref1是可调用的，而ref2是不可调用的
```
type(ref1)
function

type(ref2)
str

callable(ref1)
True

callable(ref2)
false
```

## 闭包

函数是什么?
函数只是一段可执行代码，编译后就“固化”了，每个函数在内存中只有一份实例，得到函数的入口点便可以执行函数了。在函数式编程语言中，函数是一等公民（First class value：第一类对象，我们不需要像命令式语言中那样借助函数指针，委托操作函数），函数可以作为另一个函数的参数或返回值，可以赋给一个变量。函数可以嵌套定义，即在一个函数内部可以定义另一个函数，有了嵌套函数这种结构，便会产生闭包问题。
```
>>> def ExFunc(n):
     sum=n
     def InsFunc():
             return sum+1
     return InsFunc

>>> myFunc=ExFunc(10)
>>> myFunc()
11
>>> myAnotherFunc=ExFunc(20)
>>> myAnotherFunc()
21
>>> myFunc()
11
>>> myAnotherFunc()
21
>>>
```
当我们调用分别由不同的参数调用ExFunc函数得到的函数时（myFunc()，myAnotherFunc()），得到的结果是隔离的，也就是说每次调用ExFunc函数后都将生成并保存一个新的局部变量sum。其实这里ExFunc函数返回的就是闭包。

按照命令式语言的规则，ExFunc函数只是返回了内嵌函数InsFunc的地址，在执行InsFunc函数时将会由于在其作用域内找不到sum变量而出错。而在函数式语言中，当内嵌函数体内引用到体外的变量时，将会把定义时涉及到的引用环境和函数体打包成一个整体（闭包）返回.

如上述代码段中，当每次调用ExFunc函数时都将返回一个新的闭包实例，这些实例之间是隔离的，分别包含调用时不同的引用环境现场。不同于函数，闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

> 所谓闭包，就是将组成函数的语句和这些语句的执行环境打包在一起时，得到的对象,一个闭包就是你调用了一个函数A，这个函数A返回了一个函数B给你。这个返回的函数B就叫做闭包。你在调用函数A的时候传递的参数就是自由变量。

```
def func(name):
    def inner_func(age):
        print 'name:', name, 'age:', age
    return inner_func

bb = func('the5fire')
bb(26)  # >>> name: the5fire age: 26
```
这里面调用func的时候就产生了一个闭包——inner_func,并且该闭包持有自由变量——name，因此这也意味着，当函数func的生命周期结束之后，name这个变量依然存在，因为它被闭包引用了，所以不会被回收。

简单的说，这种内部函数可以使用外部函数变量的行为，就叫闭包。

注意: **闭包中是不能修改外部作用域的局部变量的**
> 在python3以后，在使用变量 x 之前，使用语句nonloacal x 就可以了，该语句显式的指定x不是闭包的局部变量。

闭包主要是在函数式开发过程中使用。以下介绍两种闭包主要的用途。
1 当闭包执行完后，仍然能够保持住当前的运行环境。
比如说，如果你希望函数的每次执行结果，都是基于这个函数上次的运行结果。我以一个类似棋盘游戏的例子来说明。假设棋盘大小为50*50，左上角为坐标系原点(0,0)，我需要一个函数，接收2个参数，分别为方向(direction)，步长(step)，该函数控制棋子的运动。棋子运动的新的坐标除了依赖于方向和步长以外，当然还要根据原来所处的坐标点，用闭包就可以保持住这个棋子原来所处的坐标。
```
origin = [0, 0]  # 坐标系统原点  
legal_x = [0, 50]  # x轴方向的合法坐标  
legal_y = [0, 50]  # y轴方向的合法坐标  
def create(pos=origin):  
    def player(direction,step):  
        # 这里应该首先判断参数direction,step的合法性，比如direction不能斜着走，step不能为负等  
        # 然后还要对新生成的x，y坐标的合法性进行判断处理，这里主要是想介绍闭包，就不详细写了。  
        new_x = pos[0] + direction[0]*step  
        new_y = pos[1] + direction[1]*step  
        pos[0] = new_x  
        pos[1] = new_y  
        #注意！此处不能写成 pos = [new_x, new_y]，原因在上文有说过  
        return pos  
    return player  
  
player = create()  # 创建棋子player，起点为原点  
print player([1,0],10)  # 向x轴正方向移动10步  
print player([0,1],20)  # 向y轴正方向移动20步  
print player([-1,0],10)  # 向x轴负方向移动10步  
```
输出为:
```
[10, 0]  
[10, 20]  
[0, 20]  
```

2 闭包可以根据外部作用域的局部变量来得到不同的结果，这有点像一种类似配置功能的作用，我们可以修改外部的变量，闭包根据这个变量展现出不同的功能。比如有时我们需要对某些文件的特殊行进行分析，先要提取出这些特殊行。
```
def make_filter(keep):  
    def the_filter(file_name):  
        file = open(file_name)  
        lines = file.readlines()  
        file.close()  
        filter_doc = [i for i in lines if keep in i]  
        return filter_doc  
    return the_filter  
```
如果我们需要取得文件"result.txt"中含有"pass"关键字的行，则可以这样使用例子程序
```
filter = make_filter("pass")  
filter_result = filter("result.txt")  
```
 
