#### 查看JVM运行时状态：

参数：

-XX:+PrintFlagsInitial：JVM初始化时的参数（例：`java -XX:+PrintFlagsInitial -version`）

-XX:+PrintFlagsFinal：JVM最终的运行参数

-XX:+UnlockExperimentaIVMOptions：解锁试验参数

-XX:+UnlockDiagnosticVMOptions：解锁诊断参数

-XX:+PrintCommandLineFlags：打印命令行参数

命令：

`jps -l`：查看所有运行的Java程序pid

`jinfo -flags <pid>`：查看某个Java进程，经过设置的JVM参数

#### jstat查看JVM统计信息：

类装载：

`jstat -class <pid> 1000 10`：查看某pid进程，每一秒打印一次，打印10次

Loaded：被装置的类的个数

Bytes：所有被装载类的总大小

Unloaded：被卸载的类个数

Bytes：被卸载的类大小

Time：装载和卸载的总花费时间

垃圾收集：

`jstat -gc <pid> 1000 10`：：查看某pid进程的gc情况，每一秒打印一次，打印10次

S0C、S1C、S0U、S1U：S0和S1的总量与使用量

EC、EU：Eden区总量与使用量

OC、OU：Old区总量与使用量

MC、MU：Metaspace区总量与使用量

CCSC、CCSU：压缩类空间总量与使用量

YGC、YGCT：YoungGC的次数与时间

FGC、FGCT：FullGC的次数与时间

GCT：总的GC时间

JIT编译：

`jstat -compiler <pid>`：查看某pid进程的JIT编译、运行情况

Compiled：完成了多少编译任务

Failed：失败了多少任务

Invalid：

Time：共花费时间

FailedType：

FailedMethod：

#### jmap分析内存溢出：

堆内存溢出：

设置堆内存为32M（`-Xmx32M -Xms32M`）

非堆内存溢出：

设置元空间为32M（`-XX:MetaspaceSize=32M -XX:MaxMetaspaceSize=32M`）

设置内存溢出自动导出：

`-XX:+HeapDumpOnOutOfMemoryError`

`-XX:HeapDumpPath=./`

使用jmap命令手动导出：

`jmap -dump.format=b,file=heap.hprof <pid> `

`jmap -heap <pid>`：查看堆内存使用情况

使用MAT工具打开到处的hprof文件，查看内存分析

#### 如何利用jstack解决CPU负载过高的问题？

1.`jps`查看运行进程，得到运行异常Java进程的pid

2.使用上面的到的进程pid，以线程模式使用top命令，得到运行异常的线程pid：`top -p <pid> -H`

3.dump出JVM线程运行文件：`jstack <pid> > filename.txt`

4.将得到的线程pid转为16进制：`printf "%x" <pid>`

5.到filename.txt中查找得到的16进制pid线程，查看线程在做什么操作

#### 如何利用jstack查找死锁？

1.dump出jstack文件：`jstack <pid> > filename.txt`

2.文件最后一行显示，Found 1 deadlock

#### 如何使用Java VisualVM监控远程Tomcat？

修改tomcat/bin/Catalina.sh

````shell
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun,management.jmxremote.port=9004 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.net.preferIPv4Stack=true -Djava.rmi.server.hostname=10.110.3.63"
````

#### 如何使用Java VisualVM监控远程Java进程？

````shell
nohup java -Dcom.sun.management.jmxremote -Dcom.sun,management.jmxremote.port=9004 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.net.preferIPv4Stack=true -Djava.rmi.server.hostname=10.110.3.63 -jar monitor_tuning.jar &
````

#### 基于Btrace的监控调优

windows环境安装：

1.新建环境变量：BTRACE_HOME

2.添加Path：%BTRACE_HOME%\bin

运行方式：

第一种：在JVisualVM中添加Btrace插件，添加classpath

第二种：使用命令行`btrace <pid> <trace_script>`

使用：

拦截普通方法：@OnMethod(clazz="",mehtod="")

拦截构造函数：@OnMethod(clazz="",method="<init>")

拦截同名函数，用参数区分：

在方法入口进行拦截：Kind.ENTRY

在方法返回进行拦截：Kind.RETURN

在方法发生异常拦截：Kind.THROW

在方法某行进行拦截：Kind.Line

拦截this：@Self

拦截入参：可以用AnyType，也可以用真实类型，同名的用真实的

拦截返回：@Return

