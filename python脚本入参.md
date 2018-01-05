
有时候需要在执行python脚本的时候接收入参，所以特地总结了2中方法，有理论有实例

# 方法

## 1 arvg参数获取

```
import sys  

print sys.argv[0] ##脚本名  
print sys.argv[1] ## 第一个参数,后面以此类推

print len(sys.argv)  ##参数个数  
```

## 2 getopt模块获取

getopt模块是专门用来处理命令行参数的,里面的提供了2个函数和一个类，我们主要使用getopt函数,原型为:

```
def getopt(args, shortopts, longopts = [])： 
```
调用语句为:

```
opts, args = getopt.getopt(sys.argv[1:], "ho:", ["help", "output="]) 
```

## 参数解释
有两种格式的参数,一种为短参数,如"-h";另外一种是长参数,如"--version"
而且,参数后面又要区分,是否带了入参值,如查看help命令,只需要"-h",就是不需要带入参值得;如果是输入ip,如"-i 127.0.0.1",这个就是带了入参值得

短参数中,需要输入的如入参名后面加了冒号":",表明需要带参数,不加冒号则表明不要,
如上面的例子,短参数为`"ho:"`,表明有两个短参数,"-h","-o",其中"-o"需要后面接参数

长参数中,后面如果接了等号"=",表明需要带参数,不加则表明不要
如上面的例子,长参数为` ["help", "output="]`,表明有两个长参数,一个"--help",不需要后面带参数;一个"--output",需要后面带参数

## 返回值
调用getopt 函数。函数返回两个列表：opts 和args 。opts 为分析出的格式信息。args 为不属于格式信息的剩余的命令行参数(即那些不含"-/--"的参数)。opts 是一个两元组的列表。每个元素为：( 选项串, 附加参数) 。如果没有附加参数则为空串'' 。

如果附加参数为'-h -o file --help --output=out file1 file2' 
在分析完成后，opts 应该是：  [('-h', ''), ('-o', 'file'), ('--help', ''), ('--output', 'out')] 
而args 则为：  ['file1', 'file2'] 

# 实例

封装一个MySQL的SQL语句封装层,可以自动从配置文件里面读取数据库(主-主复制集群)IP

db.ini文件
```
[DB_IP]
db1=172.16.101.202
db2=172.16.101.203

[USER]
user=admin

[PASSWORD]
password=cgsl123

[DB]
database=test
```


```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import sys
import getopt
import MySQLdb
import ConfigParser

class mysql_opt():

    def __init__(self):
        config        = ConfigParser.ConfigParser()
        config.read("db.ini")

        db1_ip = config.get("DB_IP","db1")
        db2_ip = config.get("DB_IP","db2")
        user   = config.get("USER","user")
        pwd    = config.get("PASSWORD","password")
        db_name= config.get("DB", "database")
        try:
            self.db = MySQLdb.connect(db1_ip, user, pwd, db_name)
        except:
            self.db = MySQLdb.connect(db2_ip, user, pwd, db_name)
        #self.db = MySQLdb.connect("172.16.101.203","admin","cgsl123","test" )
        self.cursor = self.db.cursor()

    def __del__(self):
        self.db.close()

    def exec_sql(self,sql):
       
        print sql
        try:
            #cursor = self.db.cursor()
            # 执行sql语句
            self.cursor.execute(sql)
            # 提交到数据库执行
            self.db.commit()
        except Exception,e:
            # Rollback in case there is any error
            print "exec sql error"
            print Exception,":",e

            self.db.rollback()

    def create(self,table,args):

        sql = "create table " + table + "( " + args[0]+ " " + args[1] + " unique key" +","

        for para in args[2:-1]:
            if para in ["char","varchar"]:
                sql = sql + para + "(20)" + ","
            elif para in ["int","double","datetime"]:
                sql = sql + para + ","
            else:
                sql = sql + para + " "

        if args[-1] in ["char", "varchar"]:
            self.exec_sql(sql + args[-1]+ "(20)" +")")
        else:
            self.exec_sql(sql + args[-1] + ")")

    def drop(self,table):

        sql = "drop table " + table

        self.exec_sql(sql)

    def select(self,table):

        sql = "select * from " + table

        self.exec_sql(sql)

    def insert(self,table,args):

        sql = "insert into " + table + "("

        col = []
        value = []
        #print args
        for index in range(len(args)):
            if (index % 2) == 0:
                col.append(args[index])
            else:
                value.append(args[index])
        print col
        print value
        for c in range(len(col)):
            if c < len(col)-1:
                sql = sql + col[c] + ","
            else:
                sql = sql + col[c]

        sql = sql + ") values ("

        for v in range(len(value)):
            if v < len(value)-1:
                if col[v] in ["char","varchar"]:
                    sql = sql + '"' + value[v] + '"'  + ','
                else:
                    sql = sql + value[v] + ","
            else:
                if col[v] in ["char","varchar"]:
                    sql = sql + '"' + value[v] + '"' 
                else:
                    sql = sql + value[v]

        self.exec_sql(sql + ")")

    def update(self,table,args):

        sql = "update " + table + " set " + args[0] + "=" + args[1] + " where " + args[2] + "=" + args[3]

        self.exec_sql(sql)


def help():
    print " ****** command format ****** "
    print " [create table] ./test.py -c/--create table_name col1 type1 col2 type2...(first col is key)"
    print " [drop table]   ./test.py -d/--drop table_name "
    print " [select table] ./test.py -s/--select table_name "
    print " [insert table] ./test.py -s/--insert table_name col1 value1 col2 value2...(if value is string,use \"string\"),-i table id 4 name \"chen\"  "
    print " [update table] ./test.py -s/--update table_name col new_value row row_value (if value is string,use \"string\"),-u table name \"chen\" id 2 "


def main(argv):

    try:
        opts, args = getopt.getopt(argv, "hc:d:s:i:u:", ["help", "create=","drop=","select=","insert=","update=",])
    except:
        print "getopt error!"
        sys.exit(2)

    if len(opts)==0:
        help()

    db_conn = mysql_opt()

    for opt,arg in opts:
        if opt in ["-h","--help"]:
            help()
        elif opt in ["-c","--create"]:
            db_conn.create(arg,args)
        elif opt in ["-d","--drop"]:
            db_conn.drop(arg)
        elif opt in ["-s","--select"]:
            db_conn.select(arg)
        elif opt in ["-i", "--insert"]:
            db_conn.insert(arg,args)
        elif opt in ["-u", "--update"]:
            db_conn.update(arg,args)


if __name__ == '__main__':
    main(sys.argv[1:])


```

