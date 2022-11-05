# 1 ArgumentParser()的用法

argparse是一个python模块：命令行选项、参数和子命令解析器，argparse将会从sys.argv解析出这些参数。argparse模块还会自动生成帮助和使用手册，并在用户给程序传入无效参数是报出错误信息。

主要有四个步骤：

* import argparse

* 创建ArgumentParser()对象
* 调用add_argument()方法添加参数
* 使用parse_args()解析添加的参数

## 1.1 创建解析器

```python
parser = argparse.ArgumentParser(description="Process some integers")
```

使用argparse的第一步是创建一个ArgumentParser对象

ArgumentParse对象包含将命令解析成python数据类型所需的全部信息。

### 1.1.1 ArgumentParser对象

```python
class argparse.ArgumentParser(prog=None, usage=None, description=None, epilog=None, parents=[], formatter_class=argparse.HelpFormatter, prefix_chars='-', fromfile_prefix_chars=None, argument_default=None, conflict_handler='error', add_help=True, allow_abbrev=True)
```

* prog - 程序的名称（默认：sys.argv[0]）
* usage - 描述程序用途的字符串（默认值：从添加到解析器的参数生成）
* description - 在参数帮助文档之前显示的文本（默认值：无）
* epilog - 在参数帮助文档之后显示的文本（默认值：无）
* parents - 一个 ArgumentParser 对象的列表，它们的参数也应包含在内
* formatter_class - 用于自定义帮助文档输出格式的类
* prefix_chars - 可选参数的前缀字符集合（默认值：’-’）
* fromfile_prefix_chars - 当需要从文件中读取其他参数时，用于标识文件名的前缀字符集合（默认值：None）
* argument_default - 参数的全局默认值（默认值： None）
* conflict_handler - 解决冲突选项的策略（通常是不必要的）
* add_help - 为解析器添加一个 -h/–help 选项（默认值： True）
* allow_abbrev - 如果缩写是无歧义的，则允许缩写长选项 （默认值：True）

## 1.2 添加参数

```python
ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])
```

* name or flags - 一个命名或者一个选项字符串的列表，例如 foo 或 -f, --foo。

* action - 当参数在命令行中出现时使用的动作基本类型，默认是store。

  * action='append'：将多次输入的参数保存到一个list中

    * ```python
      parser.add_argument('-test', action='append')
      print(args.test)
      
      命令行：python test.py -test 12 -test 13
      命令行：['12', '13']
      ```

  * action='count'：存储遇到参数命令的次数

    * ```python
      parser.add_argument('-test',action='count')
      print(args)
      
      命令行：python test.py -test
      命令行：Namespace<test=1>
      
      命令行：python test.py -test -test
      命令行：Namespace<test=2>
      ```

  * action='append_const'：类似于append使用，这次是使用const值，而不是输入给定

    * ```
      parser.add_argument('-test',action='append_const',const=1)
      parser.add_argument('-int', action='append_const',const=12.3)
      
      print(args)
      
      命令行：python test.py -test -int
      命令行：Namespace<int=[12.3], test=[1]>
      ```

    * 

  * action = 'store'表示存储参数值

  * action='store_true'/'store_false'，store_true就代表一旦有这个参数，做出动作“将其值标为True”，也就是没有时，默认状态下其值为False。反之亦然，store_false也就是默认为True，一旦命令中有此参数，其值则变为False。

  * action='store_const'，表示赋值为const

    * >```python
      >parser.add_argument('-test', action='store_const', const=12)
      >print(args.test)
      >
      >#action="store_const"不能与type一起使用
      >命令行：python test.py -test
      >output:12
      >命令行：python test.py -test 13 # 报错
      >```

