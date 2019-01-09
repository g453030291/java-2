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

 MySQL Query Optimizer

1.Mysql中有专门负责优化select语句的优化器模块，主要功能：通过计算分析系统中收集到的统计信息，为客户端请求的Query提供他认为最优的执行计划（他认为最优的数据检索方式，但不见得是DBA认为是最优的）。

2.当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是SELECT并转发给MySQL Query Optimizer时，MySQL Query Optimizer首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值，并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等。然后分析Query中的Hint信息（如果有），看显示Hint信息是否可以完全确定该Query的执行计划。如果没有Hint或Hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据Query进行写相应的计算分析，然后再得出最后的执行计划。

MySQL常见瓶颈

CPU：CPU在饱和的时候一般发生在数据库装入内存或从磁盘上读取数据的时候

IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候

服务器硬件的性能瓶颈：top，free，iostat和vmstat来查看系统性能状态

Explain（***）

1.使用explain关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。

2.explain结果分析：

重点：id、type、key、rows、extra

##### id：

表示表的读取顺序。select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。有三种情况：1.id相同，执行顺序从上至下；2.id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行；3.id相同或不同，同时存在，可以认为id如果相同都属于同一个组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行。（如果table变成derived，表示这个表是子查询创建出的虚表，后边的数字，表示查询顺序）

##### select_type：

数据读取操作的操作类型。枚举值有六种，1.simple，2.primary，3.subquery，4.derived，5.union，6.union result。查询的类型，主要是用于区别普通查询、联合查询、子查询等复杂查询。

| 枚举值       | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| simple       | 简单的select查询，查询中不包含子查询或union                  |
| primary      | 查询中若包含任何复杂的子部分最外层查询则被标记为primary      |
| subquery     | 在select或where列表中包含了子查询                            |
| derived      | 在form列表中包含的子查询被标记为derived（衍生），MySQL会递归执行这些子查询，把结果放在临时表里。 |
| union        | 若第二个select出现在union之后，则被标记为union。若union包含在form子句的子查询中，外层select将被标记为：derived。 |
| union result | 从union表获取结果的select                                    |

table：显示这一行的数据是关于那张表的。

##### type：

表示访问类型。有八种枚举值：1.all，2.index，3.range，4.ref，5.eq_ref，6.const，7.system，8.null。从最好到最差依次是：system>const>eq_ref>ref>range>index>all。只需要到range或ref即可。

| 枚举值 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| system | 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计 |
| const  | 表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，mysql就能将该查询转换为一个常量 |
| eq_ref | 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描 |
| ref    | 非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体 |
| range  | 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是在你的where语句中出现了between、<、>、in等的查询。这种范围索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引 |
| index  | Full Index Scan，Index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘中读的） |
| all    | Full Table Scan，将遍历全表以找到匹配的行                    |

##### pollible_keys：

显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出。但不一定被查询实际使用。

##### key：

实际使用的索引。如果为null，表示没有使用索引。

查询中若使用了覆盖索引，则该索引仅出现在key列表中。

##### Key_len:

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好。key_len现实的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。

##### ref:

显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引列上的值

##### rows：

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

##### extra:

重点:using filesort、using temporary、using index

包含不适合在其他列中显示但十分重要的额外信息。

1.using filesort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。mysql中无法利用索引完成的排序操作称为“文件排序”。

2.using temporary：使用了临时表保存中间结果，mysql在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。

3.using index：表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。

覆盖索引的概念有两种：

	1.就是select的数据列只用从索引中就能取得，不必读取数据行，mysql可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，换句话说查询列表要被所建的索引覆盖。

	2.索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了。一个索引包含了（或覆盖了）满足查询结果的数据也就叫做覆盖索引。

注意：如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select *。因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。

4.using where：表明使用了where过滤

5.using join buffer：使用了连接缓存

6.impossible where：where子句的值总是false，不能用来获取任何元素

7.select tables optimized away：在没有group by子句的情况下，基于索引优化min/max操作或者对于MyISAM存储引擎优化count（*）操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

8.distinct：优化distinct操作，在找到第一匹配的元素后即停止找同样值的动作

##### 优化实例1:

![sql优化实例1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例1.png)