# argparse模块介绍
argparse是python标准库中的模块,可以完成对命令行的参数定义、解析以及后续的处理.

http://www.cnblogs.com/xiaofeiIDO/p/6154953.html 

主要有三个步骤:

## 1、创建 ArgumentParser() 对象

```
parser = argparse.ArgumentParser(description="some information here")
```
ArgumentParser函数有很多可选参数，prog、usage、description、epilog分别定义解析器的名称、使用说明、描述、最后的结尾描述

## 2、调用 add_argument() 方法添加参数

```
ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])
```

- name or flags - 选项字符串的名字或者列表，例如 foo 或者 -f, --foo。

- action - 命令行遇到参数时的动作，

a) 默认值是 store,只是保存参数的值
b) store_const，表示赋值为const；

```
>>> parser.add_argument('--foo', action='store_const', const=42)
>>> parser.parse_args('--foo'.split())
Namespace(foo=42)
```

c) append，将遇到的值存储成列表，也就是如果参数重复则会保存多个值;

```
>>> parser.add_argument('--foo', action='append')
>>> parser.parse_args('--foo 1 --foo 2'.split())
Namespace(foo=['1', '2'])
```

d) append_const，保存一个列表，并将const关键字参数指出的值附加在列表的后面。（注意const关键字参数默认是None。）'append_const' 动作在多个参数需要保存常量到相同的列表时特别有用

```
>>> parser.add_argument('--str', dest='types', action='append_const', const=str)
>>> parser.add_argument('--int', dest='types', action='append_const', const=int)
>>> parser.parse_args('--str --int'.split())
Namespace(types=[<type 'str'>, <type 'int'>])
```

e) count，计算关键字参数出现的次数；例如，这可用于增加详细的级别

```
>>> parser.add_argument('--verbose', '-v', action='count')
>>> parser.parse_args('-vvv'.split())
Namespace(verbose=3)
```

- nargs - 将一个动作与不同数目的命令行参数关联在一起，可以是具体的数字，或者是?号，当不指定值时对于 - Positional argument 使用 default，对于 Optional argument 使用 const；或者是 * 号，表示 0 或多个参数；或者是 + 号表示 1 或多个参数。

```
>>> parser.add_argument('--foo', nargs='*')
>>> parser.add_argument('--bar', nargs='*')
>>> parser.add_argument('baz', nargs='*')
>>> parser.parse_args('a b --foo x y --bar 1 2'.split())
Namespace(bar=['1', '2'], baz=['a', 'b'], foo=['x', 'y'])
```

如果没有提供nargs关键字参数，读取的参数个数取决于action。通常这意味着将读取一个命令行参数并产生一个元素（不是一个列表）。

- const - action 和 nargs 所需要的常量值。

- default - 不指定参数时的默认值。

- type - 默认情况下，ArgumentParser对象以简单字符串方式读入命令行参数。然而，很多时候命令行字符串应该被解释成另外一种类型，例如浮点数或者整数。add_argument()的type关键字参数允许任意必要的类型检查并作类型转换

- choices - 某些命令行参数应该从一个受限的集合中选择。这种情况的处理可以通过传递一个容器对象作为choices关键字参数给add_argument()。当解析命令行时，将检查参数的值，如果参数不是一个可接受的值则显示一个错误信息。

- required - argparse模块假定-f和--bar标记表示可选参数，它们在命令行中可以省略。如果要使得选项是必需的，可以指定True作为required=关键字参数的值给add_argument()： (仅针对可选参数)。

