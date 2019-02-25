#### 一、Java中如何对字符串进行编码和解码（使用不同编码方式）？

把字符串转成对应编码下的字节数组：`byte[] getBytest(String charsetName)`

把字节数组按照编码方式进行编码：`String(byte[] bytes,String charsetName)`

#### 二、什么是JIT编译器？

	JIT编译器，即Just-In-Time编译，译为实时编译器.使用即时编译器技术，能够加速 Java 程序的执行速度。
	通常通过 javac 将程序源代码编译，转换成 java 字节码，JVM 通过解释字节码将其翻译成对应的机器指令，逐条读入，逐条解释翻译。很显然，经过解释执行，其执行速度必然会比可执行的二进制字节码程序慢很多。为了提高执行速度，引入了 JIT 技术。
	在运行时 JIT 会把翻译过的机器码保存起来，以备下次使用，因此从理论上来说，采用该 JIT 技术可以接近以前纯编译技术。
	JAVA程序还是通过解释器进行解释执行，当JVM发现某个方法或代码块运行特别频繁的时候，就会认为这是“热点代码”（Hot Spot Code)。然后JIT会把部分“热点代码”翻译成本地机器相关的机器码，并进行优化，然后再把翻译后的机器码缓存起来，以备下次使用。
	HotSpot虚拟机中内置了两个JIT编译器：Client Complier和Server Complier，分别用在客户端和服务端，目前主流的HotSpot虚拟机中默认是采用解释器与其中一个编译器直接配合的方式工作。
当 JVM 执行代码时，它并不立即开始编译代码。首先，如果这段代码本身在将来只会被执行一次，那么从本质上看，编译就是在浪费精力。因为将代码翻译成 java 字节码相对于编译这段代码并执行代码来说，要快很多。第二个原因是最优化，当 JVM 执行某一方法或遍历循环的次数越多，就会更加了解代码结构，那么 JVM 在编译代码的时候就做出相应的优化。

#### 三、什么是JIT编译器的热点检测，作用是什么？

	热点检测就是找到特别频繁运行的代码,编译器以整个方法作为编译对象,因此如果编译器发现一个方法或循环体被频繁调用,就会出发即时编译.
如何发现:
	基于采样的热点预测:虚拟机周期性检查各个线程的栈顶,如果发现某些方法经常出现在 栈顶,那这段代码就会被认定为热点代码
	好处:实现简单高效,可以很容易的获取方法间的调用关系
	缺点:难以精确确认一个方法的热度,容易收到线程阻塞或别的外界因素的影响
	基于计数器的热点检测:为每个方法甚至是代码块建立计数器,统计执行次数,超过设定阀值就会被认定为热点代码
	好处:计算热点精确严谨
	缺点:实现复杂,需要为每个方法建立并维护计数器,而且不能直接获取方法调用关系
HotSpot:
	HotSpot 中采用的是第二种,其为每个方法准备了两个计数器:方法调用计数器和回边计数器
方法调用计数器:默认设置下,统计一段时间内方法被调用的次数
	回边计数器:统计一个方法中循环体执行的次数
作用:
	将热点代码编译成目标机器的机器码文件,提高执行效率.

#### 四、什么是逃逸分析，有什么用？

https://www.hollischuang.com/archives/2583

#### 五、Java中如何使用正则表达式进行匹配？

java.util.regex 包主要包括以下三个类：
Pattern 类：
	pattern 对象是一个正则表达式的编译表示。Pattern 类没有公共构造方法。要创建一个 Pattern 对象，你必须首先调用其公共静态编译方法，它返回一个 Pattern 对象。该方法接受一个正则表达式作为它的第一个参数。
Matcher 类：
	Matcher 对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法。你需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象。
PatternSyntaxException：
	PatternSyntaxException 是一个非强制异常类，它表示一个正则表达式模式中的语法错误。
以下实例中使用了正则表达式 .*runoob.* 用于查找字符串中是否包了 runoob 子串：

```java
import java.util.regex.*;
class RegexExample1{
    public static void main(String args[]){
        String content = "I am noob " +
        "from runoob.com.";
        String pattern = ".*runoob.*";
        boolean isMatch = Pattern.matches(pattern, content);
        System.out.println("字符串中是否包含了 'runoob' 子字符串? " + isMatch);
    }
}实例输出结果为：
字符串中是否包含了 'runoob' 子字符串? true
```

#### 六、什么是函数式编程？

	函数式编程是种编程方式，它将电脑运算视为函数的计算。函数编程语言最重要的基础是λ演算（lambda calculus），而且λ演算的函数可以接受函数当作输入（参数）和输出（返回值）。

	比起指令式编程，函数式编程更加强调程序执行的结果而非执行的过程，倡导利用若干简单的执行单元让计算结果不断渐进，逐层推导复杂的运算，而不是设计一个复杂的执行过程。

	它属于"结构化编程"的一种，主要思想是把运算过程尽量写成一系列嵌套的函数调用。

	举例来说，现在有这样一个数学表达式： (1 + 2) * 3 - 4 

	传统的过程式编程，可能这样写： 

	var a = 1 + 2; 

	var b = a * 3; 

	var c = b - 4; 

	函数式编程要求使用函数，我们可以把运算过程定义为不同的函数，然后写成下面这样： 

	var result = subtract(multiply(add(1,2), 3), 4); 这就是函数式编程。 

#### 七、什么是Java8 中的lambda表达式？

Lambda 表达式是推动 Java 8 发布的最重要新特性。
Lambda 允许把函数作为一个方法的参数传递进方法中，使用 Lambda 表达式可以使代码变的更加简洁紧凑。
lambda 表达式的语法格式如下：
(parameters) -> expression
或
(parameters) ->{ statements; }
以下是lambda表达式的重要特征:
	可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
	可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
	可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
	可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。
Lambda 表达式的简单例子:
// 1. 不需要参数,返回值为 5 
() -> 5 
// 2. 接收一个参数(数字类型),返回其2倍的值 
x -> 2 * x 
// 3. 接受2个参数(数字),并返回他们的差值 
(x, y) -> x – y 
// 4. 接收2个int型整数,返回他们的和 
(int x, int y) -> x + y 
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void) 
(String s) -> System.out.print(s)

#### 八、什么是Java8中的Stream？

	Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念。它也不同于 StAX 对 XML 解析的 Stream，也不是 Amazon Kinesis 对大数据实时处理的 Stream。
	Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。
	Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。
	通常编写并行代码很难而且容易出错, 但使用 Stream API 无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。所以说，Java 8 中首次出现的 java.util.stream 是一个函数式语言+多核时代综合影响的产物。

详细使用方案参考：

[Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)