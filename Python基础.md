#### 数据类型（4种）

整数（int）、浮点数（float）、字符串（str）、布尔值（bool）

#### 数据结构（3种）

序列：成员都是有序排列，并且可以通过索引访问。

序列只是一个概念，具体的实现有三种：

字符串('abcd')、列表（[0,"abcd"]）、元组（（“abc”，“def”））

序列的基本操作

成员关系操作符：in、not in

连接操作符：+

重复操作符：*

切片操作符：[:]

元组存储的内容不可变更

列表存储的内容可以增删改

#### 条件语句

````python
if 表达式:
  代码块
  
if 表达式:
  代码块
elif 表达式:
  代码块
else:
  代码块
````

#### 循环语句

````python
while 表达式:
  代码块
  
for 迭代变量 in 可迭代对象:
  代码块
````

#### 字典

{"哈希值":"对象"}

{"length":80,"weight":80}

#### 特殊语法

列表推导式：

求从1到10所有偶数的平方

````python
[i*i for i in range(0,11) if(i%2)]
````

字典推导式:

```python
zidian = {}
{i:0 for i in zidian}
```

#### 文件内建函数

打开文件:open()

输入:read()

输入一行:readline()

文件内移动:seek()

输出:write()

关闭文件:close()

#### 错误和异常

````python
try:
  <监控异常>
except Exception[,reason]
	<异常处理代码>
finally:
  <无论异常是否发生都执行>
````

#### 函数

定义：

````python
def 函数名称:
  代码
	return 需要返回的内容
````

调用：

````python
函数名称()
````

可变长参数：

````python
def fun1(first,*others):
	return len(1 + others)
````

变量作用域：

````python
var1 = 123

def fun():
  var1 = 456
  print(var1)

fun()
print(var1)
````

此时输出为456，123。表明变量作用域只作用于fun内部。外部全局的var1不受影响。如果想修改外部的var1，可以在fun内部var1变量添加global关键字。

````python
var1 = 123

def fun():
  global var1
  var1 = 456
  print(var1)

fun()
print(var1)
````

#### 迭代器与生成器

迭代器：

````python
list = [1,2,3]
it = iteam(list)
print(next(it))
````

生成器：

````python
def frange(start,stop,step):
  x = start
  while x < step:
    yield x
    x += step

for i in frange(10,20,0.5):
  print(i)
````

​	range()迭代器函数只支持整数步长，这里实现的是一个支持小数步长的迭代器。

yield：表示运行到这里时，会暂时暂停，并且记录当前位置，再次运行到此处时，会继续往下执行。（如果这里不加yield关键字，会直接将循环执行至结束。这个自定义迭代器编写的关键就是yield关键字。yield关键字叫做生成器，也是迭代器的一种。方便你自己编写迭代器）

#### Lambda表达式

是一种函数简写的格式。

````python
def add(x,y):
  return x+y
````

简写为：

````python
lambda x,y:x+y
````

 （其实就是省去函数名和return关键字）

#### 闭包

概念：外部函数的变量被内部函数引用，这种写法就称为闭包。

````python
def fun():
	a = 1
  b = 2
  return a + b
-------------------------
#上面的代码用闭包改写
def sum(a):
  def add(b):
    return a + b
  return add
````

实现一个调用函数计数的功能

````python
def count(FIRST=0):
  cnt = [FIRST]
  def add_one():
    cnt[0] += 1
    return cnt[0]
  return add_one
````

​	实现闭包注意最后return的函数不能加括号，否则就变成返回一个函数。

使用闭包实现：a * x + b = y的例子

````python
def a_line(a,b):
  def arg_y(x):
    return a * x + b
  return arg_y
````

调用

````python
line1 = a_line(3,5)
print(line1(10))#这里省略的是如果ab不发生改变,那就只需要传入x的值,就可以求出y的值。如果ab发生变化,只需要重新调用a_line方法,就可以实现复用。
````

上边闭包的例子其实还能用lambda改写成更优雅的形式

````python
def a_line(a,b):
  return lambda x: a*x+b
````

#### 装饰器

就是增强方法，类似AOP

使用装饰器实现一个计算函数调用时间的方法

````python
def timer(func):
  def wrapper():
    start_time = time.time()
    func()
    stop_time = time.time()
    print("运行时间是 %s 秒" % (stop_time - start_time))
  return wrapper
  
````

调用

````python
@timer
def i_can_sleep():
  time.sleep(3)
````

以下例子展示了有参数的装饰器，和如何复用装饰器代码（再包裹一层函数）。

````python
def new_tips(argv):
    def tips(func):
        def nei(a, b):
            print('start %s %s' % (argv, func.__name__))
            func(a, b)
            print('stop')

        return nei
    return tips


