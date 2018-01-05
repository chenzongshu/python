
# JSON介绍

JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。独立于程序语言。

## JSON结构

json简单说就是javascript中的对象和数组，所以这两种结构就是对象和数组两种结构，通过这两种结构可以表示各种复杂的结构

- 对象：对象在js中表示为“{}”括起来的内容，数据结构为{key：value,key：value,...}的键值对的结构，在面向对象的语言中，key为对象的属性，value为对应的属性值，所以很容易理解，取值方法为 对象.key 获取属性值，这个属性值的类型可以是 数字、字符串、数组、对象几种。

- 数组：数组在js中是中括号“[]”括起来的内容，数据结构为["java","javascript","vb",...]，取值方式和所有语言中一样，使用索引获取，字段值的类型可以是 数字、字符串、数组、对象几种。


# python操作JSON

有两种操作，一种是把JSON格式的字符串转成python的各种数据类型，一种相反
不过都需要包含JSON包，`import json`

## 把Python数据格式转为json字符串
有两种函数,一个是dump,一个是dumps,区别在于dumps操作字符串,dump操作文件句柄
比如现在我们有个字典,可以转换为字符串

```
data = {
'name' : 'ACME',
'shares' : 100,
'price' : 542.23
}

json_str = json.dumps(data)
```

## 把json字符串转为Python数据格式
同样有两种函数,一个是loads,操作字符串;一个是load,操作文件句柄
比如我们现在有个字符串:

```
str = '[{"deleted": true, "dataset_id": "68be75bd-54cf-48eb-8214-8d178d7b6f37", "maximum_size": 83886080, "primary": "e7b72634-0cc4-4608-8d5c-bc26516dae97", "metadata": {"name": "pfn4-wksp-test-10-0"}},{"deleted": true, "dataset_id": "a05b8b5f-6723-4419-9a03-88ae030034de", "maximum_size": 83886080, "primary": "e7b72634-0cc4-4608-8d5c-bc26516dae97", "metadata": {"name": "pfn4-wksp-zxd-4"}}]'

data = json.loads(str)

print data[0]['metadata']['name']  #输出为pfn4-wksp-test-10-0

```
当调用loads函数之后,把字符串转换为了data,这是一个数组,数组里面含有2个字典,可以使用嵌套的属性来获取对应的值

注意,**python的json模块不支持单引号**,所以类似`"{'a':'A','b':[2,4],'c':3.0}"`字符串转换的时候会报错,只要转成双引号即可

# Linux 格式化查看JSON文件

```
cat xxx.json | python -m json.tool
```
