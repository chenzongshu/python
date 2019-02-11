
**前面的话: 热更新能解决一些python代码更新的问题, 但是绝对不是解决一切发版问题的灵丹妙药**

# 热更新

假设你有一个网站有不少的用户量，并时不时需要更新新功能。你不希望在更新新代码的时候把整个网站停掉重启进程，因为这期间可能会流失一部分用户，于是你就在想能不能只需要把旧代码替换成新代码无需重启进程就实现更新呢？ 这就是热更新

简单来说，热更新就是在进程不重启的情况下，让其加载修改后的程序代码，且能按照预期正确执行

# python热更新

python是一种脚本语言, 动态解析, 所以用来写服务端一个很大的优势是可以热更新, 即在服务不停的情况下, 使修改后的代码生效, 在生产环境可以以最小的代价发布版本.

有两种方法, 一种是用守护进程的方法, 另外一种是使用python自带的reload函数.

# 热更新存在的问题

- 函数热更新不支持decorator
- 进程的内存会增长
- 类的实列更新

# reload函数

python2中的reload函数可以直接使用，无需导入第三方模块，可以直接使用：

```
reload(module)  # reload接收的参数必须是已经导入的模块
```

python3中的reload函数移到了imp库里面，因此需要导入：

```
from imp import reload
reload(module)
```

## reload使用注意事项

先说结论, 

- **不是打 patch(可以有旧对象删除、有新对象增加、有旧对象修改)；而是把 reload 时生成的新对象替换掉同名旧对象；无法删除旧对象**
- **class 类继承之后不能更新, 必须重新import一次**
- **reload不支持`from plugins improt plugin`的方式重载模块，因此可以使用`import plugins.plugin`的方式导入模块并重载**



# 原理

我们新建一个test_refresh.py文件

```
from __future__ import print_function

class RefreshClass(object):
    def __init__(self):
        self.value = 1

    def print_info(self):
        print('RefreshClass value: {} ver1.0'.format(self.value))

version = 3.0

def fprint():
  print('test')
```

## 命令空间(namespace)

namesapce就是name到对象的映射, 有四类

- `local`: 包含局部变量, 比如一个函数/方法内部
- `Enclosing`: 闭包函数的命名空间
- `Global`: 模块的namespace, 每个模块有一个自己的namespace, 记录模块内的class, function等
- `Built-in`: 包含了内建的变量/关键字

那namespace是派什么用的呢？在python中，如果要访问某一个对象（包括变量，模块，方法等）都是会去namespace中根据对象名称去检索，这里涉及到一个检索顺序，称之为：LEGB ，就是：

`locals` -->> `enclosing` -->> `globals` -->> `__builtins__`

## import

### import介绍

第一次import的时候, 执行了三个步骤:

- 1、找到模块文件, 注意不需要文件的路径后缀
> 搜索路径: 程序主目录 -> PYTHONPATH目录 -> 标准链接库目录

- 2、编译成字节码
> 根据时间戳决定是否生成新的pyc文件

- 3、执行模块的代码来创建其所定义的对象
> 程序会讲导入的文件从头到尾执行一遍，在此过程中任何对变量名进行的赋值操作，都会产生得到的模块文件的属性，但是要注意这些对变量名的赋值操作必须是在模块文件的顶层进行的操作，例如使用def语句来定义函数，模块文件中便会添加这个定义的函数属性

**注意: from语句是将变量名复制到另一个作用域，我们可以在脚本重直接使用复制后的变量名**

当我们import一个module的时候，实际上python是把该import信息写到了`sys.modules`这个dict中，我们可以看看这个dict

