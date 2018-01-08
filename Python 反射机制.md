
有时候我们会碰到这样的需求，需要执行对象的某个方法，或是需要对对象的某个字段赋值，而方法名或是字段名在编码代码时并不能确定，需要通过参数传递字符串的形式输入。举个具体的例子：当我们需要实现一个通用的DBM框架时，可能需要对数据对象的字段赋值，但我们无法预知用到这个框架的数据对象都有些什么字段，换言之，我们在写框架的时候需要通过某种机制访问未知的属性

这个机制被称为反射,用于实现在运行时获取未知对象的信息

# 1.访问对象的属性

```
#coding: UTF-8
import sys #  模块，sys指向这个模块对象
import inspect
def foo(): pass # 函数，foo指向这个函数对象
 
class Cat(object): # 类，Cat指向这个类对象
    def __init__(self, name='kitty'):
        self.name = name
    def sayHi(self): #  实例方法，sayHi指向这个方法对象，使用类或实例.sayHi访问
        print self.name, 'says Hi!' # 访问名为name的字段，使用实例.name访问
 
cat = Cat() # cat是Cat类的实例对象
 
print Cat.sayHi # 使用类名访问实例方法时，方法是未绑定的(unbound)
print cat.sayHi # 使用实例访问实例方法时，方法是绑定的(bound)
```

以下列出了几个内建方法，可以用来检查或是访问对象的属性。这些方法可以用于任意对象而不仅仅是例子中的Cat实例对象；
**Python中一切都是对象。**

```
cat = Cat('kitty')
 
print cat.name # 访问实例属性
cat.sayHi() # 调用实例方法
 
print dir(cat) # 获取实例的属性名，以列表形式返回
if hasattr(cat, 'name'): # 检查实例是否有这个属性
    setattr(cat, 'name', 'tiger') # same as: a.name = 'tiger'
print getattr(cat, 'name') # same as: print a.name
 
getattr(cat, 'sayHi')() # same as: cat.sayHi()
```
- dir([obj]): 
调用这个方法将返回包含obj大多数属性名的列表（会有一些特殊的属性不包含在内）。obj的默认值是当前的模块对象。

- hasattr(obj, attr): 
这个方法用于检查obj是否有一个名为attr的值的属性，返回一个布尔值。

- getattr(obj, attr): 
调用这个方法将返回obj中名为attr值的属性的值，例如如果attr为'bar'，则返回obj.bar。

- setattr(obj, attr, val): 
调用这个方法将给obj的名为attr的值的属性赋值为val。例如如果attr为'bar'，则相当于obj.bar = val。

# 2.访问对象的元数据
当你对一个你构造的对象使用dir()时，可能会发现列表中的很多属性并不是你定义的。这些属性一般保存了对象的元数据，比如类的\__name__属性保存了类名。大部分这些属性都可以修改，不过改动它们意义并不是很大；修改其中某些属性如function.func_code还可能导致很难发现的问题，所以改改name什么的就好了，其他的属性不要在不了解后果的情况下修改。

## a).判断对象的类型
在types模块中定义了全部的Python内置类型

- isinstance(object, classinfo): 
检查object是不是classinfo中列举出的类型，返回布尔值。classinfo可以是一个具体的类型，也可以是多个类型的元组或列表。
types模块中仅仅定义了类型，而inspect模块中封装了很多检查类型的方法，比直接使用types模块更为轻松，所以这里不给出关于types的更多介绍，如有需要可以直接查看types模块的文档说明。本文第3节中介绍了inspect模块。

## b).模块
- \__doc__: 文档字符串。如果模块没有文档，这个值是None。
- \*__name__: 始终是定义时的模块名；即使你使用import .. as 为它取了别名，或是赋值给了另一个变量名。
- \*__dict__: 包含了模块里可用的属性名-属性的字典；也就是可以使用模块名.属性名访问的对象。
- \__file__: 包含了该模块的文件路径。需要注意的是内建的模块没有这个属性，访问它会抛出异常！
```
import fnmatch as m
print m.__doc__.splitlines()[0] # Filename matching with shell patterns.
print m.__name__                # fnmatch
print m.__file__                # /usr/lib/python2.6/fnmatch.pyc
print m.__dict__.items()[0]     # ('fnmatchcase', <function>)</function>
```
## c).类(class)
实例是指类实例化以后的对象。

- \*__dict__: 包含了可用的属性名-属性字典。
- \*__class__: 该实例的类对象。对于类Cat，cat.__class__ == Cat 为 True。
```
print cat.__dict__
print cat.__class__
print cat.__class__ == Cat # True
```

## d).内建函数和方法
- \__doc__: 函数或方法的文档。
- \__name__: 函数或方法定义时的名字。
- \__self__: 仅方法可用，如果是绑定的(bound)，则指向调用该方法的类（如果是类方法）或实例（如果是实例方法），否则为None。
- \*__module__: 函数或方法所在的模块名。

## e). 函数(function)
这里特指非内建的函数。注意，在类中使用def定义的是方法，方法与函数虽然有相似的行为，但它们是不同的概念。