@new_tips('add_module')
def add(a, b):
    print(a + b)


@new_tips('sub_module')
def sub(a, b):
    print(a - b)


print(add(4, 5))
print(sub(7, 3))
````

#### 上下文管理器

````python
fd = open('name.txt')
try:
    for line in fd:
        print (line)
finally:
    fd.close()
----------------------------#使用上下文管理器重写上面的方法

with open('name.txt') as f:
    for line in f:
        print(line)
````

#### 模块

import

#### AUTOPep8

| pycharm 安装PEP8 |                                                              |
| ---------------- | ------------------------------------------------------------ |
|                  | cmd窗口输入：pip install autopep8                            |
|                  | Tools→Extends Tools→点击加号                                 |
|                  |                                                              |
|                  | Name：Autopep8（可以随便取）                                 |
|                  | - Tools settings:                                            |
|                  | - Programs：`autopep8` （前提是你已经安装了哦）              |
|                  | - Parameters:`--in-place --aggressive --aggressive $FilePath$` |
|                  | - Working directory:`$ProjectFileDir$`                       |
|                  | - 点击Output Filters→添加，在对话框中的：Regular expression to match output中输入：`$FILE_PATH$\:$LINE$\:$COLUMN$\:.*` |

#### 类

````python
class Player():    #定义一个类
    def __init__(self, name, hp, occu):
        self.__name = name  # 变量被称作属性
        self.hp = hp
        self.occu = occu
    def print_role(self):    #定义一个方法
        print('%s: %s %s' %(self.__name,self.hp,self.occu))

    def updateName(self,newname):
        self.name = newname
````

#### 继承

````python
class Monster():
    '定义怪物类'
    def __init__(self,hp=100):
        self.hp = hp
    def run(self):
        print('移动到某个位置')

    def whoami(self):
        print('我是怪物父类')
class Animals(Monster):
    '普通怪物'
    def __init__(self,hp=10):
        super().__init__(hp)


class Boss(Monster):
    'Boss类怪物'
    def __init__(self,hp=1000):
        super().__init__(hp)
    def whoami(self):
        print('我是怪物我怕谁')
````

实例化，并判断类的类型

````python
a1 = Monster(200)
print(a1.hp)
print(a1.run())
a2 = Animals(1)
print(a2.hp)
print(a2.run())

a3 = Boss(800)
a3.whoami()

print('a1的类型 %s' %type(a1))
print('a2的类型 %s' %type(a2))
print('a3的类型 %s' %type(a3))

print(isinstance(a2,Monster))
# user1 = Player('tom',100,'war')  #类的实例化
# user2 = Player('jerry',90,'master')
# user1.print_role()
# user2.print_role()
#
#
# user1.updateName('wilson')
# user1.print_role()
# user1.__name = ('aaa')
# user1.print_role()
````

with和类结合

````python
class Testwith(object):
    '''
    with 包含了 __enter__ 和 __exit__ 方法
    '''

    def __enter__(self):
        print('run now ')

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_tb is None:
            print('exit normal ')
        else:
            print('exit with exception')


with Testwith():
    print('test')
    raise NameError('Exception')
````

#### 多线程

````python
import  threading
from threading import  current_thread

class Mythread(threading.Thread):
    def run(self):
        print(current_thread().getName(),'start')
        print('run')
        print(current_thread().getName(),'stop')


t1 = Mythread()
t1.start()
t1.join()

print(current_thread().getName(),'end')
````

多线程和队列

````python
from threading import Thread,current_thread
import time
import random
from queue import Queue

queue = Queue(5)

class ProducerThread(Thread):
    def run(self):
        name = current_thread().getName()
        nums = range(100)
        global queue
        while True:
            num = random.choice(nums)
            queue.put(num)
            print('生产者 %s 生产了数据 %s' %(name, num))
            t = random.randint(1,3)
            time.sleep(t)
            print('生产者 %s 睡眠了 %s 秒' %(name, t))

class ConsumerTheard(Thread):
    def run(self):
        name = current_thread().getName()
        global queue
        while True:
            num = queue.get()
            queue.task_done()
            print('消费者 %s 消耗了数据 %s' %(name, num))
            t = random.randint(1,5)
            time.sleep(t)
            print('消费者 %s 睡眠了 %s 秒' % (name, t))


p1 = ProducerThread(name = 'p1')
p1.start()
p2 = ProducerThread(name = 'p2')
p2.start()
p3 = ProducerThread(name = 'p3')
p3.start()
c1 = ConsumerTheard(name = 'c1')
c1.start()
c2 = ConsumerTheard(name = 'c2')
c2.start()
````