拦截简单类型：直接获取

拦截复杂类型：反射，类名+属性名

注意：

<u>默认只能本地运行</u>

<u>生产环境下可以使用，但是被修改的字节码不会被还原</u>

#### Tomcat调优

tomcat远程debug：

使用了：jdwp（Java Debug Wire Protocol）技术

1.修改/bin/startup.sh：最后一行start之前，加上jpda

2.修改/bin/catalina.sh：搜索JPDA，

其实就是在启动脚本中加入了一段配置：`-agentlib:jdwp=transport=dt_socket,address=54321,server=ysuspend=n`

tomcat-manager监控：

文档：docs/manager-howto.html

1.conf/tomcat-users.xml添加用户

````xml
<role rolename="tomcat"/>
<role rolename="manager-status"/>
<role rolename="manager-gui"/>
<user username="tomcat" password="123456" roles="tomcat,manager-gui,manager-status"/>
````

2.新建conf/Catalina/localhost/manager.xml配置允许的远程连接

````xml
<?xml version="1.0" encoding="UTF-8"?>
<Context privileged="true" antiResourceLocking="false" docBase="${catalina.home}/webapps/manager">
	<Value className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.0\.0\.1" />
</Context>
````

3.重启

psi-probe监控：

1.github上找到对应项目，到本地打包。`mvn clean package -Dmaven.test.skip`

2.也需要配置tomcat的用户等信息

tomcat调优：

线程优化：

docs/config/http.html

maxConnections：最大连接数

acceptCount：请求超出后，超出的请求，先压到一个队列中

maxThreads：工作线程数

minSpareThreads：最小空闲的工作线程

配置优化：

1.docs/config/host.html

zutoDeploy：tomcat会周期性的检查是否有新的资源被部署。默认为true，生产环境应设置为false。

2.docs/config/http.html

enableLookups：false

3.docs/config/context.html

reloadable：false

4.conf/server.xml

适用于大并发环境下的apr连接器，使用了很多原生的native方法。protocol="org.apache.coyote.http11.Http11AprPortocol"

5.如果不使用自带的session，可以禁用session，提高性能。

#### Nginx性能监控与调优

修改yum源：

````shell
cat /etc/yum.repos.d/nginx.repo
vim nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
````

查看nginx编译参数：`nginx -V`

监控nginx连接信息：

使用ngx_http_stub_status配置：

添加配置：

location = /nginx_status{

	stub_status on;

	access_log off;

	allow 127.0.0.1;

	deny all;

}

监控nginx请求信息：

ngxtop安装：

1.安装python：

`yum install epel-release`

`yum install python-pip`

2.安装ngxtop

`pip install ngxtop`

常用指令：

指定配置文件：`ngxtop -c /etc/nginx/nginx.conf`

查询状态是200：`ngxtop -c /etc/nginx/nginx.conf -i 'status == 200'`

查询访问最多ip：`ngxtop -c /etc/nginx/nginx.conf -g remote addr`

Nginx-rrd监控：

......

Nginx优化：

增加工作线程数和并发连接数：

worker_processes 4;#cpu

events{

	worker_connections 10240;#每一个进程打开的最大连接数，包含了nginx与客户端和nginx与upstream之间的连接

	multi_accept on;#可以一次建立多个连接

	use epoll;

}

启用长连接：

upstream server_pool{

	server localhost:8080 weight=1 max_fails=2 fail_timeout=30s;

	server localhost:8081 weight=1 max_fails=2 fail_timeout=30s;

	keepalive 300;#300个长连接

}

启用缓存、压缩：

gzip on;

gzip_http_version 1.1;

gzip_disable "MSIE [1-6]\.(?!.*SV1)";

gzip_proxied any;

gzip_types text/plain text/css application/javascript application/x-javascript application/json application/xml application/vnd.ms-fontobject application/x-font-ttf application/svg+xml application/x-icon;

gzip_cary on;

gzip_station on;#如果有压缩好的直接使用

操作系统优化：

tcp/ip相关配置：

配置文件：/etc/sysctl.conf

sysctl -w net.ipv4.tcp_syncookies=1 #防止一个套接字在有过多视图连接到达时引起过载

sysctl -w net.core.somaxconn=1024#默认128，连接队列

sysctl -w net.ipv4.tcp_fin_timeout=10 #timewait的超时时间

sysctl -w net.ipv4_tw_reuse=1 #os直接使用timewait的连接

