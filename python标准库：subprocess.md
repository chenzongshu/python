

>subprocess意在替代其他几个老的模块或者函数，比如：os.system os.spawn* os.popen* popen2.* commands.*

**subprocess的目的就是启动一个新的进程并且与之通信。**

subprocess最简单的用法就是调用shell命令了,另外也可以调用程序,并且可以通过stdout,stdin和stderr进行交互。

# subprocess主类
subprocess模块中只定义了一个类: Popen。可以使用Popen来创建进程，并与进程进行复杂的交互。
```
subprocess.Popen(
      args, 
      bufsize=0, 
      executable=None,
      stdin=None,
      stdout=None, 
      stderr=None, 
      preexec_fn=None, 
      close_fds=False, 
      shell=False, 
      cwd=None, 
      env=None, 
      universal_newlines=False, 
      startupinfo=None, 
      creationflags=0)
```
- args可以是字符串或者序列类型（如：list，元组），用于指定进程的可执行文件及其参数。如果是序列类型，第一个元素通常是可执行文件的路径。我们也可以显式的使用executeable参数来指定可执行文件的路径。
- bufsize：指定缓冲。0 无缓冲,1 行缓冲,其他 缓冲区大小,负值 系统缓冲(全缓冲)
- stdin, stdout, stderr分别表示程序的标准输入、输出、错误句柄。他们可以是PIPE，文件描述符或文件对象，也可以设置为None，表示从父进程继承。
- preexec_fn只在Unix平台下有效，用于指定一个可执行对象（callable object），它将在子进程运行之前被调用。
- Close_sfs：在windows平台下，如果close_fds被设置为True，则新创建的子进程将不会继承父进程的输入、输出、错误管道。我们不能将close_fds设置为True同时重定向子进程的标准输入、输出与错误(stdin, stdout, stderr)。
- shell设为true，程序将通过shell来执行。
- cwd用于设置子进程的当前目录
- env是字典类型，用于指定子进程的环境变量。如果env=None，子进程的环境变量将从父进程中继承。
- Universal_newlines:不同操作系统下，文本的换行符是不一样的。如：windows下用'/r/n'表示换，而Linux下用'/n'。如果将此参数设置为True，Python统一把这些换行符当作'/n'来处理。
- startupinfo与createionflags只在windows下用效，它们将被传递给底层的CreateProcess()函数，用于设置子进程的一些属性，如：主窗口的外观，进程的优先级等等。

# Popen方法
- Popen.poll()：用于检查子进程是否已经结束。设置并返回returncode属性。
- Popen.wait()：等待子进程结束。设置并返回returncode属性。
- Popen.communicate(input=None)：与子进程进行交互。向stdin发送数据，或从stdout和st- derr中读取数据。可选参数input指定发送到子进程的参数。Communicate()返回一个元组：(stdoutdata, stderrdata)。注意：如果希望通过进程的stdin向其发送数据，在创建Popen对象的时候，参数stdin必须被设置为PIPE。同样，如果希望从stdout和stderr获取数据，必须将stdout和stderr设置为PIPE。
- Popen.send_signal(signal)：向子进程发送信号。
- Popen.terminate()：停止(stop)子进程。在windows平台下，该方法将调用Windows API TerminateProcess（）来结束子进程。
- Popen.kill()：杀死子进程。
- Popen.stdin：如果在创建Popen对象是，参数stdin被设置为PIPE，Popen.stdin将返回一个文件对象用于策子进程发送指令。否则返回None。
- Popen.stdout：如果在创建Popen对象是，参数stdout被设置为PIPE，Popen.stdout将返回一个文件对象用于策子进程发送指令。否则返回None。
- Popen.stderr：如果在创建Popen对象是，参数stdout被设置为PIPE，Popen.stdout将返回一个文件对象用于策子进程发送指令。否则返回None。
- Popen.pid：获取子进程的进程ID。
- Popen.returncode：获取进程的返回值。如果进程还没有结束，返回None。
subprocess.call(*popenargs,**kwargs)：运行命令。该函数将一直等待到子进程运行结束，并返回进程的returncode。文章一开始的例子就演示了call函数。如果子进程不需要进行交互,就可以使用该函数来创建。
- subprocess.check_call(*popenargs, **kwargs)：与subprocess.call(*popenargs, **kwargs)功能一样，只是如果子进程返回的returncode不为0的话，将触发CalledProcessError异常。在异常对象中，包括进程的returncode信息。

# 程序中运行其他程序
可以这样写
```
subprocess.Popen('脚本/shell', shell=True)
```
也可以这样

```
subprocess.call('脚本/shell', shell=True)
```

两者的区别是前者无阻塞,会和主程序并行运行,后者必须等待命令执行完毕,如果想要前者编程阻塞可以这样

```
s = subprocess.Popen('脚本/shell', shell=True)
s.wait()
```

# 程序返回运行结果
有时候我们需要程序的返回结果,可以这样做

```
>>> s = subprocess.Popen('ls -l', shell=True, stdout=subprocess.PIPE) 
>>> s.communicate() 
>>> res=s.communicate()[0]
('\xe6\x80\xbb\xe7\x94\xa8\xe9\x87\x8f 152\n-rw------- 1 limbo limbo   808  7\xe6\x9c\x88  6 17:46 0000-00-00-welcome-to-jekyll.markd (......)')
```
如果是多个命令，管道相连

```
child1 = subprocess.Popen(["cat","/etc/passwd"], stdout=subprocess.PIPE)
child2 = subprocess.Popen(["grep","0:0"],stdin=child1.stdout, stdout=subprocess.PIPE)
out = child2.communicate()[0]
print out
```

它会返回一个元组：(stdoutdata, stderrdata)

checkout更简单,但前者可以实现更多的交互,如stderr和stdin,但是在前面调用Popen的时候要实现定义Popen(stdin=subprocess.PIPE, stderr=subprocess)

# Call方法

Call方法只能执行命令,不能获取返回值

# checkout方法

它会返回stdout
```
>>> s = subprocess.check_output('ls -l', shell=True)
>>> s
('\xe6\x80\xbb\xe7\x94\xa8\xe9\x87\x8f 152\n-rw------- 1 limbo limbo   808  7\xe6\x9c\x88  6 17:46 0000-00-00-welcome-to-jekyll.markd (......)')
```
