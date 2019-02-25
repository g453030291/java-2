#### JVM调优

##### CPU飙高的解决方法

1.使用`top`命令查看系统运行情况

2.使用1，查看各个核心的CPU运行情况

3.使用shift+h，切换到线程模式，找到CPU飙升的线程号

4.使用jstack，将该线程运行的详细情况，dump到a.txt文件中。（jstack + 线程号 > a.txt）

5.将线程号，转为16进制。`printf "%x \n" 39060`

6.将转换后的16进制线程号，到dump文件中查询出对应有问题的代码



`vmstat 1`：每秒钟查看一次CPU的负载



内存查看：

`free -g`：查看内存



io查看：

`iostat -dx 1`：每秒钟查看一次io读写情况。（重点查看三个参数：w_await、svctm、%util）

%util：表达的是io操作占CPU的百分比



查看一个文件夹大小：

`df -h /root`、`du -h root/`：查看root文件夹大小



查看网络情况：

`nicstat -l`



##### 术语

吞吐量：单位之间内完成工作的数量

平均相应时间：一个请求完成的时间，从请求到相应。

TPS：

QPS：一次查询时间



查看系统内所有java应用pid：jps

查看某个java应用的资源dump：jmap -heap （pid号）



##### 什么时候可能会触发STW的Full GC？

1.Perm空间不足。

2.CMS GC时出现promotion failed和concurrent mode failure（concurrent mode failure发生的原因一般是CMS正在进行，但是由于老年代空间不足，需要尽快回收老年代里面不再被使用的对象，这时停止所有的线程，同时终止CMS，直接进行Serial Old GC）；

3.统计得到的Young GC晋升到老年代的平均大小大于老年代的剩余空间；

4.主动触发Full GC（执行jmap -histo:live [pid]）来避免碎片问题。