![sql实例1答案](https://github.com/g453030291/java-2/blob/master/images/sql实例1答案.png)

#### 5.索引优化

1.索引分析：

##### 单表：

![sql优化实例2](https://github.com/g453030291/java-2/blob/master/images/sql优化实例2.png)

分析sql性能：

![sql优化实例2分析](https://github.com/g453030291/java-2/blob/master/images/sql优化实例2分析.png)

第一次优化：

![sql优化实例2优化1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例2优化1.png)

![sql优化实例2结论1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例2结论1.png)

	第一次试着对where后面的三个条件加上复合索引，解决了全表扫描，没有解决文件内排序。如果where后的条件都用等于，就会全使用索引。不会出现filesort。但实际情况是where后的条件往往是范围查询。所以本次优化失败。

第二次优化：

![sql优化实例2结论2](https://github.com/g453030291/java-2/blob/master/images/sql优化实例2结论2.png)

##### 两表：

![sql优化实例3book表](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3book表.png)

![sql优化实例3class表](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3class表.png)

![sql优化实例3](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3.png)

两表以book表和class为例，book表代表商品，class代表类别。

![sql优化实例3分析1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3分析1.png)

第一次优化：

将book表的card字段加上索引，发现有了效果。

![sql优化实例3优化1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3优化1.png)

将class表的card字段加上索引，发现并没有效果。

![sql优化实例3优化2](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3优化2.png)

结论：

	由于左、右连接的特性决定，索引应该反着加（即左连接应给右表加索引，右连接应给左表加索引）。因为，拿左链接举例来说，左表是一定需要全部加载的，但对于右表却只需要加载一部分。所以这时候，确定右表需要加载哪些数据就很关键，所以索引需要加在右表。

![sql优化实例3结论1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例3结论1.png)

##### 三表：

![sql优化实例4phone表](https://github.com/g453030291/java-2/blob/master/images/sql优化实例4phone表.png)

分析sql性能：

![sql优化实例4分析1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例4分析1.png)

![sql优化实例4分析2](https://github.com/g453030291/java-2/blob/master/images/sql优化实例4分析2.png)

三表全部是all，都是全表扫描，有性能问题，需要加索引，优化查询效率。

第一次优化：

![sql优化实例4优化1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例4优化1.png)

![sql优化实例4结果1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例4结果1.png)

结论：

<u>sql的join查询，永远是小表驱动大表，来减少io。并且优先优化内层的嵌套查询。在被驱动的表字段加索引。并且，如果无法保证被驱动的表join条件被索引的情况下，就需要调整JoinBuffer的设置。</u>

![sql优化实例4结论1](https://github.com/g453030291/java-2/blob/master/images/sql优化实例4结论1.png)

2.索引失效（应该避免）：

示例表：

![索引失效示例表](https://github.com/g453030291/java-2/blob/master/images/索引失效示例表.png)

案例（索引失效）：

![索引失效案例详解](https://github.com/g453030291/java-2/blob/master/images/索引失效案例详解.png)

##### 2.1.全值匹配我最爱：

![索引失效示例表索引情况](https://github.com/g453030291/java-2/blob/master/images/索引失效示例表索引情况.png)

正常使用上面建立的复合索引，是没有问题的。

![索引失效示例表正常使用复合索引](https://github.com/g453030291/java-2/blob/master/images/索引失效示例表正常使用复合索引.png)

出现了索引失效的情况：

![索引失效示例表出现索引失效](https://github.com/g453030291/java-2/blob/master/images/索引失效示例表出现索引失效.png)

复合索引建立的是name，age，pos的顺序。如果where后直接跟上name为开头，符合索引就会生效。如果不是name开头作为where后的条件，那么就会发生索引失效的情况。

##### <u>2.2最佳左前缀法则</u>

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始，<u>并且不跳过索引中间的列。</u>

![最佳左前缀法则](https://github.com/g453030291/java-2/blob/master/images/最佳左前缀法则.png)

##### 2.3不要在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表所描。 

![不在索引列上做操作](https://github.com/g453030291/java-2/blob/master/images/不在索引列上做操作.png)

##### 2.4存储引擎不能使用索引中范围条件右边的列。 

![索引右边的列不能用](https://github.com/g453030291/java-2/blob/master/images/索引右边的列不能用.png)

##### 2.5尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *

![减少select*](https://github.com/g453030291/java-2/blob/master/images/减少select*.png)

![减少select*2](https://github.com/g453030291/java-2/blob/master/images/减少select*2.png)

##### 2.6mysql在使用不等于（!=或者<>）的时候无法使用索引会导致全表扫描

![避免使用不等于](https://github.com/g453030291/java-2/blob/master/images/避免使用不等于.png)

##### 2.7is null,is not null也无法使用索引

![isnotnull](https://github.com/g453030291/java-2/blob/master/images/isnotnull.png)

##### 2.8like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作

![百分like加右边](https://github.com/g453030291/java-2/blob/master/images/百分like加右边.png)

<u>问题：解决like‘%字符串%’时索引不被使用的方法？</u>

<u>答：使用覆盖索引。将需要模糊查询的字段全部加上索引，即可解决全模糊匹配索引失效问题。</u>

##### 2.9字符串不加单引号索引失效

字符串不加单引号，mysql会在底层做阴式的类型转换。自动帮你加上单引号。这样就会造成索引失效。

![字符串不加单引号](https://github.com/g453030291/java-2/blob/master/images/字符串不加单引号.png)

##### 2.10少用or，用它来连接时会索引失效

![少用or](https://github.com/g453030291/java-2/blob/master/images/少用or.png)

##### 2.11总结练习

![练习1](https://github.com/g453030291/java-2/blob/master/images/练习1.png)

3.一般性建议：