* nargs - 命令行参数的数量

  * nargs=N(N是一个大于等于1的整数)

    * ```
      parser.add_argument('test',type=int,nargs=2)#该参数可以有两个输入
      print(args.test)
      
      python test.py -test 1 2
      [1, 2]
      ```

  * nargs='?'允许不指定参数，利用const中的常量给予值，这与default参数是不同的，default参数允许命令行输入其它值，而const是传入0个参数时赋予const值

    * ```
      parser.add_argument('-test',type=int, const=10, nargs='?')#允许不输入，也可以输入1个值
      print(args.test)
      
      python test.py -test
      10
      ```

    * 

* const - 被一些 action 和 nargs 选择所需求的常数。

* default - 当参数未在命令行中出现时使用的值。

* type - 命令行参数应当被转换成的类型。

* choices - 可用的参数的容器。

* required - 此命令行选项是否可省略 （仅选项可用）。

* help - 一个此选项作用的简单描述。

* metavar - 在用户手册声明中的参数名称，对于必选参数默认就是参数名称，对于可选参数默认是全大写的参数名称

* dest - 解析后的参数名称，默认情况下，对于可选参数选取最长的名称，中下划线转换为下划线。

## 1.3 解析参数

```
args = parser.parse_args(['--sum', '7', '-1', '42'])
```

# 2 os模块

## 2.1 os.listdir()

os.listdir()方法用于返回指定的文件夹包含的文件或文件夹的名字列表

它不包括.和..即使它在文件夹中。只支持在Unix和Windows下使用

### 2.2 语法

```
os.listdir(path)
```

**参数**

* path –需要列出的目录路径

**返回值**

返回指定路径下的文件和文件夹列表。

**实例**

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os, sys

# 打开文件
path = "/var/www/html/"
dirs = os.listdir( path )

# 输出所有文件和文件夹
for file in dirs:
   print (file)
```

执行以上程序输出结果为：

```python
test.htm
stamp
faq.htm
_vti_txt
robots.txt
itemlisting
resumelisting
writing_effective_resume.htm
advertisebusiness.htm
papers
resume
```

## 2.3 os.walk()

**概述**

os.walk()方法用于通过在目录树中游走输出在目录中的文件名，向上或向下。

os.walk()方法是一个简单易用的文件、目录遍历器，可以帮助我们高效的处理文件、目录方面的事情。

在Unix、Window中有效。

**语法**

walk()方法语法格式如下：

```python
os.walk(top[, topdown=True[, onerror=None[, followlinks=False]]])
```

**参数**

* top –是你所要遍历的目录地址，返回的是一个三元组（root, dirs, files）。
  * root所指的是当前正在遍历的这个文件夹的本身的地址
  * dirs是一个list，内容是该文件夹中所有目录的名字（不包括子目录）。
  * files同样是list，内容是该文件中所有的文件（不包括子目录）。
* topdown –可选，为True，则优先遍历top目录，否则优先遍历top的子目录（默认为开启）。如果topdown参数为true，walk会遍历top文件夹，与top文件夹中的每一个子目录。
* onerror –可选，需要一个callable对象，当walk需要异常时，会调用。
* followlinks –可选，如果为True，则会遍历目录下的快捷方式（linux下是软链接symbolic link）实际所指的目录（默认关闭）。

**返回值**

该函数没有返回值，会使用yield关键字抛出一个存放当前该层目录（root,dirs,files）的三元组，最终将所有目录层的结果变为一个生成器。

**实例**1

![](G:\images\70)

```pyghon
print(list(os.walk('/mnt',topdown = False)))
```

输出：

```python
[('/mnt', ['dira', 'dirb'], ['file1', 'file2']), ('/mnt/dira', [], ['file_test_A']), ('/mnt/dirb', [], ['file_test_B'])]
```



这个列表（因为我们将生成器强制转化为列表）中的每一个元素都是一个元组，每一个元组都是都是一层目录，元组中的元素又代表了当前目录层的路径(根据赋给形参top的值来决定是绝对路径还是相对路径)、当前目录下有的文件目录、和当前目录下的文件。
       当我们对生成器使用for循环时，会将每层一目录信息返回。

```python
for info in os.walk('/mnt', topdown=True):
    print(info)
    print('***分隔符***')
