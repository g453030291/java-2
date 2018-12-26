### 一、MySQL架构介绍

#### 1.查询是否安装过mysql：

rpm -qa|grep -i mysql

删除命令：rpm -e RPM软件包名（该名称是上一个命令查询出来的名字）

#### 2.安装mysql服务端：

rpm -ivh MySQL-server-5.5.48-l.Linux2.6.i308.rpm

客户端：rpm -ivh MySQL-client-5.5.48-l.Linux2.6.i308.rpm

#### 3.查看是否安装成功：

查看后台进程：ps -ef|grep mysql

查看mysql的用户：cat /etc/passwd|grep mysql

查看mysql的用户组：cat /etc/group|grep mysql

查看mysql的版本：mysqladmin --version

以后台的方式启动mysql服务：service mysql start

停止mysql服务：service mysql stop

启动mysql后，第一次进入输入mysql，可以直接进入。

设置mysql root用户密码：/usr/bin/mysqladmin -u root password 123456

开机自动启动myql服务：chkconfig mysql on

查看是否修改成功：chkconfig --list|grep mysql

查看是否修改成功：ntsysv

查看mysql的安装位置：

![mysql安装位置](https://github.com/g453030291/java-2/blob/master/images/mysql安装位置.png)

修改mysql配置文件：cp /usr/share/mysql/my-huge.cnf  /etc/my.cnf(mysql5.5版本)

				      cp /usr/share/mysql/my-default.cnf  /etc/my.cnf(mysql5.6版本)

查看mysql默认的字符集：show variables like 'character%'

					      show variables like '%char%'

修改mysql字符集：

![修改mysql字符集](https://github.com/g453030291/java-2/blob/master/images/修改mysql字符集.png)

#### 4.mysql主要的配置文件

二进制日志log-bin：主要用于主从复制

```shell
#Replication Master Server(default)
#binary logging is required ffor replication
log-bin=mysql-bin
```

错误日志log-error：默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等

查询日志log：默认关闭，记录查询的sql语句，如果开启会降低myslql的整体性能，因为记录日志也是需要消耗系统资源的

数据文件：windows下data目录中

		   linux下：查看系统中所有数据库ls -1F|grep ^d

		                 默认路径：/var/lib/mysql

		    frm文件：存放表结构

		    myd文件：存放表数据

		    myi文件：存放表索引 						

如何配置：windows下的my.ini

		   linux下/etc/my.cnf

5.mysql架构

和其他数据库相比，mysql有点与众不同，他的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上，插件式的存储引擎架构将查询和处理其他的系统任务以及数据的存储提取相分离。这种架构可以根据业务需求和实际需要选择合适的存储引擎。

![mysql架构图](https://github.com/g453030291/java-2/blob/master/images/mysql架构图.png)

6.mysql存储引擎：

查看mysql提供的存储引擎：show engines；

查看mysql当前默认的存储引擎：show variables like '%storage_engine%';

7.myisam和innodb:

![myisam和innodb](https://github.com/g453030291/java-2/blob/master/images/myisam和innodb.png)

### 二、索引优化分析

#### 1.性能下降sql慢，执行时间长，等待时间长

1.查询语句写得烂

2.索引失效：单值索引，复合索引

3.关联查询太多join

4.服务器调优及各个参数设置（缓冲、线程数等）

#### 2.常见通用的join查询

1.sql执行顺序

手写：select、from、join、on、where、group by、having、order by、limit

机读：from、on、join、where、group by、having、select、distinct、order by、limit

总结：

![sql关键字执行顺序总结](https://github.com/g453030291/java-2/blob/master/images/sql关键字执行顺序总结.png)

2.七种join类型：

![七种join类型A](https://github.com/g453030291/java-2/blob/master/images/七种join类型A.png)

![七种join类型B](https://github.com/g453030291/java-2/blob/master/images/七种join类型B.png)

#### 3.索引简介

索引是什么？

1.mysql官方对索引的定义为：索引是帮助mysql高效获取数据的数据结构。可以得到索引的本质：索引是数据结构。

2.可以简单理解为“排好序的快速查找数据结构”

3.在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。

![索引和数据的关系](https://github.com/g453030291/java-2/blob/master/images/索引和数据的关系.png)

为了加快col2的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取到相应数据，从而快速的检索出符合条件的记录。

4.一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上

5.我们平常所说的索引，如果没有特别说明，都是指B树（多路搜索树，并不一定是二叉的）结构组织的索引。其中聚集复合索引，前缀索引，唯一索引默认都是使用B+树索引，统称索引。除了B+树这种以外，还有哈希索引等其它类型索引。

索引优势：

查找：建立数据的索引，提高数据检索的效率，降低数据库的IO成本（影响where后的查找条件）

排序：通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗（影响order by后的排序）

索引劣势：

1.实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占用空间的

2.虽然索引提高了查询速度，但是会降低更新表的速度

3.索引只是提高效率的一个因素，如果你的mysql有大数据量的表，就需要花时间研究最优秀的索引，或优化查询语句

mysql索引分类：

1.单值索引：即一个索引只包含单个列，一个表可以有多个单列索引

2.唯一索引：索引类的值必须唯一，但允许有空值

3.复合索引：即一个索引包含多个列

4.基本语法：

创建：create [unique] index indexName on mytable(columnname(length));

           Alert mytable add [unique] index [indexname] on (columnname(length));

删除：drop index [indexName] on mytable;

查看： show index form table_name

使用alert添加索引：

![alert添加索引](https://github.com/g453030291/java-2/blob/master/images/alert添加索引.png)

mysql索引结构：

1.BTree索引：原理

![磁盘中的索引树](https://github.com/g453030291/java-2/blob/master/images/磁盘中的索引树.png)

![btree索引原理](https://github.com/g453030291/java-2/blob/master/images/btree索引原理.png)

2.Hash索引

3.full-text全文索引

4.R-Tree索引

哪些情况需要建立索引？

1.主键自动建立唯一索引

2.频繁作为查询条件的字段应该创建索引

3.查询中与其他表关联的字段，外键关系建立索引

4.频繁更新的字段不适合创建索引，因为每次更新需要更新数据和索引

5.where条件里用不到的字段不创建索引

6.单键/组合索引的选择问题，在高并发下倾向创建组合索引

7.查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度

8.查询中统计或者分组字段

哪些情况不需要建立索引？

1.表记录太少

2.经常增删改的表

3.数据重复且分布平均的表字段

索引的选择性：

![索引的选择性](https://github.com/g453030291/java-2/blob/master/images/索引的选择性.png)

#### 4.性能分析

 

#### 5.索引优化