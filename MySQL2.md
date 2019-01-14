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



#### 4.Show Profile

#### 5.全局查询日志