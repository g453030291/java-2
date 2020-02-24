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

迭代器与生成器

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

Lambda表达式：

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