sysctl -w net.ipv4.tcp_tw_recycle=0 #回收禁用

每个进程打开的文件数配置：

配置文件：/etc/security/limits.conf

*hard nofile 204800

*soft nofile 204800

*soft core unlimited

*soft stack 204800

其他配置：

sendfile        on;#减少文件在应用和内核之间拷贝

tcp_nopush   on;#当数据包达到一定大小再发送

tcp_nodelay  off;#有数据随时发送

#### GC调优

垃圾收集器：

串行收集器Serial：Serial、Serial Old

并行收集器Parallel：Parallel Scavenge、Parallel Old（吞吐量优先）

并发收集器Concurrent：CMS、G1（停顿时间优先）

	并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。适合科学计算、后台处理等弱交互场景。

	并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），垃圾收集线程在执行的时候不会停顿用户程序的运行。适合对响应时间有要求的场景，比如web。

	停顿时间：垃圾收集器做垃圾回收中断应用执行的时间。`-XX:MaxGCPauseMillis`

      吞吐量：花在垃圾收集的时间和花在应用时间的占比。`-XX:GCTimeRatio=<n>`，垃圾收集时间占：1/1+n

	一个优秀的垃圾收集配置或垃圾收集算法。指的是在吞吐量最大的时候，停顿时间最小。但，现实情况往往是吞吐量越高，停顿时间越长。这也是GC调优的目的。

开启串行收集器：新生代`-XX:+UseSerialGC`、老年代`-XX:+UseSerialOldGC`

开启并行收集器：新生代`-XX:+UseParallelGC`、老年代`-XX:+UseParallelOldGC`（Server模式下默认的垃圾收集器）

开启并发收集器：新生代`-XX:+UseParNewGC`、老年代`-XX:+UseConcMarkSweepGC`

开启G1：`-XX:+UserG1GC`

如何选择垃圾收集器：

优先调整堆的大小，让服务器自己来选择

如果内存小于100M，使用串行收集器

如果是单核，并且没有停顿时间的要求，串行或者JVM自己选

如果允许停顿时间超过1秒，选择并行或者JVM自己选

如果响应时间最重要，并且不能超过1秒，使用并发收集器

垃圾收集器常用参数：

Parallel Collector：

`-XX:+UseParallelGC`，手动开启，Server默认开启

`-XX:ParallelGCThreads=<N>`，多少个GC线程（CPU>8，N=5/8），（CPU<8 N=CPU）

并行垃圾收集器，有自适应的特性。

`-XX:MaxGCPauseMillis=<N>`，最大停顿时间

`-XX:GCTimeRatio=<N>`，吞吐量

`-Xmx<N>`，最大堆的大小

这种收集器会优先满足最大停顿时间的要求，然后再依次满足吞吐量，堆的大小等要求。会动态调整堆的各个分区大小。

`-XX:YoungGenerationSizeIncrement=<Y>`，动态调整young区大小，默认值20%

`-XX:TenuredGenerationSizeIncrement=<T>`，动态调整old区大小，默认值20%

`-XX:AdaptiveSizeDecrementScaleFactor=<D>`，动态减小堆的大小，默认值4%

CMS Collector：

并发收集、低停顿、低延迟、老年代收集器

1.CMS initial mark：出师表集Root，STW

2.CMS Concurrent mark：并发标记

3.CMS-concurrent-preclean：并发预清理

4.CMS remark：重新标记，STW

5.CMS concurrent sweep：并发清楚

6.CMS-concurrent-reset：并发重置

缺点：CPU敏感、浮动垃圾、空间碎片

`-XX:ConcGCThreads`，并发的GC线程数

`-XX:+UseCMSCompactAtFullCollection`，FullGC之后做压缩

`-XX:CMSFullGCsBeforeCompaction`，多少次FullGC之后压缩一次

`-XX:CMSInitiationOccupancyFraction`，触发FullGC（默认92%）

`-XX:+UseCMSInitiatingOccupancyOnly`，是否动态调整

`-XX:+CMSScavengeBeforeRemark`，FullGC之前先做YGC

`-XX:+CMSClassUnloadingEnabled`，启用回收Perm区

iCMS：使用于单核或者双核

G1 Collector：

新生代和老年代收集器，适用于大内存

Region

SATB：Snaphot-At-The-Beginning，它是通过Root Tracing得到的，GC开始的时候存活对象的快照。

