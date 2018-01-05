
# 什么是正则表达式

正则表达式是一种方法,用来匹配字符串中的指定文本,它是一种独特的让人眼花缭乱的规则,从Unix系统而来,在各个语言中都有对应的接口函数,在各个语言中语法差不多

当然,你也可以用一些脚本或者语言中的自带函数,如python中`find`来实现对应功能,不过代码会复杂一些,逼格也没那么高

如下面字符串,我们需要从中找出"hello world",不用正则的话

```
s = <html><body><h1>hello world<h1></body></html>
start_index = s.find('<h1>')

#然后找到下一个<h1>,如果有多个还需要循环
```

# 基本语法

举个栗子

```
import re

key = r"<html><body><h1>hello world<h1></body></html>"#这段是你要匹配的文本
p1 = r"(?<=<h1>).+?(?=<h1>)" #这是我们写的正则表达式规则，你现在可以不理解啥意思
pattern1 = re.compile(p1)    #我们在编译这段正则表达式
matcher1 = re.search(pattern1,key)   #在源文本中搜索符合正则表达式的部分
print matcher1.group(0)      #打印出来
```

看着是不是有点头晕,下面我们来了解基本语法

- 特殊字符'^'和'\$'
  分别表示一个字符串的开头和结尾
> "^The"：表示所有以"The"开始的字符串（"There"，"The cat"等）；
"of despair \$"：表示所以以"of despair"结尾的字符串；
"^abc\$"：表示开始和结尾都是"abc"的字符串——呵呵，只有"abc"自己了；
"notice"：表示任何包含"notice"的字符串。

- '*'，'+'和'?'
  表示一个或一序列字符重复出现的次数。它们分别表示“没有或更多”，“一次或更多”还有“没有或一次”
> "ab*"：表示一个字符串有一个a后面跟着零个或若干个b。（"a", "ab", "abbb",……）；
"ab+"：表示一个字符串有一个a后面跟着至少一个b或者更多；
"ab?"：表示一个字符串有一个a后面跟着零个或者一个b；
"a?b+$"：表示在字符串的末尾有零个或一个a跟着一个或几个b。

- 大括号{}
  用大括号括起，用以表示重复次数的范围
> "ab{2}"：表示一个字符串有一个a跟着2个b（"abb"）；
"ab{2,}"：表示一个字符串有一个a跟着至少2个b；
"ab{3,5}"：表示一个字符串有一个a跟着3到5个b。

请注意，你必须指定范围的下限（如："{0,2}"而不是"{,2}"）。还有，你可能注意到了，'*'，'+'和'?'相当于"{0,}"，"{1,}"和"{0,1}"。

- 或"|"操作
  这个很好理解,注意用括号表示其范围
> "hi¦hello"：表示一个字符串里有"hi"或者"hello"；
"(b¦cd)ef"：表示"bef"或"cdef"；
"(a¦b)*c"：表示一串"a""b"混合的字符串后面跟一个"c"；

- "."号
  可以替代任何字符
> "a.[0-9]"：表示一个字符串有一个"a"后面跟着一个任意字符和一个数字；
"^.{3}$"：表示有任意三个字符的字符串（长度为3个字符）；

- []方括号
  表示某些字符允许在一个字符串中的某一特定位置出现
> "[ab]"：表示一个字符串有一个"a"或"b"（相当于"a¦b"）；
"[a-d]"：表示一个字符串包含小写的'a'到'd'中的一个（相当于"a¦b¦c¦d"或者"[abcd]"）；
"^[a-zA-Z]"：表示一个以字母开头的字符串；
"[0-9]%"：表示一个百分号前有一位的数字；
",[a-zA-Z0-9]$"：表示一个字符串以一个逗号后面跟着一个字母或数字结束。

你也可以在方括号里用'^'表示不希望出现的字符，'^'应在方括号里的第一位。（如："%[^a-zA-Z]%"表示两个百分号中不应该出现字母）。

请注意在方括号中，不需要转义字符。

- 特殊用法
  为了方便我们写简洁的正则表达式，它本身还提供下面这样的写法

