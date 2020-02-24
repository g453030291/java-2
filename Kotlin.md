### 基本数据类型

1.Boolean

2.Number（Int、Short、Float、Long、Double）

3.Char

4.字符串

### 类

class<类名>{<成员>}

### 区间

````kotlin
val range:IntRange = 0..1024//表示闭区间[0,1024]

val range_exclusive:IntRange = 0 util 1024//表示开区间[0,1024)也就是[0,1023]
````

### 数组

为了不必要的装箱、拆箱。基本类型的数组都是定制的。

| Java     | Kotlin      |
| -------- | ----------- |
| int[]    | IntArray    |
| short[]  | ShortArray  |
| long[]   | LongArray   |
| float[]  | FloatArray  |
| double[] | DoubleArray |
| char[]   | CharArray   |

### 常量与变量

val、var

### 函数

没有返回值就返回Unit，相当于java的void

````kotlin
fun sayHi(name:String){println("Hi,$name")}
fun sayHi(name:String) = println("Hi,$name")
````

#### 匿名函数

```kotlin
val sayHi = fun(name:String) = println("Hi,$name")
```

### Lambda表达式

#### 写法

{[参数列表]->[函数体,最后一行是返回值]}

例子：

`val sum = {a:Int,b:Int -> a+b}`

#### 调用

用（）进行调用

等价于invoke()

例子：

````kotlin
val sum = {a:Int,b:Int -> a+b}
sum(2,3)//这两行是等价的
sum.invoke(2,3)//这两行是等价的
````

#### 标签

````kotlin
fun main(args:Array<String>){
  args.forEach ForEach@{
    if(it == "q") return@ForEach//表示退出当前循环,执行下边代码
  }
  println("The End")
}
````

### 类成员

#### 延迟加载类属性

var延迟初始化：lateinit关键字

`lateinit var c:String`

Val延迟初始化：lazy方法

````kotlin
class A{
  val e:X by lazy{
    println("延迟加载的属性")
    X()
  }
}
````

### 运算符

Kotlin中任意类可以重载父类的运算符，通过运算符对应的具名函数来定义。使用关键字operator。

中缀表达式：只有一个参数的话，可以使用infix修饰函数。来省去.和()来调用。

### 表达式

if

swich

when

### 循环语句

for循环：for(element in elements)(运行机制就是实现了Interator方法)

while循环和java完全一样

continue和break和java用法完全一样

多层循环嵌套的终止可以结合标签使用

````kotlin
Outter@for(...){
  Inner@while(i<0){if(...) break@Outter}
}
````

### 异常捕获

try...catch...finally

### 具名参数、变长参数、默认参数

具名参数：arg1=xxx

变长参数：vararg xxx

默认参数：

````kotlin
fun hello(double:Double = 3.0){
  println(double)
}
````

### 类、抽象类和接口

和java相同

接口体现能力，抽象类反应本质

接口决定你能做什么，抽象类决定你是什么

### 类及成员的可见性

public、private、protected与java相同。

Kotlin中有一个特殊的访问修饰符，internal（表示模块内可见）。在本module中相当于public，在其他module中视为private。

### object

其实是class一种特殊情形，单例使用object修饰即可。其他的与一个普通class没有区别。可以有自己的属性，方法，函数等。

这个关键字等价Java代码如下：（静态公有对象+私有构造）

```java
public class MusicPlayerJava{
  public static MusicPlayerJava INSTANCE = new MusicPlayerJava();
  private MusicPlayerJava(){}
}
```

object特点：（生成饿汉式的单例）

1.只有一个实例的类

2.不能自定义构造方法

3.可以实现接口、继承父类

4.本质上就是单例模式最基本的实现

### 伴生对象与静态成员

Java当中可以对一个类定义静态方法和静态变量。Kotlin中使用伴生对象来声明一个静态变量或者方法。

````kotlin
class Latitude private constructor(val:value:Double){
  companion object{
    @JvmStatic
    fun ofDouble(double:Double):Latitude{
      return Latitude(double)
    }
    
    fun ofLatitude(latitude:Latitude):Latitude{
      return Latitude(latitude.value)
    }
    @JvmField
    val TAG:String = "Latitude"
  }
}
````

companion object 就是类的伴生对象。如果不需要java访问本类的静态方法、变量。就可以不加@JvmStatic、@JvmField。

### 方法重载与默认参数

Kotlin中可以使用默认参数的方式，替代方法重载。

@JvmOverloads。在Java调用Kotlin有默认参数的方法时，加上此注解。Java会将此方法视为重载的方法。

### 扩展成员

为现有的类添加方法、属性

fun X.y():Z{...}

val X.m注意扩展属性不能初始化，类似接口属性

Java调用扩展成员类似调用静态方法

### 属性代理

定义方法：

````kotlin
val/var <property name>:<Type> by <expression>
````

代理者需要实现相应的getValue、setValue方法。

代理val需要实现getValue，代理var需要实现两个方法。

### dataClass

。。。。。。