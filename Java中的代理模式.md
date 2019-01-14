#### 一、什么是Java中的代理模式（静态代理）？

所谓静态代理，就是代理类由程序员自己编写的，在编译期就确定好了的。

```java
public interface HelloService{
    public void say();
}

public class HelloServiceImpl implements HelloService{
    @Override
    public void say(){
        System.out.println("hello world");
    }
}
```

上面的代码比较简单，定义了一个接口和其实现类。这就是代理模式中的目标对象和目标对象的接口，结下来定义代理对象。

```java
public class HelloServiceProxy implements HelloService{
    private HelloService target;
    public HelloServiceProxy(HelloService target){
        this.target = target;
    }
    
    @Override
    public void say(){
        System.out.println("记录日志");
        target.say();
        System.out.println("清理数据");
    }
}
```

上面就是一个代理类，它也实现了目标对象的接口，并且扩展了say方法。下面是一个测试类。

```java
public class Main{
    @Test
    public void testProxy(){
        //目标对象
        HelloService target = new HelloServiceImpl();
        //代理对象
        HelloServiceProxy proxy = new HelloServiceProxy(target);
        proxy.say();
    }
}
```

//记录日志

//hello world

//清理数据

这就是一个简单的静态代理模式的实现。代理模式中的所有角色（代理对象、目标对象、目标对象的接口）等都是在编译期就确定好的。

静态代理的用途

控制真实对象的访问权限，通过代理对象控制对真实对象的使用权限。

避免创建大对象， 通过使用一个代理小对象来代表一个真实的大对象，可以减少系统资源的消耗，对系统进行优化并提高运行速度。

增强真实对象的功能，这个比较简单，通过代理可以在调用真实对象的方法前后增加额外功能。

#### 二、什么是动态代理，和反射有什么关系

前面介绍了静态代理，虽然静态代理模式很好用，但是静态代理还是存在一些局限性的，比如使用静态代理模式需要程序员手写很多代码，这个过程是比较浪费时间和精力的。一旦需要代理的类中方法比较多，或者需要同时代理多个对象的时候，这无疑会增加很大的复杂度。

有没有一种方法，可以不需要程序员自己手写代理类呢。这就是动态代理。

动态代理中的代理类并不要求在编译期就确定，而是可以在运行期动态生成，从而实现对目标对象的代理功能。

反射是动态代理的一种实现方式。 

#### 三、Java中的动态代理有几种实现方式，各有什么优缺点？

Java中，实现动态代理有两种方式：

1.JDK动态代理：java.lang.reflect包中的Proxy类和InvocationHandler接口提供了生成动态代理类的能力。

2.Cglib动态代理：Cglib(Code Generation Library)是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。

关于这两种动态代理的区别和用途：

区别：JDK的动态代理有一个限制，就是使用动态代理的对象必须实现一个或多个接口。如果想代理没有实现接口的类，就可以使用Cglib实现。

Cglib是一个强大的高性能代码生成包，它可以在运行期扩展Java类与实现Java接口。它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。

Cglib包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。不鼓励直接使用ASM，因为它需要你对JVM内部结构包括class文件的格式和指令集都很熟悉。

Cglib与动态代理最大的区别就是：

使用动态代理对象必须实现一个或多个接口

使用Cglib代理的对象则无需实现接口，达到代理类无侵入。

#### 四、Java中动态代理有哪些用途？

Java的动态代理，在日常开发中可能并不经常使用，但是并不代表它不重要。Java的动态代理最主要的用途就是应用在各种框架中。因为使用动态代理可以很方便的运行期生成代理类，通过代理类可以做很多事情，比如AOP，比如过滤器，拦截器等。

在我们平时使用的框架中，像servlet的filter、包括spring提供的aop以及struts2的拦截器都使用了动态代理功能。我们日常看到的mybatis分页插件，以及日志拦截、事物拦截、权限拦截这些几乎全部有动态代理的身影。

#### 五、Java实现动态代理的大致步骤是怎样的？

