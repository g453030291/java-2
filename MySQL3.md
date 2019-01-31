#### 一、MySQL架构：

![mysql架构图2](https://github.com/g453030291/java-2/blob/master/images/mysql架构图2.png)

两层架构：service层、存储引擎层

service又包括：连接器、查询缓存、分析器、优化器、执行器。

存储引擎层：插件式架构，支持InnoDB、MyISAM等存储引擎。

连接器：负责和客户端建立连接、获取权限、维持和管理连接。

查询缓存：mysql拿到请求后，会先去缓存查看，如果命中，会直接返回。mysql8.0已经去掉查询缓存。

分析器：分析器会将你输入的sql解析，并分析语法是否正确。

优化器：将你输入的sql优化，选择索引，关联表连接等。

执行器：调用对应表的存储引擎，开始对对应表每个行进行扫描，找到你要查找的数据。

#### 二、MySQL的redo log和binlog：

redo log：是InnoDB所自带的磁盘日志记录。

binlog：是mysql在service层自带的日志记录。

区别：

1.redo log是InnoDB引擎特有的；binlog是MySQL的service层实现，所有引擎都可以使用。

2.redo log是物理日志，记录的是“在某个数据页上做了什么修改”；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如“给ID=2这一行的c字段加1”。

3.redo log是循环写的，空间会固定用完；binlog是追加写入的。“追加写”是指binlog文件写到一定大小会切换到下一个，并不会覆盖以前的日志。

redo log的执行逻辑图：

![redolog逻辑图](https://github.com/g453030291/java-2/blob/master/images/redolog逻辑图.jpg)

redo log是固定大小，比如可如图配置一组4个固定大小文件，每个1G。write pos是当前记录的位置，一边写一边后移。checkpoint是擦除，并写到磁盘的更新点，也是往后移并循环的。checkpoint和write pos之间是空着的部分。

有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个特性被称作：crash-safe。

执行一次数据库update流程图：

![update流程](https://github.com/g453030291/java-2/blob/master/images/update流程.jpg)