```

输出：

```python
('/mnt', ['dira', 'dirb'], ['file1', 'file2'])
***分隔符***
('/mnt/dira', [], ['file_test_A'])
***分隔符***
('/mnt/dirb', [], ['file_test_B'])
***分隔符***
```

**实例2**

```python
import os
def SearchAbsPath(dirname):
    dirname = os.path.abspath(dirname)
    filenames = list()
    for root,dirs,files in os.walk(dirname, topdown=False): #扫描一层目录
        for name in files:
            filenames.append(root+os.path.sep+name) #每一个文件的绝对路径放入列表
            print(root+os.path.sep+name)
    return filenames
SearchAbsPath('.')
```

输出：

```python
/mnt/dira/file_test_A
/mnt/dirb/file_test_B
/mnt/packeage/__init__.py
/mnt/file1
/mnt/file2
```

## 2.4 常用模块

```python
"""
python中os.path常用模块
os.path.sep:路径分隔符      linux下就用这个了’/’
os.path.altsep: 根目录
os.path.curdir:当前目录
os.path.pardir：父目录
os.path.abspath(path)：绝对路径
os.path.join():       常用来链接路径
os.path.split(path):      把path分为目录和文件两个部分，以列表返回
"""

 1 >>> import os
 2 >>> help('os.path.sep')
 3 
 4 >>> print os.path.sep #win和linux下的不一样，如果脚本在win下开发，在linux下面跑，可以这个，避免报错
 5 /
 6 >>> print os.path.altsep
 7 None
 8 >>> print os.path.curdir
 9 .
10 >>> print os.path.abspath('/root')
11 /root
12 >>> print os.path.abspath('/root/pp')
13 /root/pp
14 >>> print os.path.abspath('/root/pp/f.c')
15 /root/pp/f.c
16 >>> print os.path.split('/root/pp/f.c')
17 ('/root/pp', 'f.c')

```

#  3 python进度条tqdm详细介绍

有时候在使⽤Python处理⽐较耗时操作的时候，为了便于观察处理进度，这时候就需要通过进度条将处理情况进⾏可视化展⽰，以便我们能够及时了解情况。这对于第三⽅库⾮常丰富的Python来说，想要实现这⼀功能并不是什么难事。tqdm就能⾮常完美的⽀持和解决这些问题，可以实时输出处理进度⽽且占⽤的CPU资源⾮常少，⽀持windows、Linux、mac等系统，⽀持循环处理、多进程、递归处理、还可以结合linux的命令来查看处理情况，等进度展⽰。

## 3.1 使用tqdm子模块

* 基于迭代类型

```python
# 向tqdm中传入迭代类型即可
from tqdm import tqdm
import time

text = ""
for char in tqdm(["a", "b", "c", "d"]):
    time.sleep(0.25)
    text = text + char

```

结果：

![](G:\images\python\ac348d0bf37bfd126ef7224ef34bcfd1.gif)

* 手动更新进度

```python
# 使用with语句来控制tqdm的更新
with tqdm(total=100) as pbar:
	for i in range(10):
		time.sleep(0.01)
		pbar.update(1)	# 每次更新多少

# 当然也可以不用with，将tqdm赋值给一个变量
pbar = tqdm(total=100)
for i in range(100):
	time.sleep(0.1)
	pbar.update(10)
pbar.close() # 注意这样使用之后必须调用del，或者close方法删除该变量
```

效果如下：

![](G:\images\python\3b65c548ebacfcf1e2eb5ad7e609fe17.gif)



* 在命令行中使用

  ```python
  # 将输出的结果内容通过管道传送给 tqdm
  time find . -name '*.py' -type f -exec cat \{} \; | tqdm | wc -l
  ```

  效果如下：

  ![](G:\images\python\881571c0fb01580e006c74c7d646ca39.gif)

<img src="G:\images\python\983e7bb09235edb4f03e1927d25cc88f.png" style="zoom:25%;" />
