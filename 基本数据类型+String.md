#### 一、什么是Static Nested Class、什么是Inner Class、二者有什么不同？

Inner Class：内部类；

不可以定义静态成员变量、可以访问外部类的成员变量

Static Nested Class：静态内部类；

不能访问非静态的外部类成员变量

#### 二、整型的几种中，各个类型的取值范围是多少，如何计算的？超出范围会发生什么？

	Java中的整型主要包括byte、short、int、long这四种。数字范围也是从小到大。主要和他们存储数据时占的字节数有关。

	1字节=8位（bit）；表示的范围是（-2^7～2^7-1）；

byte：用1个字节存储，范围-128（-2^7）～127（2^7-1）。初始化时默认值为0。

short：用2个字节存储，范围-32768（-2^15）～32767（2^15-1）。初始化时默认值为0。

int：用4个字节存储，范围-2147483648（-2^31）～2147483647（2^31-1）。初始化时默认值为0。

long：用8个字节存储，范围-9223372036854775808（-2^63）～9223372036854775807（2^63-1）。初始化时默认值为0l。

如果使用两个Integer.MAX_VALUE相加，使用一个int接受，就会发生溢出，溢出不会抛异常，也没有任何提示。所以在程序运算时，一定要注意数据溢出的问题。

#### 三、Java中的char是否可以存储中文字符？

Java中的char存储占2个字节，中文使用unicode编码，同样占2个字节。只要在unicode编码中有的中文，即可以使用char存储。生僻字和特殊字除外。

#### 四、int和Integer，boolean和Boolean等，之间有什么区别？

1.默认值不同，基本类型默认值为0、false、\u0000，包装类默认null

2.初始化不同，一个需要new，一个不需要

3.存储方式不同

4.int.class是原始类型，Integer.class是对象类型，所以一个有成员变量和方法，一个没有。

包装类就是把基本类型，包装在一个类里，提供一些常用的操作。

#### 五、什么是自动拆箱、自动装箱？

	自动装箱就是Java自动将原始类型值转换为对应的对象。比如将int值转为Integer这个对象。整个过程叫装箱，反之叫拆箱。

	反编译后得知，自动装箱是调用了Integer的valueOf(int)方法，拆箱是调用了Integer的intValue方法。

#### 六、接口定义时，表示一个字段是否成功，最好使用哪种方式？

boolean success

原因：

1.不建议使用包装类Boolean，因为包装类默认值为null。但是boolean应该为确定的true或false。

2.POJO类型属姓名通常不要使用isXXX。会导致某些框架解析错误。

#### 七、String a = new String("b");定义了几个对象？

	字符串的分配，和其他对象分配一样，耗费高昂的时间、空间代价。JVM为了提高性能和减少内存开销，在实例字符串常量的时候进行了一些优化。为了减少在JVM中创建字符串的数量，字符串为维护了一个字符串常量池，每当代码创建字符串常量时，JVM会首先检查字符串常量池。如果字符串已经存在池中，就返回池中实例引用。如果字符串不在池中，就会实例化一个字符串并放到池中。Java能够进行这样的优化，是因为字符串是不可变的，可以不用担心数据冲突进行共享。

String a = new String("b");创建了几个对象？

1.在常量池中检查是否有“b”这个对象

2.有则返回对应引用的实例

3.没有则创建对应的实例对象

4.在堆中new一个string("b")对象

5.将对象地址赋值给a，创建一个引用

	所以，在常量池中没有“b”这个字面量，则创建两个对象，否则，创建一个对象，以及创建一个引用。

#### 八、如何比较两个字符串？

使用equels()是比较内容是否相同；

‘==’比较的是地址是否相同

#### 九、String有没有长度限制，为什么？如果有，超过限制会发生什么？

编译期：

	最多可以存储65534个字符。因为String会在常量池中存储一份，这个限制其实是常量池的限制。所有在常量池中保存的数据，长度最大不能超过65535。

运行期：

	最大可以存储Integer.MAX_VALUE个字符。约等于4G。

#### 十、String的“+”是如何实现的？

1.String s = “a“ + ”b“；编译器会进行常量折叠（因为两个都是编译期常量，编译期可知），即变成String s = ”ab“；

2.对于能够优化的String s = ”a“ + 变量；使用StringBuilder的append()方法替代，最后调用toString()方法。（底层就是new String()）。

#### 十一、String、StringBuilder、StringBuffer之间的区别与联系？

String：String是不可变对象，每次对String进行操作，等同于生成了一个新的String对象，操作完成后将指针指向新的String对象。所以经常改变内容的，推荐使用StringBuffer。他每次的操作都是对对象本身进行。

StringBuffer：线程安全的可变字符串序列。一个类似于String的字符串缓冲区，可将字符串缓冲区安全的用于多个线程。StringBuffer上主要操作是append和insert方法，可重载这些方法以接受任意类型数据。

StringBuilder：提供一个与StringBuffer兼容的API，但不保证同步，该类被设计用于StringBuffer的一个简易替换，用于字符串缓冲区被单个线程使用时。（大多数实现中，它比StringBuffer要快）

#### 十二、SubString方法到底做了什么？不同版本JDK有什么区别？

subString(int beginIndex, int endIndex);方法截取字符串并返回其[beginIndex,endIndex-1]范围的内容。

JDK1.6：

String是通过字符数组实现的。在jdk1.6中，String包含三个成员变量：char value[],int offset, int count。他们分别用来存储真正的字符数组，数组第一个位置索引以及字符串中包含的字符个数。

当调用substring方法时，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset的值不同。

问题：