- help - 参数的帮助信息，当指定为 argparse.SUPPRESS 时表示不显示该参数的帮助信息.

- metavar - 当ArgumentParser生成帮助信息时，它需要以某种方式引用每一个参数。 默认情况下，ArgumentParser对象使用dest的值作为每个对象的“名字”。默认情况下，对于位置参数直接使用dest的值，对于可选参数则将dest的值变为大写。所以，位置参数dest='bar'将引用成bar。后面带有一个命令行参数的可选参数--foo将引用成FOO。

- dest - 大部分ArgumentParser动作给parse_args()返回对象的某个属性添加某些值。该属性的名字由add_argument()的dest关键字参数决定。对于位置参数的动作，dest 通常作为第一个参数提供给add_argument()：

恩,上面大串参数如果看着头晕,可以不看,下面解释和举例出常用的
在python脚本后面的参数,可以分为两类

- **定位参数**:
  即不需要带"-/--"之类就可以使用的，

```
parser = argparse.ArgumentParser()
parser.add_argument("square", help="display a square of a given number", type=int)
args = parser.parse_args()
print args.square**2
```
```
$ python argparse_usage.py 9    #不需要带什么 -x 之类的
81
```

- **可选参数**
即需要

```
parser = argparse.ArgumentParser()

parser.add_argument("-s","--square", help="display a square of a given number", type=int)
parser.add_argument("-c","--cubic", help="display a cubic of a given number", type=int)

args = parser.parse_args()

if args.square:
    print args.square**2

if args.cubic:
    print args.cubic**3
```
```
$ python argparse_usage.py --square 8
64

$ python argparse_usage.py --cubic 8
512
```

- **两种参数混合**

```
parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                   help='an integer for the accumulator')
parser.add_argument('--sum', dest='accumulate', action='store_const',
                   const=sum, default=max,
                   help='sum the integers (default: find the max)')

args = parser.parse_args()
print args.accumulate(args.integers)
```

```
$ python prog.py 1 2 3 4
4

$ python prog.py 1 2 3 4 --sum
10
```
> 调用parse_args()返回的对象将带有两个属性，integers和accumulate。属性integers将是一个包含一个或多个整数的列表，如果命令行上指定 --sum，那么属性accumulate将是sum()函数，如果没有指定，则是max()函数

- **子命令**
将多个命令组合进一个程序中，使用子解析器来处理命令行的每个部分。就像svn，以及OpenStack各个组件那样。

```
ArgumentParser.add_subparsers([title][, description][, prog][, parser_class][, action][, option_string][, dest][, help][, metavar])
```

```
parser = argparse.ArgumentParser()
 
subparsers = parser.add_subparsers(help='commands')
 
create_parser = subparsers.add_parser('create',help='Create a directory')

create_parser.add_argument('dirname',action='store',help='New directoryto create')

create_parser.add_argument('--read-only',default=False, action='store_true',
    help='Setpermissions to prevent writing to the directory',
    )
```



## 3、使用 parse_args() 解析添加的参数

使用函数parse_args()进行参数解析，这个函数的输入默认是sys.argv[1:]，也可以使用其他字符串列表,parse_args的返回值是一个包含命令参数的Namespace。所有参数以属性的形式存在，比如args.myoption


## 附:通用模板

```
import sys
 
def version_info():
    VERSION_INFO = 'PROG v1'
    AUTHOR_INFO = 'Author: crazyof.me'
    print VERSION_INFO
    print AUTHOR_INFO
 
def scan(host, ports, threads, useragent):
    print host
    print ports
    print threads
    print useragent
 
if __name__ == '__main__':
    parse = argparse.ArgumentParser()
    parse.add_argument('-v', '--version', action='store_true', default=False, help='Version information')
    parse.add_argument('-s', '--scan', action='store_true', default=False, help='Run masscan.')
    # parse.add_argument('-H', '--host', required=True, type=str, help='Scan this host or list of hosts')
    parse.add_argument('-H', '--host', type=str, help='Scan this host or list of hosts')
    parse.add_argument('-P', '--ports', type=str, default='21,22,80,443,8000,8080,8443,2080,2443,9090,6000,8888,50080,50443,5900', help='Specify ports in masscan format. (ie.0-1000 or 80,443...)')
    parse.add_argument('-t', '--threads', type=int, default=1, help='Specify the number of threads to use.')
    parse.add_argument('-a', '--ua', type=str, default='o_O O_o', help='Specify a User Agent for requests')
    args = parse.parse_args()
 
    if len(sys.argv) <= 1:
        parse.print_help()
        sys.exit(0)
 
    if args.version:  #因为「action='store_true'」，所以「args.version的值为True/False」
        version_info()
        sys.exit(0)
 
    if args.scan and (args.host is not None):  #因为为args.host设置了「type=str」，所以「args.host的值为None/设定的字符串的值」
        scan(args.host, args.ports, args.threads, args.ua)
```
