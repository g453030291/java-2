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