如果你有一个很长很长的字符串，当你使用substring方法时，这可能导致性能问题。如果你只需要很短一个字符，但底层substring却引用了整个字符串（因为这个长的字符数组一直被引用，所以无法被回收，可能会导致内存泄漏）。在jdk1.6中，使用x = x.substring (x,y) + "";来解决该问题。其原理就是生成一个新的字符串并引用他。

JDK1.7:

上面提到的问题，在jdk1.7中得到解决。jdk1.7的substring方法会在堆中创建一个新的数组（对字符串截取后会return new String()）。避免对老字符串的引用。来解决内存泄漏的问题。

#### 十三、如何理解String的intern方法，不同版本JDK有何不同？为什么？

Intern();是String的public方法，用法是：

```java
String s1 = new String("aa");
String s2 = s2.intern();
System.out.pringln(s1==s2);
----->true;
```

当一个String实例调用intern()方法时，Java查找常量池中是否有相同的Unicode的字符串常量，如果有，返回其引用，如果没有，则在常量池中增加一个Unicode等于str的字符串并返回其引用。

不同版本的JDK有何不同？

```java
String str1 = new StringBuilder("aa").append("bb").toString();
System.out.pringtl(str1.intern()==str1);
JDK1.6---->false;
JDK1.7---->true;
```

1.JDK1.6和JDK1.7的intern()表现不同，原因是Java7常量池的位置从PermGen区改到了Java堆中。

2.JDK1.6中intern方法会把首次遇到的字符串实例复制到永久带（常量池）中，并返回此引用。但是在JDK1.7中，只会把首次遇到的字符串实例添加到常量池中（没有复制），并返回其引用。

3.对意义上代码的str1.intern(),在JDK1.6中，会把aabb这个字符串复制到常量池中，并返回他的引用。所以str1.intern()的值和str1的值，即两个对象的地址是不一样的。

4.对于以上代码中的str1.intern()，在JDK1.7中，会把str2的引用保存到常量池中，并把这个引用返回。所有str1.intern()的值和str1的值相等。

另：

```java
String str2 = new StringBuilder("ja").append("va").toString();
System.out.println(str2.intern==str2);
```

以上代码,在JDK1.6和JDK1.7中都返回false。因为"java"这个常量在常量池中已经有了。这个字符串在Java中随处可见，肯定被初始化过。在JDK1.7中str2.intern()返回的内容是很久之前初始化时候的引用，自然和刚刚创建的字符串的引用不相等。

#### 十四、Java中整型的缓存机制

```java
public static void main(String... args){
    Integer integer1 = 3;
    Integer integer2 = 3;
    
    if(integer1 == integer2){
        System.out.println("integer1 == integer2");
    }else{
        System.out.println("integer1 != integer2");
    }
    
    Integer integer3 = 300;
    Integer integer4 = 300;
    
    if(integer3 == integer4){
        System.out.println("integer3 == integer4");
    }else{
        System.out.println("integer3 != integer4");
    }
}
```

上面输出的结果是：

Integer1 == integer2

Integer3 != integer4

Java中Integer的缓存实现：

在Java1.5中，在Integer操作上引入了一个新功能来节省内存提高性能。整型对象通过使用相同的对象引用实现了缓存和重用。（用于与整数值区间-128至+127。只适用于自动装箱。使用构造函数创建对象不适用。）

Java的编译器把基本数据类型自动转换成封装对象的过程，叫自动装箱。相当于使用valueOf方法：

```java
Integer a = 10;//autoboxing

Integer b = Integer.valueOf(10);//under the hood
```

##### 下面是valueOf的源码实现(JDK1.8)

```java
/**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

在创建对象前先从IntegerCache.cache中寻找。如果没找到才使用new新建对象。

##### 下面是IntegerCache的源码实现：

```java
/**
     * Cache to support the object identity semantics of autoboxing for values between
     * -128 and 127 (inclusive) as required by JLS.
     *
     * The cache is initialized on first usage.  The size of the cache
     * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
     * During VM initialization, java.lang.Integer.IntegerCache.high property
     * may be set and saved in the private system properties in the
     * sun.misc.VM class.
     */

    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

	javadoc详细说明了缓存支持-128到+127之间的自动装箱过程。最大值127可以通过-XX:AutoBoxCacheMax=size修改。缓存通过一个for循环实现，从低到高创建尽可能多的整数并存储在一个整数数组中。这个缓存会在Integer类第一次被使用的时候初始化出来。以后就可以使用混存中包含的实例对象。

	实际上这个功能在JDK1.5中第一次引入时，范围是固定的-128到+127。后来在JDK1.6中，可以通过java.lang.Integer.IntegerCache.high设置最大值。这样我们可以根据需求灵活配置缓存来提高性能。

##### Java语言规范中的缓存行为：

在Boxing Conversion部分的Java语言规范规定如下：

如果一个变量p的值是：

1.-128到+127之间的整数

2.true和false的布尔值

3.'\u000'至'\u007f'之间的字符

中时，将p包装成a和b两个对象时，可以直接使用a==b来判断a和b的值是否相等。

##### 其他缓存的对象：

这种缓存行为不仅适用于Integer，所有整数的类型都有类似的缓存机制。

ByteCache用于缓存Byte对象。

ShortCache用于缓存Short对象。

LongCache用于缓存Long对象。

CharacterCache用于缓存Character对象。

Byte,Short,Long有固定范围：-128到+127。对于Character范围是0到127。除了Integer以外，这个范围都不能改变。

#### 十五、Java代码及数据库存储中，如何对金额进行表示和计算？

1.以元为单位。Java中存储类型为BigDecimal。数据库中存储类型为number(10,2)。计算过程保留两位小数，四舍五入、向上取整、向下取整根据业务实际情况而定。

2.以分为单位。Java中存储类型为Long，数据库中存储类型为big int。计算过程中保留整数，考虑四舍五入或者取整。