RSet：记录了其他Region中的对象引用本Region中对象的关系，术语points-into结构（谁引用了我的对象）。

发生YoungGC的时候：新对象进入Eden区，存活对象拷贝到Survivor区，存活时间达到年龄阈值时，对象晋升到Old区。

接下来会发生MixedGC：不是FullGC，回收所有的Young和部分Old。触发一次，global concurrent marking（全局并发标记）。

1.Initial marking phase：标记GC Root，STW

2.Root region scanning phase：标记存活Region

3.Concurrent marking phase：标记存活的对象

4.Remark phase：重新标记，STW

5.Cleanup phase：部分STW

MixedGC相关参数：

InitiatingHeapOccupancyPercent：堆占有率达到这个数值则触发flobal concurrent marking，默认45%

G1HeapWastePercent：在clobal concurrent marking结束之后，可以知道有多少空间要被回收，在每次YGC之后和再次发生Mixed GC之前，会检查垃圾占比是否达到此参数，只有达到了，下次才会发哼Mixed GC。

G1MixedGCLiveThresholdPercent：Old区的region被回收的时候的存活对象占比

G1MixedGCCountTarget：一次global concurrent marking之后，最多执行Mixed GC的次数

G1OldCSetRegionThresholdPercent：一次Mixed GC中能被选入CSet的最多Old区的region数量

G1常用参数：

`-XX:+UseG1GC`，开启G1

`-XX：G1HeapRegionSize=n`，region的大小，1-32M，最多2048个

`-XX:MaxGCPauseMillis=200`，最大停顿时间

`-XX:G1NewSizePercent`，young区大小

`-XX:G1MaxNewSizePercent`，young区最大大小

`-XX:G1ReservePercent=10`，保留防止to space溢出

`-XX:ParallelGCThreads=n`，STW线程数

`-XX:ConcGCThreads=n`，并发线程数=1/4*并行

G1最佳实践：

年轻代大小：避免使用-Xmn、-XX:NewRatio等显式设置Young区大小，会覆盖暂停时间目标。

暂停时间目标：暂停时间不要太严苛，其吞吐量目标时90%的应用程序时间和10%的垃圾回收时间，太严苛会直接影响到吞吐量。

#### GC可视化工具

打印GC日志相关参数：

`-XX:+PrintGCDetails`

`-XX:+PrintGCTimeStamps`

`-XX:+PrintGCDateStamps`

`-Xloggc:$CATALINA_HOME/logs/gc.log`

`-XX:+PrintHeapAtGC`

`-XX:+PrintTenuringDistribution`

MixGC调优：

`-XX:InitiatingHeapOccupancyPercent`

`-XX:G1MixedGCLiveThresholdPercent`

`-XX:G1HeapWastePercent`

`-XX:G1MixedGCCountTarget`

`-XX:G1OldCSetRegionThresholdPercent`

Mixed GC的时机：

关于MixGC调优：堆占有率达到这个数值则触发global concurrent marking，默认45%

G1HeapWastePercent：在global concurrent marking结束之后，可以知道有多少空间要被回收，在每次YGC之后和再次发生Mixed GC之前，会检查垃圾占比是否达到此参数，只有达到了，才会发生Mixed GC。

两款GC调优可视化工具：GCeasy、GCviewer

GC调优步骤：

1.打印GC日志

2.根据日志得到关键性能指标（吞吐量、停顿时间）

3.分析GC原因，调整相关的JVM参数

初始设置：

````shell
-XX:+DisableExplicitGC -XX:+HeapDumpOnOutOfMemoryErroe -XX:HeapDumpPath=$CATALINA_HOME/logs/ -XX:+PrintGCDetails -Xloggc:$CATALINA_HOME/logs/gc.log
````

#### Parallel GC调优的原则

1.除非确定，否则不要设置最大堆内存

2.优先设置吞吐量目标

3.如果吞吐量目标达不到，调大最大内存，不能让操作系统使用Swap，如果仍然达不到，降低目标

4.如果吞吐量能达到，GC时间太长，设置停顿时间的目标

#### G1 GC调优的原则

1.年轻代大小：避免使用-Xmn、-XX：NewRatio等显式设置Young区大小，会覆盖暂停时间目标

2.暂停时间目标：暂停时间不要太严苛，其设计目标是90%的应用程序时间和10%的垃圾回收时间，太严苛会直接影响吞吐量

3.调优MixGC（参考上面的参数释义）

