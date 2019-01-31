### 三、查询截取分析

#### 1.查询优化

##### 1.永远小表驱动大表，类似全套循环Nested Loop

示例：

![exists和in](https://github.com/g453030291/java-2/blob/master/images/exists和in.png)

![exists用法](https://github.com/g453030291/java-2/blob/master/images/exists用法.png)

##### 2.order by关键字优化

1.order by子句，尽量使用index方式排序，避免使用FileSort方式排序。mysql支持两种方式排序，FileSort和Index，Index效率高，它指mysql扫描索引本身完成排序。FileSort方式效率较低。Order By满足两种情况，会使用Index方式排序：一、order by语句使用索引最左前列；二、使用where子句与order by子句条件列组合满足索引最左前列。

2.尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀。

3.如果不在索引列上，filesort有两种算法：mysql会启动双路排序和单路排序。

双路排序：mysql4.1之前是使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出。从磁盘取排序字段，在buffer进行排序，再从磁盘取数据。

       取一批数据，要对磁盘进行两次扫描，众所周知，I\O时很耗时的，所以从mysql4.1之后，优化排序为单路排序。

单路排序：从磁盘读取查询需要的所有列，按照orderby列在buffer对他们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。

结论：由于单路时后出的，总体而言好过双路。但是用单路有问题：在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出，所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排序完再取sort_buffer容量大小，再排序......从而多次IO。本来想省一次IO操作，反而造成了大量的IO操作， 得不偿失。

4.优化策略：增大sort_buffer_size参数设置、增大max_length_for_sort_data参数设置

why：

![why优化策略](https://github.com/g453030291/java-2/blob/master/images/why优化策略.png)

排序orderby使用索引总结：

![排序使用索引总结](https://github.com/g453030291/java-2/blob/master/images/排序使用索引总结.png)

##### 3.group by关键字优化

1.group by实质是先排序后进行分组，遵照索引键的最佳左前缀

2.当无法使用索引列时，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置

3.where高于having，能写在where限定的条件就不要去having限定了

#### 2.慢查询日志

![慢查询日志1](https://github.com/g453030291/java-2/blob/master/images/慢查询日志1.png)

默认情况下，mysql数据库没有开启慢查询日志，需要手动设置参数开启。当然，如果不是调优需要，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定性能影响。慢查询日志支持将日志记录写入文件。

查看是否开启：show variables like '%slow_query_log%';

开启：set global slow_query_log=1;(只对当前数据库生效，且数据库重启失效)

要永久生效，就必须修改配置文件my.cnf。

修改my.cnf文件，[mysqld]下增加或修改参数

slow_query_log和slow_query_log_file后，重启mysql服务。配置如下：

slow_query_log=1

slow_query_log_file=/var/lib/mysql/xxxxx-slow.log

![设置慢查询参数](https://github.com/g453030291/java-2/blob/master/images/设置慢查询参数.png)

查看当前多少秒算慢：show variables like 'long_query_time%';

设置慢的阙值时间：set global long_query_time=3;

需要重新连接或新开一个会话才能看到修改值：show global variables like 'long_query_time%';

查看当前系统有多少条慢sql：show global status like '%slow_queries%';

my.cnf配置版：

[mysqld]下配置：

Slow_query_log=1

Slow_query_log_file=/var/lab/mysql/xxxx-slow.log

Long_query_time=3

Log_output=FILE

日志分析工具mysqldumpslow：在生产环境中，如果手动分析日志，查找分析sql，可以使用mysql提供的日志分析工具mysqldumpslow。

使用mysqldumpslow --help查看帮助信息。

![mysqldumpslow帮助信息](https://github.com/g453030291/java-2/blob/master/images/mysqldumpslow帮助信息.png)

![mysqldumpslow使用示例](https://github.com/g453030291/java-2/blob/master/images/mysqldumpslow使用示例.png)

#### 3.批量数据脚本

插入1000万条数据：

1.建表

```sql
create table dept{
id int unsigned primary key auto_increment,
deptno mediumint unsigned not null default 0,
dname varchar(20) not null default "",
loc varchar(13) not null default ""
}engine=innodb default charset=gbk;

create table emp{
id int unsigned primary key auto_increment,
empno mediumint unsigned not null default 0,"编号"
ename varchar(20) not null default "","名字"
job varchar(9) not null default "","工作"
mgr mediumint unsigned not null default 0,"上级编号"
hiredate date not null,"入职时间"
sal decimal(7,2) not null,"薪水"
comm decimal(7,2) not null,"红利"
deptno mediumint unsigned not null default 0"部门编号"
}engine=innodb default charset=gbk;
```

2.添加配置参数

[mysqld]下添加：log_bin_trust_gunction_creators=1

3.创建函数，保证每条数据都不同

```sql
##自定义产生随机字符串的sql函数
delimiter $$
create function rand_string(n int) returns varchar(255)
begin
 declare chars_str varchar(100) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
 declare return_str varchar(255) default '';
 declare i int default 0;
 while i < n do
 set return_str = concat(return_str,substring(chars_str,floor(1+rand()*52),1));
 set i = i+1;
 end while;
 return return_str;
end $$

##自定义随机产生部门编号的函数
delimiter $$
create function rand_num()
returns int(5)
begin
 declare i int default 0;
 set i = floor(100+rand()*10);
return i;
 end $$
 
 ##删除自定义函数
 drop function rand_num;
```

4.创建存储过程

```sql
#往emp表中添加数据的存储过程
delimiter $$
create procedure insert_emp(in start int(10),in max_num int(10))
bginx
declare i int default 0;
#set autocommit = 0 把autocommit设置成0
 set autommit = 0;
 repeat
 set i = i + 1;
 insert into emp(empno,ename,job,mgr,hiedate,sal,comm,deptno)values((start+i),rand_string(6),'salesman',0001,curdate(),2000,400,rand_num());
 until i + max_num
 end repeat;
 commit;
 end $$
 
 #往dept表添加随机数据的存储过程
 delimiter $$
 create procedure insert_dept(in start int(10),in max_num int(10))
 begin
 declare i int default 0;
  set autocommit = 0;
  repeat
  set i = i + 1;
  insert into dept(deptno,dname,loc)values((start+i),rand_string(10),rand_string(8));
  until i + max_num
  end repeat;
  commit;
  end $$
```

5.调用存储过程

```sql
#先往部门表添加10个部门
delimiter;
call insert_dept(100,10);
#再调用添加员工，插入50w条员工
delimiter;
call insert_emp(100001,500000);

```

#### 4.Show Profile

是什么：是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量

默认情况下，参数处于关闭状态，并保存最近15次的运行结果

分析步骤：

1.是否支持：

show variables like 'profiling';

2.设置开启：

set profiling=on;

3.运行一些耗时SQL：

```sql
select * from emp group by id%10 limit 150000;
select * from emp group by id%20 order by 5;
```

4.查看结果，show profiles；

5.诊断SQL，show profile cpu, block io for query 3;(这里的3代表上边show profiles查询出的id)

6.可选参数

all                                 显示所有开销信息

block io                       显示块io相关开销

context switches       上下文切换相关开销

cpu                              显示cpu相关开销信息

ipc                                显示发送和接受相关开销信息

memory                      显示内存相关开销信息

page faults                 显示页面错误相关开销信息

source                         显示和source_function,source_file,source_line相关的开销信息

Swaps                          显示交换次数相关开销的信息

7.需要注意的参数：

converting HEAP to MyISAM：查询结果太大，内存都不够用了，往磁盘上搬了。

creating tmp table：创建临时表：导致拷贝数据到临时表，用完再删除。

copying to tmp table on disk：把内存中临时表复制到磁盘。

locked：

#### 5.全局查询日志

（不要在生产环境开启）

在mysql的my.cnf中，设置：

开启：general_log=1

记录日志文件的路径：general_log_gile=/path/logfile

输出格式：log_output=FILE

查询记录的sqllog：select * from mysql.general_log;

### 四、MySQL锁机制

#### 概述：

	锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、IO）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

锁的分类：

	从对数据库的操作类型区分，分为读锁和写锁。从对数据库操作粒度区分，分为表锁和行锁。
	
	读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
	
	写锁（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。

#### 重点介绍：

##### 表锁（偏读）：

特点：偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。

读锁案例：

![读锁案例1](https://github.com/g453030291/java-2/blob/master/images/读锁案例1.png)

![读锁案例2](https://github.com/g453030291/java-2/blob/master/images/读锁案例2.png)

![读锁案例3](https://github.com/g453030291/java-2/blob/master/images/读锁案例3.png)

写锁案例：

![写锁案例1](https://github.com/g453030291/java-2/blob/master/images/写锁案例1.png)

![写锁案例2](https://github.com/g453030291/java-2/blob/master/images/写锁案例2.png)

案例结论：

![案例结论1](https://github.com/g453030291/java-2/blob/master/images/案例结论1.png)

![案例结论2](https://github.com/g453030291/java-2/blob/master/images/案例结论2.png)

表锁分析：

查看哪些表被加锁了：show open tables；

如何分析表锁定：可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定；

![分析表锁](https://github.com/g453030291/java-2/blob/master/images/分析表锁.png)

##### 行锁（偏写）：

特点：偏向InnoDB存储引擎，开销大，加锁慢，会出现死锁；锁定力度最小，发生锁冲突的概率最低，并发度也最高。InnoDB与MyISAM的最大不同有两点：一是支持事物（TRANSACTION）；二是采用了行级锁。

事物相关的知识：

![ACID特性](https://github.com/g453030291/java-2/blob/master/images/ACID特性.png)

并发处理带来的问题：

更新丢失（Lost Update）：当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其它事务的存在，就会发生丢失更新问题——最后的更新覆盖了由其它事务所做的更新。例如，两个程序员修改同一java文件。每个程序员独立的更改其副本，然后保存更改后的副本，这样就覆盖了原始文档。最后保存其更改副本的编辑人员覆盖前一个程序员所做的更改。如果在一个程序员完成并提交事物之前，另一个程序员不能访问同一文件，则可避免此问题。

脏读（Dirty Reads）：一个事务正在对一条记录做修改，在这个事务完成并提交前，这记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读”。例：事务A读取到了事务B已经修改但尚未提交的数据，还在这个数据基础上做了操作。此时，如果B事务回滚，A读取的数据无效，不符合一致性要求。

不可重复读（Non-Repeatable Reads）：一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变，或某些记录已经被删除了。这种现象叫“不可重复读”。例：事务A读取到了事务B已经提交的修改数据，不符合隔离性。

幻读（Phantom Reads）：一个事务按相同的查询条件重新读取以前检索过的数据，却发现其它事务插入了满足其查询条件的新数据，这种现象就称为“幻读”。例：事务A读取到了事务B提交的新增数据，不符合隔离性。

区别：脏读是事务B里面修改了数据，

            幻读是事务B里面新增了数据。

事务的隔离级别：

![隔离级别](https://github.com/g453030291/java-2/blob/master/images/隔离级别.png)

案例：

![行锁案例1](https://github.com/g453030291/java-2/blob/master/images/行锁案例1.png)

索引使用不当导致行锁升级为表锁：

	update的时候，如果where后条件列是加索引的，而且又是varchar类型，这时候，不给参数加单引号，会导致索引失效，进一步会将行锁升级为表锁。

间隙锁的危害：

![间隙锁1](https://github.com/g453030291/java-2/blob/master/images/间隙锁1.png)

![间隙锁2](https://github.com/g453030291/java-2/blob/master/images/间隙锁2.png)

如何锁定一行：

![锁定一行](https://github.com/g453030291/java-2/blob/master/images/锁定一行.png)

总结：

	Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面带来的性能损耗可能比表级锁定会更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了。
	
	但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会使Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

行锁分析：

![行锁分析1](https://github.com/g453030291/java-2/blob/master/images/行锁分析1.png)

![行锁分析2](https://github.com/g453030291/java-2/blob/master/images/行锁分析2.png)

优化建议：

1.尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。

2.合理设计索引，尽量缩小锁的范围

3.尽可能较少检索条件，避免间隙锁

4.尽量控制事务大小，减少锁定资源量和时间长度

5.尽可能低级别事务隔离

##### 页锁：

开锁和加锁时间介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般。

### 五、主从复制

#### 复制的基本原理

slave会从master读取binlog来进行数据同步。

![复制原理](https://github.com/g453030291/java-2/blob/master/images/复制原理.png)

#### 复制的基本原则

每个slave只有一个master

每个slave只能有一个唯一的服务器id

每个master可以有多个slave

#### 复制的最大问题

延时

#### 一主一从常见配置

mysql版本一致且后台以服务运行

主从都配置在[mysqld]节点下，都是小写

my.ini配置

```properties
#主服务器唯一ID
server-id=1
#启用二进制日志
log-bin=自己本地的路径/mysqlbin
#启用错误日志
log-err=自己本地路径/mysqlerr
#根目录
basedir="自己本地路径"
#临时目录
tmpdir="自己本地路径"
#数据目录
datadir="自己本地路径/data"
#主机读写都可以
read-only=0
#设置不要复制的数据库
binlog-ignore-db=mysql
#设置需要复制的数据库
binlog-do-db=需要复制的数据库名
```

my.cnf

```properties
#从服务器唯一ID
server-id=2
#启用二进制日志
```

改过配置后，需要重启mysql服务

在主机上建立账户，授权从机访问：

```sql
grant replication slave on *.* to 'zhangsan'@'从机数据库IP' identified by '123456'
```

刷新配置

```sql
flush privileges
```

查询master状态

```sql
show master status
```

<u>记录下file和position</u>

```sql
change master to master_host='主机ip',master_user='zhangsan',masterr_password='123456',master_log_file='file名字',master_log_pos=’position数字'
```

启动从服务器复制功能

```sql
start slave
```

查看从机状态

```sql
show slave status\G
```

这里出现slave_io_running:yes;和slave_sql_running:yes；表示配置成功。

停止从服务器复制功能

```sql
stop slave
```

