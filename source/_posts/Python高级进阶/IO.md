---
layout: post
title: Python高级进阶-IO
date: '2020-5-14 08:02'
categories: Python
tags: [Python]
description: Python学习
comments: true
---

# IO 
## 文件目录操作
Python内置的os模块提供了调用操作系统提供的接口函数。

### 环境变量
在操作系统中定义的环境变量，全部保存在os.environ这个变量中，可以直接查看：

```
>>> os.environ
environ({'VERSIONER_PYTHON_PREFER_32_BIT': 'no', 'TERM_PROGRAM_VERSION': '326', 'LOGNAME': 'michael', 'USER': 'michael', 'PATH': '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin:/usr/local/mysql/bin', ...})
要获取某个环境变量的值，可以调用os.environ.get('key')：

>>> os.environ.get('PATH')
'/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin:/usr/local/mysql/bin'
>>> os.environ.get('x', 'default')
'default'
```
### 文件和目录

```
# 查看当前目录的绝对路径:
>>> os.path.abspath('.')
'/Users/michael'
# 在某个目录下创建一个新目录，首先把新目录的完整路径表示出来:
>>> os.path.join('/Users/michael', 'testdir')
'/Users/michael/testdir'
# 然后创建一个目录:
>>> os.mkdir('/Users/michael/testdir')
# 删掉一个目录:
>>> os.rmdir('/Users/michael/testdir')
# 列出当前目录:
>>> os.listdir('.')
# 切换目录
>>> os.chdir('/')

# 对文件重命名:
>>> os.rename('test.txt', 'test.py')
# 删掉文件:
>>> os.remove('test.py')
```
在os模块中不存在复制文件！但是shutil模块提供了copyfile()的函数，你还可以在shutil模块中找到很多实用函数。
是shutil复制文件

```
>>>  shutil.copy('src','dst')
```

## StringIO和BytesIO

### StringIO
很多时候，数据读写不一定是文件，也可以在内存中读写。

StringIO顾名思义就是在内存中读写str。

要把str写入StringIO，我们需要先创建一个StringIO，然后，像文件一样写入即可：

```
>>> from io import StringIO
>>> f = StringIO()
>>> f.write('hello')
5
>>> f.write(' ')
1
>>> f.write('world!')
6
>>> print(f.getvalue())
hello world!
```

```
>>> from io import StringIO
>>> f = StringIO('Hello!\nHi!\nGoodbye!')
>>> while True:
...     s = f.readline()
...     if s == '':
...         break
...     print(s.strip())
...
Hello!
Hi!
Goodbye!
```

### BytesIO
StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：


```
>>> from io import BytesIO
>>> f = BytesIO()
>>> f.write('中文'.encode('utf-8'))
6
>>> print(f.getvalue())
b'\xe4\xb8\xad\xe6\x96\x87'
```

```
>>> from io import BytesIO
>>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
>>> f.read()
b'\xe4\xb8\xad\xe6\x96\x87'
```

## 文件读写


```
import os

try:
    f = open('/path/to/file', 'r')
    print(f.read()) # 一次性读取文件的全部内
    
    #for line in f.readlines():
    #   print(line.strip()) # 把末尾的'\n'删掉
    
    # read(size)方法，每次最多读取size个字节的内容
    
finally:
    if f:
        f.close()
try:      
    fd = os.open("dd.txt", 777)
    os.write(fd, "212")
finally:
    if fd:
        fd.close()
```

## 序列化  
**序列化:** 把内存中数据序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

**反序列化**：把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling。

Python提供了pickle模块来实现序列化。


```
>>> import pickle
>>> d = dict(name='Bob', age=20, score=88)
>>> pickle.dumps(d)
b'\x80\x03}q\x00(X\x03\x00\x00\x00ageq\x01K\x14X\x05\x00\x00\x00scoreq\x02KXX\x04\x00\x00\x00nameq\x03X\x03\x00\x00\x00Bobq\x04u.'

>>> f = open('dump.txt', 'wb')
>>> pickle.dump(d, f)
>>> f.close()
```

## JSON

Python内置的json模块提供了非常完善的Python对象到JSON格式的转换。


```
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)
'{"age": 20, "score": 88, "name": "Bob"}'
```

```
json_str = '{"age": 20, "score": 88, "name": "Bob"}'
json.loads(json_str)
```
