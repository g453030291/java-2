#### Spirng简介

​	Spring将各种设计模式融合到框架中，开发人员只需要关注自己的逻辑，在不知不觉中就已经用到了最佳的设计模式，自己的代码也得以健壮。

![spring架构图](https://github.com/g453030291/java-2/blob/master/images/spring架构图.png)

#### Spring核心模块：

##### Core Container

核心容器由spring-core、spirng-beans、spring-context、spring-context-support、spring-expression（Spirng Expression Language）模块组成。

​	spring-core和spirng-beans模块提供了框架的基本部分，包括IOC和依赖注入功能。BeanFactory是一个复杂的工厂模式的实现。

​	spring-context模块建立在core和beans模块之上，它是一种提供类似于JNDI注册表的框架样式访问对象的手段。ApplicationContext接口是context模块的焦点。

​	spring-context-support提供支持将常见的第三方库集成到Spring应用上下文中。

​	spirng-expression模块提供了一个强大的表达式语言。该语言支持设置获取属性值、属性赋值、方法调用、访问数组等内容。以及通过springIOC容器中的名称检索对象。

##### Spring AOP

​	spring-aop模块提供了一个符合AOP面向切面编程的实现，可以定义方法拦截器和切入点来干净的解耦应该实现分离的功能代码。单独的spirng-aspects模块提供与AspectJ的集成。

##### **Data Access/Integration**

​	spring-jdbc提供了一个JDBC抽象层，消除了繁琐的JDBC编码。

​	spring-tx支持实现特殊接口的类以及所有POJO的编程和声明式事务管理。

​	spring-orm为流行的对象关系映射API提供集成层。

​	spring-oxm提供了一个支持对象/xml映射实现的抽象层。

​	spring-jms提供了与spring-messaging模块的集成。

##### WEB

​	spring-web提供基本的面向web的集成功能。例如多部分文件上传和使用servlet和面向web的应用程序上下文来初始化IOC容器。他还包括一个http客户端和web相关部分的spring远程支持。

​	spring-webmvc模块包含用于web应用程序的spring模型视图控制器（MVC）和REST WEB入股实现。spring的mvc框架提供了domain model（领域模型）代码和web表单之间的清晰分离，并且集成了spring framework所有其它功能。

##### Test

​	spring-test模块支持使用junit或TestNG对spring组件进行单元测试。它提供了SpringApplicationContexts的一致加载和这些上下文缓存。它还提供了mock objects（模拟对象），可以使用它来单独测试代码。

#### **Spring使用场景**

##### 典型的spring web应用程序

![spring-web应用程序](https://github.com/g453030291/java-2/blob/master/images/spring-web应用程序.png)

​	spring的声明式事务管理使web应用程序具有完全事务性。所有定制业务逻辑都可以通过简单的POJO实现，并由spring的ioc容器管理。spring的ORM支持集成了JPA和Hibernate。表单控制器将web层与领域模型无缝集成，从而无需使用ActionForms或其它将http参数转换为领域模式值的类（也就是这些参数无需单独创建JOPO来表达）。

##### spring中间层使用第三方web框架

![spring与第三方web框架集成](https://github.com/g453030291/java-2/blob/master/images/spring与第三方web框架集成.png)

​	spring不强迫你使用它里面的一切。可以使用struts或其它UI框架与spring集成。它允许你使用spring的事务功能。你只需要使用ApplicationContext连接你的业务逻辑，并使用WebApplicationContext来集成你的web层。

##### 远程使用场景

​	当你需要通过web服务访问现有代码时，可以使用spring的hessian-Rmi或HttpInvokerProxyFactoryBean类。

##### EJB - 包装现有POJO

​	Spring Framework 还为 Enterprise JavaBeans 提供了一个 访问和抽象层，使您可以重用现有 的 POJOs ，并将其封装在无状态会话 bean 中，以用于可能需要声明性安全性的扩展即有安全 故障的 Web 应用程序中。

#### IOC容器

​	IoC也称为依赖注入（DI），它是一个过程，对象通过构造函数参数，工厂方法构造或者返回时在对象实例上设置属性来定义他们的依赖关系，即它们一起合作的其它对象。

​	org.framework.beans和org.springframework.context包时spirng framework的IoC容器的基础。BeanFactory接口提供了一种能够管理任何类型对象的高级配置机制。ApplicationContext是BeanFactory的子接口。它增加了与Spring的AOP特性更容易集成到一起的实现（如WebApplicationContext用于web程序）。

​	总之，BeanFactory提供了配置框架和基本功能，ApplicationContext添加了更多的企业功能。ApplicationContext是BeanFactory的完整超集。

​	Spring中，构成程序的主干并且由Spring IoC容器管理的对象称为bean。bean是由IoC容器实例化，组装和以其它方式管理的对象。此外，bean只是应用程序中许多对象之一。bean及其之间的依赖关系被容器所使用的的配置元数据所反射。

##### 容器概述

​	接口org.springframework.context.ApplicationContext表示Spring IoC容器，并负责实例化配置和组合上述bean。配置元数据以xml，java注解，java代码表示。它允许你表达组成你的应用程序的对象之间相互的依赖。

​	ApplicationContext接口的几个实现是Spring提供的开箱即用的。在独立的应用程序中，通常创建一个ClassPathXMLApplicationContext或FileSystemXMLApplicationContext的实例。也可以通过少量的XML配置，指示容器使用Java注解或代码作为元数据。（如图所示）

##### 配置元数据

![spring-应用程序与元数据结合](https://github.com/g453030291/java-2/blob/master/images/spring-应用程序与元数据结合.png)

​	Java配置元数据有三种方式，1.XML。2.注解。3.Java代码

##### 实例化容器

​	实例化Spring IoC容器很简单，提供一个ApplicationContext构造函数的参数即可。（本地文件系统，Java ClassPath）

````java
ApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"server.cml","dao.xml"});
````

##### 使用容器

​	ApplicationContext是一个高级工厂的接口，能够维护不同bean及其依赖关系的注册表。使用方法T getBean(String name,Class requiredType)可以检索bean的实例。

##### Bean概述

一、Spring IoC容器管理多个beans，这些bean是使用你给容器配置的数据创建的。

容器内，这些bean被定义为BeanDefinition对象，它包含以下元数据：

​	1.包限定类名：通常是定义的bean的实际实现类。

​	2.Bean行为配置元素，他说明了bean在容器内的行为（范围，生命周期，回调等等）

​	3.引用其它bean所需要做的工作，也称为依赖性

​	4.在新创建的对象中设置其它配置设置，例如，管理连接池的bean中使用的连接数或者池大小的限制

二、除了包含如何创建特定的bean定义外，ApplicationContext还允许用户注册在容器外部创建现有对象。这是通过访问ApplicationContext的BeanFactory通过方法getBeanFactory()来实现，它返回BeanFactory实现DefaultListableBeanFactory。通过registerSingleton(..)和registerBeanDefinition(..)支持这种注册。

注：Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其它自检步骤期间正确的运行。虽然在某种程序上支持覆盖现有元数据和现有单例实例，但是在运行时对新bean的注册未被官方支持，并且可能导致并发异常或bean容器中的不一致状态。

##### Bean命名

​	id唯一，name为别名。约定命名bean时使用Java约定作为实例字段名称。就是bean名称以小写字母开头，后边使用驼峰式。命名Bean一致使得配置更容易阅读和理解，如果使用Spirng AOP，它按照名称相关的一组bean应用。

##### 实例化bean

​	bean定义本质上是创建一个或多个对象的方法。并使用由该bean定义封装的配置元数据来创建实际对象。

​	如果基于XML的配置元数据，则要在bean元素的class属性中实例化对象的类型。这个class属性，在内部是一个BeanDefinition实例的Class属性。通常是强制的。使用class属性有两种方法，1.通常，容器本身通过反射调用构造函数直接创建bean，等同于new运算符的java代码。2.要指定包含将被调用来创建对象的static工厂方法的实际类，类中包含静态方法。从static工厂方法返回的可以是完全相同的类或另一个类。

​	内部类通过`com.example.Foo$Bar`调用。

##### 通过构造函数实例化

​	当你通过构造函数方法创建一个bean时，所有正常的类都可以被Sprin使用并兼容。Spring IoC容器可以管理你想要管理的任何类，它不限于管理真正的JavaBean。

##### 使用静态工厂方法实例化

​	当定义一个使用静态工厂方法创建的bean时，除了需要指定class属性外，还需要通过factory-method属性来指定创建bean实例的工厂方法。Spring讲调用该方法。（创建对象的方法必须是static的）

````xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
````

````java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}
    public static ClientService createInstance() {
        return clientService;
} }
````

##### 使用实例工厂（instance factory）方法实例化

​	使用实例工厂方法的实例化从容器调用现有的bean非静态方法创建新bean。要使用此机制，将class属性留空，并在factory-bean中，指定当前容器中包含调用的实例方法的bean的名称。使用factory-method属性设置工厂方法本身的名称。

````xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator"> <!-- inject any dependencies required by this locator bean -->
</bean>
<!-- the bean to be created via the factory bean -->
<bean id="clientService" factory-bean="serviceLocator" factory-method="createClientServiceInstance"/>
````

````java
public class DefaultServiceLocator {
    private static ClientService clientService = new ClientServiceImpl();
    private DefaultServiceLocator() {}
    public ClientService createClientServiceInstance() {
        return clientService;
} }
````

注：一个工厂类也 可以有多个工厂方法

##### 依赖

​	依赖注入是一个过程，对象通过构造函数，工厂方法的参数构建实例后设置的属性来定义他们的依赖关系。容器在创建bean时注入这些依赖。这一过程是相反的，因此命名为Inversion of Control（控制反转）。主要有两个形式：1.基于构造函数的依赖注入。2.基于Setter的依赖注入。



