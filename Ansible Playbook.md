# 介绍

ansible是相当经典的运维工具了， 其中生产中使用最多的就是playbook

playbook简单来说就是把主机资源和操作弄成一个yaml文件，让我们可以同步或者异步执行，记住， 这不是一种语法， 只是一种配置

# Playbook语法

## 语法

- 在单一一个`playbook`文件中，可以连续三个连子号(`---`)区分多个`play`。还有选择性的连续三个点好(`...`)用来表示`play`的结尾，也可省略。
- 次行开始正常写`playbook`的内容，一般都会写上描述该`playbook`的功能。
- 使用#号注释代码。
- 缩进必须统一，不能空格和`tab`混用。
- 缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行实现的。
- `YAML`文件内容和`Linux`系统大小写判断方式保持一致，是区分大小写的，`k/v`的值均需大小写敏感
- `k/v`的值可同行写也可以换行写。同行使用:分隔。
- `v`可以是个字符串，也可以是一个列表
- 一个完整的代码块功能需要最少元素包括 `name: task`



先看一个简单例子

```
[root@ansible ~]# cat playbook01.yml
---                       #固定格式
- hosts: 192.168.1.31     #定义需要执行主机
  remote_user: root       #远程用户
  vars:                   #定义变量
    http_port: 8088       #变量

  tasks:                             #定义一个任务的开始
    - name: create new file          #定义任务的名称
      remote_user: test              #可以在某一个tasks中定义要执行该任务的远程用户
      file: name=/tmp/playtest.txt state=touch   #调用模块，具体要做的事情
    - name: create new user
      user: name=test02 system=yes shell=/sbin/nologin
    - name: install package
      sudo_user: test                #可以定义使用sudo授权用户执行该任务
      sudo: yes    
      yum: name=httpd
    - name: config httpd
      template: src=./httpd.conf dest=/etc/httpd/conf/httpd.conf
      notify:                 #定义执行一个动作(action)让handlers来引用执行，与handlers配合使用
        - restart apache      #notify要执行的动作，这里必须与handlers中的name定义内容一致
    - name: copy index.html
      copy: src=/var/www/html/index.html dest=/var/www/html/index.html
    - name: start httpd
      service: name=httpd state=started
  handlers:                                    #处理器：更加tasks中notify定义的action触发执行相应的处理动作
    - name: restart apache                     #要与notify定义的内容相同
      service: name=httpd state=restarted      #触发要执行的动作
      
```

- hosts: 指定哪些主机或者主机组可以来执行下面的命令

- Tasks 任务集

- Varniables 内置变量或自定义变量在playbook中调用

- Templates 模板，即使用模板语法的文件，比如配置文件等

- Handlers 和notity结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行

  > 当发生改动时，`notify actions`会在`playbook`的每一个task结束时被触发，而且即使有多个不同task通知改动的发生，`notify actions`知会被触发一次；比如多个`resources`指出因为一个配置文件被改动，所以`apache`需要重启，但是重新启动的操作知会被执行一次。

- tags 标签，指定某条任务执行，用于选择运行playbook中的部分代码。

  

## 运行

通过`ansible-playbook`命令运行

```
[root@ansible PlayBook]# ansible-playbook -h
#ansible-playbook常用选项：
--check  or -C    #只检测可能会发生的改变，但不真正执行操作
--list-hosts      #列出运行任务的主机
--list-tags       #列出playbook文件中定义所有的tags
--list-tasks      #列出playbook文件中定义的所以任务集
--limit           #主机列表 只针对主机列表中的某个主机或者某个组执行
-f                #指定并发数，默认为5个
-t                #指定tags运行，运行某一个或者多个tags。（前提playbook中有定义tags）
-v                #显示过程  -vv  -vvv更详细
```

## 变量

1、执行`playbook`时候通过参数`-e`传入变量，这样传入的变量在整个`playbook`中都可以被调用，属于全局变量

```
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: install pkg
      yum: name={{ pkg }}

#执行playbook 指定pkg
[root@ansible PlayBook]# ansible-playbook -e "pkg=httpd" variables.yml
```

