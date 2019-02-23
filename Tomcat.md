### Tomcat目录结构

#### conf目录

`catalina.policy`：Tomcat安全策略文件，控制JVM相关权限，具体可参考`java.security.Permission`

`catalina.properties`：Tomcat Catalina行为控制配置文件，比如Common CLassLoader

`logging.properties`：Tomcat日志配置文件，JDK Logging

`server.xml`：Tomcat Server配置文件

	`GlobalNamingResources`：全局JNDI资源

`context.xml`：全局Context配置文件

`tomcat-user.xml`：Tomcat角色配置文件，=（Realm文件实现方式）

`web.xml`：Servlet标砖的web.xml部署文件，Tomcat默认实现部分配置入内：

	`org.apache.catalina.servlets.DefaultServlet`
	
	`org.apache.jasper.servlet.JspServlet`

#### lib目录

Tomcat存放公用类库

	`ecj-*.jar`：Eclipse java编译器
	
	`jasper.jar`：JSP编译器

#### logs目录

`localhost.${date}.log`	：当Tomcat应用起不来的时候，多看该文件，比如，类冲突等

	`NoClassDefFoundError`
	
	`ClassNotFoundException`

#### webapps目录

简化web应用部署的方式

#### 部署web应用

方法一：放置在webapps目录

直接拖进去

方法二：修改`conf/server.xml`

添加Context元素：

```xml
<Context docBase="${webAppAbslolutePath}" path="/" reloadable="true" />
<Context docBase="${webAppAbslolutePath}" path="/tomcat" reloadable="true" />
```

`Container`

`Context`

该方式不支持动态部署，建议考虑在生产环境使用。

方法三：独立context xml配置文件

首先注意`conf\Catalina\localhost`

独立context XML 配置文件路径：`${TOMCAT_HOME}/conf/Catalina/localhost`+`${ContextPath}`.xml

注意：该方式可以实现热部署，因此建议开发环境使用

#### I/O连接器

实现类：`org.apache.catalina.connector.Connector`

### 问题

1.独立context XML配置文件时，设置path属性是无效的

2.根独立context XML配置文件路径`${TOMCAT_HOME}/conf/${Engine.name}/${HOST.name}/ROOT.xml`

3.实现热部署。调整`<context>`元素中的属性`reloadable="true"`

4.tomcat使用的线程池是自行实现的java的excutor类，实现了一套标准的线程池类，底层还是调用jdk底层的线程池类中的executor方法。

5.tomcat6之前，使用的是tomcat自己实现的一套生产者，消费者模式。

6.tomcat8之前，默认使用的是阻塞（bloking）的实现方式。在tomcat8以后，使用的非阻塞方式（non-bloking）。但是其实，在读取http body的时候，两者都是阻塞的。只有在http请求的非核心阶段（如读取http headers ，进行ssl握手）才是非阻塞的。这样也就解释了为什么在java中使用Thread local是没有线程安全问题的。

### Tomcat性能优化

#### 减配优化

场景一：假设当前REST应用（微服务）

分析：它不需要静态资源，Tomcat容器静态和动态

* 静态处理：`DefaultServlet`

- 优化方案：通过移除`conf/web.xml`中`org.apache.catalina.servlets.DefaultServlet`
- 动态：应用`Servlet`、`JspServlet`
- 优化方案：通过移除`conf/web.xml`中JspServlet（`org.apache.jasper.servlet.JspServlet`）
- 移除welcome-file-list

````xml
<welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
````

- 如果程序是REST JDON Content-Type 或者 MIME Type：application/json

- 移除Session设置（对于微服务/REST应用，不需要Session，因为不需要状态）

  Session通过jsessionId进行用户跟踪，HTTP无状态，需要一个ID与当前用户会话联系。Spring Session HttpSession jsessionId 作为Redis的key，实现多个机器登陆，用户会话不丢失。

  存储方法：Cokkie、URL重写、SSL