```
>>> print sys.modules
{
    'copy_reg': <module 'copy_reg' from '/usr/lib64/python2.7/copy_reg.pyc'>,   
    'sre_compile': <module 'sre_compile' from '/usr/lib64/python2.7/sre_compile.pyc'>,  
    '_sre': <module '_sre' (built-in)>, 
    'encodings': <module 'encodings' from '/usr/lib64/python2.7/encodings/__init__.pyc'>,   
    'site': <module 'site' from '/usr/lib64/python2.7/site.pyc'>, 
    '__builtin__': <module '__builtin__' (built-in)>,   
    'sysconfig': <module 'sysconfig' from '/usr/lib64/python2.7/sysconfig.pyc'>,    
    '__main__': <module '__main__' (built-in)>, 
    'encodings.encodings': None,    
    'abc': <module 'abc' from '/usr/lib64/python2.7/abc.pyc'>,  
    'posixpath': <module 'posixpath' from '/usr/lib64/python2.7/posixpath.pyc'>,    
    '_weakrefset': <module '_weakrefset' from '/usr/lib64/python2.7/_weakrefset.pyc'>,  
    'errno': <module 'errno' (built-in)>,   
    'encodings.codecs': None,   
    'sre_constants': <module 'sre_constants' from '/usr/lib64/python2.7/sre_constants.pyc'>,
    're': <module 're' from '/usr/lib64/python2.7/re.pyc'>, 
    '_abcoll': <module '_abcoll' from '/usr/lib64/python2.7/_abcoll.pyc'>,  
    'types': <module 'types' from '/usr/lib64/python2.7/types.pyc'>,    
    '_codecs': <module '_codecs' (built-in)>,   
    'encodings.__builtin__': None,  
    '_warnings': <module '_warnings' (built-in)>,   
    'genericpath': <module 'genericpath' from '/usr/lib64/python2.7/genericpath.pyc'>,  
    'stat': <module 'stat' from '/usr/lib64/python2.7/stat.pyc'>, 
    'zipimport': <module 'zipimport' (built-in)>,   
    '_sysconfigdata': <module '_sysconfigdata' from '/usr/lib64/python2.7/_sysconfigdata.pyc'>, 
    'warnings': <module 'warnings' from '/usr/lib64/python2.7/warnings.pyc'>,   
    'UserDict': <module 'UserDict' from '/usr/lib64/python2.7/UserDict.pyc'>,   
    'sys': <module 'sys' (built-in)>,   
    'codecs': <module 'codecs' from '/usr/lib64/python2.7/codecs.pyc'>, 
    'readline': <module 'readline' from '/usr/lib64/python2.7/lib-dynload/readline.so'>,    
    'os.path': <module 'posixpath' from '/usr/lib64/python2.7/posixpath.pyc'>,  
    'signal': <module 'signal' (built-in)>, 
    'traceback': <module 'traceback' from '/usr/lib64/python2.7/traceback.pyc'>,    
    'linecache': <module 'linecache' from '/usr/lib64/python2.7/linecache.pyc'>,    
    'posix': <module 'posix' (built-in)>,   
    'exceptions': <module 'exceptions' (built-in)>, 
    'sre_parse': <module 'sre_parse' from '/usr/lib64/python2.7/sre_parse.pyc'>,    
    'os': <module 'os' from '/usr/lib64/python2.7/os.pyc'>, 
    '_weakref': <module '_weakref' (built-in)>
}
>>> 
>>> type(sys.modules)
<type 'dict'>
```

单独打印上面的module，可以看到

```
>>> print sys.modules['test_refresh']
<module 'test_refresh' from 'test_refresh.py'>
```

即key是`test_refresh`, value是`<module 'test_refresh' from 'test_refresh.py'>`

当我们导入一个新的模块的时候，以下两件事情将会发生：

- 1、会在sys.module中插入一条key-value对，key是module名，value就是所导入的module对象。当下一次import相同模块的时候，会先在sys.module中查找该模块，如果存在则直接导入sys.module中的module对象。
- 2、将module对象加入到global namespace中，当程序需要调用该模块时，会从global namespace中检索。


## reload

reload之后，通过变量的值，可以看到该变量是重新指向了新的地址, 注意reload之后, 如果原来py的文件没改, pyc文件不会更新; 如果py文件被修改, pyc会更新

```
>>> id(test_refresh.version)
26305408
>>>
>>> reload(test_refresh)
3.0
<module 'test_refresh' from 'test_refresh.py'>
>>>
>>> id(test_refresh.version)
25376416
```

同理，修改函数值之后，函数也指向了新的地址

```
>>> test_refresh.fprint
<function fprint at 0x7f66a0300668>
>>>
>>> reload(test_refresh)
<module 'test_refresh' from 'test_refresh.py'>
>>>
>>> test_refresh.fprint
<function fprint at 0x7f66a03007d0>
```

### 类的更新



```
class Class11(object):
    v = 1
    def __init__(self):
        self.value = 11

    def print_info(self):
        print('RefreshClass11 value: {} ver1.1'.format(self.value))
        print('v is {}'.format(self.v))
```

```
>>> import test1
>>> c1 = test1.Class11()
>>> c1.print_info()
RefreshClass11 value: 11 ver1.0
v is 1

# 更改同时更改Class11()的v, value和打印值, 然后reload
>>> reload(test1)
<module 'test1' from 'test1.py'>
>>> c1.print_info()
RefreshClass11 value: 11 ver1.0
v is 1
>>> c1.__class__=test1.Class11
>>> c1.print_info()
RefreshClass11 value: 11 ver1.1
v is 2
```

**注意__init__()里面的value没有被更新, 即类变量更新了, 成员变量没更新**

**只reload逻辑, 不reload数据**

## 代码实现

可以在原来程序中增加对外调用接口(如PRC或者Restful API)

```
import time
import sys, os

def auto_reload():
    mods = ["my_config"] # the need reload modules

    for mod in mods:
        try:
            module = sys.modules[mod]
        except:
            continue

        filename = module.__file__

        if filename.endswith(".pyc"):
            filename = filename.replace(".pyc", ".py")
        mod_time = os.path.getmtime(filename)
        if not "loadtime" in module.__dict__:
            module.loadtime = 0 # first load's time  1*

        try:
            if mod_time > module.loadtime:
                reload(module)
        except:
            pass

        module.loadtime = mod_time
```