2、编写`playbook`时，直接在里面定义变量，然后直接引用，可以定义多个变量；注意：如果在执行`playbook`时，又通过`-e`参数指定变量的值，那么会以`-e`参数指定的为准。

```
# 编辑playbook
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root
  vars:                #定义变量
    pkg: nginx         #变量1
    dir: /tmp/test1    #变量2

  tasks:
    - name: install pkg
      yum: name={{ pkg }} state=installed    #引用变量
    - name: create new dir
      file: name={{ dir }} state=directory   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml

# 如果执行时候又重新指定了变量的值，那么会已重新指定的为准
[root@ansible PlayBook]# ansible-playbook -e "dir=/tmp/test2" variables.yml
```

3、`setup`模块默认是获取主机信息的，有时候在`playbook`中需要用到，所以可以直接调用。

```
# 编辑playbook文件
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: create file
      file: name={{ ansible_fqdn }}.log state=touch   #引用setup中的ansible_fqdn


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml
```

4、为了方便管理将所有的变量统一放在一个独立的变量`YAML`文件中，`laybook`文件直接引用文件调用变量即可。

```
# 定义存放变量的文件
[root@ansible PlayBook]# cat var.yml 
var1: vsftpd
var2: httpd

# 编写playbook
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root
  vars_files:    #引用变量文件
    - ./var.yml   #指定变量文件的path（这里可以是绝对路径，也可以是相对路径）

  tasks:
    - name: install package
      yum: name={{ var1 }}   #引用变量
    - name: create file
      file: name=/tmp/{{ var2 }}.log state=touch   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook  variables.yml
```

## 标签

一个`playbook`文件中，执行时如果想执行某一个任务，那么可以给每个任务集进行打标签，这样在执行的时候可以通过`-t`选择指定标签执行，还可以通过`--skip-tags`选择除了某个标签外全部执行等。

```
# 编辑playbook
[root@ansible PlayBook]# cat httpd.yml 
---
- hosts: 192.168.1.31
  remote_user: root

  tasks:
    - name: install httpd
      yum: name=httpd state=installed
      tags: inhttpd

    - name: start httpd
      service: name=httpd state=started
      tags: sthttpd

    - name: restart httpd
      service: name=httpd state=restarted
      tags: 
        - rshttpd
        - rs_httpd

# 通过-t指定tags名称，多个tags用逗号隔开
[root@ansible PlayBook]# ansible-playbook -t rshttpd httpd.yml 

```

通过`--skip-tags`选项排除不执行的`tags`

```
[root@ansible PlayBook]# ansible-playbook --skip-tags inhttpd httpd.yml 
```

## 模板

`template`模板为我们提供了动态配置服务，使用`jinja2`语言，里面支持多种条件判断、循环、逻辑运算、比较操作等。其实说白了也就是一个文件，和之前配置文件使用`copy`一样，只是使用`copy`，不能根据服务器配置不一样进行不同动态的配置。这样就不利于管理。
说明：
1、多数情况下都将`template`文件放在和`playbook`文件同级的`templates`目录下（手动创建），这样`playbook`文件中可以直接引用，会自动去找这个文件。如果放在别的地方，也可以通过绝对路径去指定。
2、模板文件后缀名为`.j2`

```
[root@ansible PlayBook]# cat testtmp.yml 
#模板示例
---
- hosts: all
  remote_user: root
  vars:
    - listen_port: 88    #定义变量

  tasks:
    - name: Install Httpd
      yum: name=httpd state=installed
    - name: Config Httpd
      template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf    #使用模板
      notify: Restart Httpd
    - name: Start Httpd
      service: name=httpd state=started
      
  handlers:
    - name: Restart Httpd
      service: name=httpd state=restarted
```

模板文件是`httpd`配置文件，这里配置文件端口使用了变量

```
[root@ansible PlayBook]# cat templates/httpd.conf.j2 |grep ^Listen
Listen {{ listen_port }}
```

模板还有很多高级用法， 这里不展开了