- \__doc__: 函数的文档；另外也可以用属性名func_doc。
- \__name__: 函数定义时的函数名；另外也可以用属性名func_name。
- \*__module__: 包含该函数定义的模块名；同样注意，是模块名而不是模块对象。
- \*__dict__: 函数的可用属性；另外也可以用属性名func_dict。 
不要忘了函数也是对象，可以使用函数.属性名访问属性（赋值时如果属性不存在将新增一个），或使用内置函数has/get/setattr()访问。不过，在函数中保存属性的意义并不大。
- func_defaults: 这个属性保存了函数的参数默认值元组；因为默认值总是靠后的参数才有，所以不使用字典的形式也是可以与参数对应上的。
- func_code: 这个属性指向一个该函数对应的code对象，code对象中定义了其他的一些特殊属性，将在下文中另外介绍。
- func_globals: 这个属性指向定义函数时的全局命名空间。
- *func_closure: 这个属性仅当函数是一个闭包时有效，指向一个保存了所引用到的外部函数的变量cell的元组，如果该函数不是一个内部函数，则始终为None。这个属性也是只读的。
```
#coding: UTF-8
def foo():
    n = 1
    def bar():
        print n # 引用非全局的外部变量n，构造一个闭包
    n = 2
    return bar
 
closure = foo()
print closure.func_closure
# 使用dir()得知cell对象有一个cell_contents属性可以获得值
print closure.func_closure[0].cell_contents # 2
```

## f).方法(method)
方法虽然不是函数，但可以理解为在函数外面加了一层外壳；拿到方法里实际的函数以后，就可以使用2.5节的属性了。

- \__doc__: 与函数相同。
- \__name__: 与函数相同。
- \*__module__: 与函数相同。
- im_func: 使用这个属性可以拿到方法里实际的函数对象的引用。另外如果是2.6以上的版本，还可以使用属性名\__func__。
- im_self: 如果是绑定的(bound)，则指向调用该方法的类（如果是类方法）或实例（如果是实例方法），否则为None。如果是2.6以上的版本，还可以使用属性名\__self__。
- im_class: 实际调用该方法的类，或实际调用该方法的实例的类。注意不是方法的定义所在的类，如果有继承关系的话。
```
im = cat.sayHi
print im.im_func
print im.im_self # cat
print im.im_class # Cat
```

## g).
生成器(generator),代码块(code),栈帧(frame),追踪(traceback) 略

# 3. 使用inspect模块
## a).检查对象类型

- is{module|class|function|method|builtin}(obj): 
检查对象是否为模块、类、函数、方法、内建函数或方法。

- isroutine(obj): 
用于检查对象是否为函数、方法、内建函数或方法等等可调用类型。用这个方法会比多个is*()更方便，不过它的实现仍然是用了多个is*()。 
```
im = cat.sayHi
if inspect.isroutine(im):
    im()
```
## b).获取对象信息

- getmembers(object[, predicate]): 
这个方法是dir()的扩展版，它会将dir()找到的名字对应的属性一并返回，形如[(name, value), ...]。另外，predicate是一个方法的引用，如果指定，则应当接受value作为参数并返回一个布尔值，如果为False，相应的属性将不会返回。使用is*作为第二个参数可以过滤出指定类型的属性。

- getmodule(object): 
还在为第2节中的__module__属性只返回字符串而遗憾吗？这个方法一定可以满足你，它返回object的定义所在的模块对象。

- get{file|sourcefile}(object): 
获取object的定义所在的模块的文件名|源代码文件名（如果没有则返回None）。用于内建的对象（内建模块、类、函数、方法）上时会抛出TypeError异常。

- get{source|sourcelines}(object): 
获取object的定义的源代码，以字符串|字符串列表返回。代码无法访问时会抛出IOError异常。只能用于module/class/function/method/code/frame/traceack对象。

- getargspec(func): 
仅用于方法，获取方法声明的参数，返回元组，分别是(普通参数名的列表, *参数名, **参数名, 默认值元组)。如果没有值，将是空列表和3个None。如果是2.6以上版本，将返回一个命名元组(Named Tuple)，即除了索引外还可以使用属性名访问元组中的元素。

```
def add(x, y=1, *z):
    return x + y + sum(z)
print inspect.getargspec(add)
#ArgSpec(args=['x', 'y'], varargs='z', keywords=None, defaults=(1,))
```
- getargvalues(frame): 
仅用于栈帧，获取栈帧中保存的该次函数调用的参数值，返回元组，分别是(普通参数名的列表, *参数名, **参数名, 帧的locals())。如果是2.6以上版本，将返回一个命名元组(Named Tuple)，即除了索引外还可以使用属性名访问元组中的元素。
```
def add(x, y=1, *z):
    print inspect.getargvalues(inspect.currentframe())
    return x + y + sum(z)
add(2)
#ArgInfo(args=['x', 'y'], varargs='z', keywords=None, locals={'y': 1, 'x': 2, 'z': ()})
```

- getcallargs(func[, *args][, **kwds]): 
返回使用args和kwds调用该方法时各参数对应的值的字典。这个方法仅在2.7版本中才有。

- getmro(cls): 
返回一个类型元组，查找类属性时按照这个元组中的顺序。如果是新式类，与cls.__mro__结果一样。但旧式类没有\__mro__这个属性，直接使用这个属性会报异常，所以这个方法还是有它的价值的。 
```
print inspect.getmro(Cat)
#(<class '__main__.Cat'>, <type 'object'>)
print Cat.__mro__
#(<class '__main__.Cat'>, <type 'object'>)
class Dog: pass
print inspect.getmro(Dog)
#(<class __main__.Dog at 0x...>,)
print Dog.__mro__ # AttributeError
```
- currentframe(): 
返回当前的栈帧对象。