| 正则表达式	| 代表的匹配字符 |
| -------- | ----- |
| [0-9]	| 0123456789任意之一 |
| [a-z]	| 小写字母任意之一 |
| [A-Z]	| 大写字母任意之一 |
| \d	| 等同于[0-9] |
| \D	| 等同于[^0-9]匹配非数字 |
| \w	| 等同于[a-z0-9A-Z_]匹配大小写字母、数字和下划线 |
| \W	| 等同于[^a-z0-9A-Z_]等同于上一条取非 |
| \s    | 空白字符,空格,等同于["t"n"r"f"v] |
| \S    | 非空白字符,等同于[^"t"n"r"f"v] |


# 贪婪模式

默认是使用贪婪模式,什么是贪婪模式呢,顾名思义,就是尽可能多,下面也来举例说明下

```
import re

key = r"chuxiuhong@hit.edu.cn"
p1 = r"@.+\."#我想匹配到@后面一直到“.”之间的，在这里是hit
pattern1 = re.compile(p1)
print pattern1.findall(key)
```

输出结果为

```
['@hit.edu.']
```

“+”代表是字符重复一次或多次。但是我们没有细说这个多次到底是多少次,所以它尽可能多的匹配了.
只要在“+”后面加一个“？”就可以解决这个问题了,改为了懒惰模式.

# python中的正则
python中使用re模块,可以方便的使用正则(C和C++不含原生正则库)

## compile
re.compile(strPattern[, flag]):  

将字符串形式的正则表达式编译为Pattern对象。第二个参数flag是匹配模式，取值可以使用按位或运算符'|'表示同时生效，比如re.I|re.M。另外，你也可以在regex字符串中指定模式，比如re.compile('pattern', re.I | re.M)与re.compile('(?im)pattern')是等价的。 

模式有

- re.I(re.IGNORECASE): 忽略大小写（括号内是完整写法，下同）
- M(MULTILINE): 多行模式，改变'^'和'$'的行为（参见上图）
- S(DOTALL): 点任意匹配模式，改变'.'的行为
- L(LOCALE): 使预定字符类 \w \W \b \B \s \S 取决于当前区域设定
- U(UNICODE): 使预定字符类 \w \W \b \B \s \S \d \D 取决于unicode定义的字符属性
- X(VERBOSE): 详细模式。这个模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。以下两个正则表达式是等价的：

## Pattern
Pattern对象是一个编译好的正则表达式，通过Pattern提供的一系列方法可以对文本进行匹配查找。

Pattern不能直接实例化，必须使用re.compile()进行构造,方法如下:

### 1.match(string[, pos[, endpos]]) | re.match(pattern, string[, flags]): 
这个方法将从string的pos下标处起尝试匹配pattern；如果pattern结束时仍可匹配，则返回一个**Match对象**；如果匹配过程中pattern无法匹配，或者匹配未结束就已到达endpos，则返回None。 

pos和endpos的默认值分别为0和len(string)；re.match()无法指定这两个参数，参数flags用于编译pattern时指定匹配模式。 
注意：这个方法并不是完全匹配。当pattern结束时若string还有剩余字符，仍然视为成功。想要完全匹配，可以在表达式末尾加上边界匹配符'$'。 


### 2.search(string[, pos[, endpos]]) | re.search(pattern, string[, flags]): 
这个方法用于查找字符串中可以匹配成功的子串。从string的pos下标处起尝试匹配pattern，如果pattern结束时仍可匹配，则返回一个Match对象；若无法匹配，则将pos加1后重新尝试匹配；直到pos=endpos时仍无法匹配则返回None。 

pos和endpos的默认值分别为0和len(string))；re.search()无法指定这两个参数，参数flags用于编译pattern时指定匹配模式。 

```
# encoding: UTF-8 
import re 

# 将正则表达式编译成Pattern对象 
pattern = re.compile(r'world') 

# 使用search()查找匹配的子串，不存在能匹配的子串时将返回None 
# 这个例子中使用match()无法成功匹配 
match = pattern.search('hello world!') 

if match: 
    # 使用Match获得分组信息 
    print match.group() 

### 输出 ### 
# world 
```
**原来group(0)，是所有匹配的内容，而group(N)指的是原先subgroup子组对应的内容，而subgroup是原先search等规则中，用括号()所括起来的。**

### 3.split(string[, maxsplit]) | re.split(pattern, string[, maxsplit]): 
按照能够匹配的子串将string分割后返回列表。maxsplit用于指定最大分割次数，不指定将全部分割。 

```
import re

p = re.compile(r'\d+')
print p.split('one1two2three3four4')

### output ###
# ['one', 'two', 'three', 'four', '']
```

### 4.findall(string[, pos[, endpos]]) | re.findall(pattern, string[, flags]): 
搜索string，以列表形式返回全部能匹配的子串。 

```
import re

p = re.compile(r'\d+')
print p.findall('one1two2three3four4')

### output ###
# ['1', '2', '3', '4']
```

### 5.finditer(string[, pos[, endpos]]) | re.finditer(pattern, string[, flags]): 
搜索string，返回一个顺序访问每一个匹配结果（Match对象）的迭代器。 

```
import re

p = re.compile(r'\d+')
for m in p.finditer('one1two2three3four4'):
    print m.group(),

### output ###
# 1 2 3 4
```

### 6.sub(repl, string[, count]) | re.sub(pattern, repl, string[, count]): 
使用repl替换string中每一个匹配的子串后返回替换后的字符串。 

当repl是一个字符串时，可以使用\id或\g<id>、\g<name>引用分组，但不能使用编号0。 
当repl是一个方法时，这个方法应当只接受一个参数（Match对象），并返回一个字符串用于替换（返回的字符串中不能再引用分组）。 
count用于指定最多替换次数，不指定时全部替换。 

```
import re

p = re.compile(r'(\w+) (\w+)')
s = 'i say, hello world!'

print p.sub(r'\2 \1', s)

def func(m):
    return m.group(1).title() + ' ' + m.group(2).title()

print p.sub(func, s)

### output ###
# say i, world hello!
# I Say, Hello World!
```

### 7.subn(repl, string[, count]) |re.sub(pattern, repl, string[, count]): 
返回 (sub(repl, string[, count]), 替换次数)。 

```
import re

p = re.compile(r'(\w+) (\w+)')
s = 'i say, hello world!'

print p.subn(r'\2 \1', s)

def func(m):
    return m.group(1).title() + ' ' + m.group(2).title()

print p.subn(func, s)

### output ###
# ('say i, world hello!', 2)
# ('I Say, Hello World!', 2)
```

## 前行后行断言

四种正则表达式的扩展匹配了，即所谓的“正向前行匹配”  (?=...)、“负向前行匹配” (?!...)、"正向后行匹配"(?<=...)、“负向后行匹配”(?<!...)。其中的...又可以是任意的合法正则匹配字符串

对于这4个断言的理解和记忆, 如下:

1.所谓的前行(lookahead)和后行(lookbehind)，
  其实就是向前看和向后看的意思。正则表达式引擎在执行字符串和表达式匹配时，会从头到尾（从前到后）连续扫描字符串中的字符，设想有一个扫描指针指向字符边界处并随匹配过程移动。前行断言，是当扫描指针位于某个位置时，引擎会尝试匹配指针还未扫过的字符，先于指针到达该字符，故称为前行。后行断言，引擎会尝试匹配指针已扫过的字符，后于指针到达该字符，故称为后行。
  
记忆方式：后行断言(?<=pattern)、(?<!pattern)中，有个像箭头一样的小于号，对于自左至右的文本方向，这个箭头是指向后的，这也比较符合我们的习惯。把小于号去掉，就是前行断言

2.所谓的正向(positive)和负向(negative)：
正向就表示匹配括号中的表达式，负向表示不匹配。

记忆方式：不等于(!=)、逻辑非(!)都是用!号来表示，所以有!号的形式表示不匹配、负向；将!号换成=号，就表示匹配、正向。

我们特别需要注意的一点是，对于后行方式的两种断言(?<=...)和(?<!...)，其中的...表达式所匹配的内容必须是定长的，这也就意味着后行断言的匹配表达式中是不能含有*、？和+符号的，这也就为后行断言的使用带来了一定的困难和麻烦

## 反斜杠陷阱
我们要在一个正则表达式里面匹配一个反斜杠"/",需要怎么写呢?

答案是"////",四个!

为什么是这样? 因为字符串和正则表达式中,"/"都是转义字符,所以,两个"/",在字符串中,被转义为一个"/",传给正则表达式,正则表达式表示一个字符"/", 需要的是两个"//",所以在源字符串中,必须就是四个'/',这样,传到正则表达式中,才会是"//"

可以使用python内置字符串r'//',这样,就不会经过字符串转义这层,就可以表达为斜杠字符啦