- 移除Valve（位于server.xml）

  `Valve`类似于`filter`

  移除`AccessLogValve`，可以通过Nginx的Access Log 替代，`Valve`实现都需要消耗Java应用的计算时间。

> DispatcherServlet：Spring Web MVC 应用程序的 Servlet
>
> JspServlet：负责编译并且执行jsp页面
>
> DefaultServlet：Tomcat负责处理静态资源的Servlet

 场景二：需要JSP的情况

分析：`JspServlet`无法去掉，了解`JspServlet`处理原理

> Servlet周期：
>
> - 实例化：Servlet 和 Filter实现类必须包含默认构造器。反射的方式进行实例化。
> - 初始化：Servlet 容器 调用 Servlet 或 Filter init() 方法
> - 销毁：Servlet 容器关闭时，Servlet 或者 Filter destory() 方法被调用

	Servlet 或者 FIlter在一个容器中，是一般情况在一个Web App 中是一个单例，不排除应用定义多个。

JspServlet相关的优化`ServletConfig`参数（web.xml）：

- 需要编译

  - compiler

  - modification TestInterval

- 不需要编译

  - development设置false

`development`=false时，那么，这些JSP要如何编译。优化方法：

Ant Task 执行 JSP 编译

Maven 插件：org.codehaus.mojo:jspc-maven-plugin

总结：

	`conf/web.cml`作为Servlet 用用默认的web.xml，实际上，应用程序存在两份web.xml，其中包括应用的web.xml，最终将两者合并。

	JspServlet 如果 development 参数为 true，它会自动检查文件是否修改，如果修改重新翻译，再编译（加载和执行）。言外之意，development=true的时候，可能会导致内存溢出。卸载Class 不及时所导致 Perm 区域不够。

#### 问题

如何卸载一个class？

	ParentClassLoader-> 1.class  2.class  3.class 

	ChildClassLoader-> 4.class  5.class

	ChildClassLoader load 1 - 5 .class

	1.class 需要卸载，需要将ParentClassLoader设置为null，当ClassLoader被GC后，1-3 class全部会被卸载。

	1.class 它是文件，文件被JVM加载，二进制->Verify->解析

#### 配置优化

关闭自动重载

```xml
<Context docBase="../...././/.././" reloadable="false"></Context>
```

修改连接线程池数量（server.xml）

```xml
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
<Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

通过Tomcat源码可知：`<Executor>`标签中实际上使用的是 tomcat 声明的一个接口，`org.apache.catalina.Executor`。扩展自J.U.C标准接口`java.util.concurrent.Executor`。tomcat中的实现类为`org.apache.catalina.core.StandardThreadExecutor`

总结：Tomcat IO 连接器使用的线程池实际上是标准的 Java 线程池的扩展，最大线程数量和最小线程数量实际上分别是MaximumPoolSize 和 CorePoolSize。

#### 通过JMX

评估一些参考：

1.正确率（Jmeter的测试数据）

2.Load（CPU->JVM GC）

3.TPS/QPS（越大越好）

4.CPU密集型（加密解密、算法）

5.I/O密集型，网络、文件读写等



问题：

到底设置多少线程数才是最优？

首先，评估整体的请求量，假设100W QPS，有机器数量10台，每台支撑1W QPS。

第二，进行压力测试，需要一些测试样本，JMeter来实现，假设一次请求需要response time 10ms，1秒就可以同时完成10x10=100个请求。10000 / 100 = 100 线程。

确保，Load 不能太高。需要减少Full GC，GC 取决于 JVM堆的大小。假设执行一次操作需要5MB内存，一共就需要50GB的内存。但是一般机器不会有这么大内存。一般机器，加入有20GB内存，必然要执行GC。要不就去调优程序，最好对象存储外化，比如Redis，同时又需要评估Redis网络开销。又要评估网卡的接受能力。

第三，常规性压测，由于业务变更，会导致底层性能变化。

#### JVM优化

调整GC算法

如果Java版本小于9，默认`PS MarkSweep`，可选设置CMS、G1。

如果Java 9的话，默认`G1`.

