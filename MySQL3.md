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

![update流程](https://github.com/g453030291/java-2/blob/master/images/update流程.png)

执行一次update的处理流程：

1.执行器先找引擎取ID=2这一行。ID是主键，引擎直接用树搜索找到这一行。如果ID=2这一行数据本来就在内存中，就直接返回给执行器，否则，需要从磁盘读入内存，再返回。

2.执行器拿到引擎给的数据，把这个值加上1，比如原来是N，现在就是N+1，得到新的一行数据，再调用引擎接口写入这行新的数据。

3.引擎将这行新数据更新到内存中，同时将这个操作记录在redo log里，此时redo log处于prepare状态。然后告知执行器执行完成了，随时可以提交事务。

4.执行器生成这个操作的binlog，并把binlog写入磁盘。

5.执行器调用引擎提交事务的接口，引擎把刚刚写入的redo log改成提交（commit）状态，更新完成。

两阶段提交：

最后的三步将redo log的写入分成了两个步骤：prepare和commit，这就是两阶段提交。redo log和binlog都可用于表示事务的提交状态，这里必须使用两阶段提交是因为需要保证两种日志和磁盘状态必须保持一致。

关键参数：

```properties
innodb_flush_log_at_grx_commit=1
#表示每次事务的redo log都直接持久化到磁盘，这样可以保证MySQL异常重启后数据不丢失。
sync_binlog=1
#表示每次事务的binlog都持久化到磁盘，这样可以保证MySQL异常重启后binlog不会丢失。
```

#### 三、事务隔离：

隔离性与隔离级别：

SQL标准的事务隔离级别包括：

读未提交（read uncommitted）：一个事务还没提交时，它做的变更就能被别的事务看到。

读提交（read committed）：一个事务提交之后，它做的变更才会被其他事务看到。

可重复读（repeatable read）：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读的隔离级别下，未提交变更对其他事务也是不可见的。

串行化（serializable）：对同一条记录，“写”会加“写锁”，”读“回家“读锁”。当出现读写锁冲突时，后访问的事务必须等前一个事务执行完成，才能继续执行。

	在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。在“可重复读”隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。在“读提交”隔离级别下，这个视图是在每个SQL语句开始执行的时候创建的。这里需要注意的是，“读未提交”隔离级别下直接返回记录上的最新值，没有视图概念；而“串行化”隔离级别下直接用加锁的方式来避免并行访问。

查看当前数据库的隔离级别：

```sql
mysql> show variables like 'transaction_isolation';

+-----------------------+----------------+

| Variable_name | Value |

+-----------------------+----------------+

| transaction_isolation | READ-COMMITTED |

+-----------------------+----------------+
```

事务隔离的实现：

![事务隔离实现](https://github.com/g453030291/java-2/blob/master/images/事务隔离实现.png)

```sql
mysql> show variables like 'transaction_isolation';

+-----------------------+----------------+

| Variable_name | Value |

+-----------------------+----------------+

| transaction_isolation | READ-COMMITTED |

+-----------------------+----------------+
```

	在MySQL中，实际上每条记录在更新的时候都会同时记录一条会滚操作。记录上次的最新值，通过会滚操作，可以得到前一个状态的值。这叫做<u>回滚日志</u>。这些会滚日志，会一直保留，直到系统认为没有用的时候，才会自动删除。什么时候系统会认为这些日志没有用呢？也就是没有比当前日志时间更早的read-view的时候。
	
	这也就是为什么尽量不要使用长事务。长事务意味着系统里会存很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能会用到的会滚记录都必须保留，这就会导致大量占用存储空间。

事务的启动方式：

1.显式启动事务语句，begin或start transaction。配套的提交语句时commit，回滚语句时rollback。

2.set autocommit=0，这个命令会将当前线程的自动提交关掉。意味着如果你只执行一个select语句，这个事务就启动了，而且并不会自动提交。这个事务持续存在直到你主动执行commit或rollback语句，或断开连接。

	建议使用set autocommit=1，显式启动事务。对于需要频繁使用事务的业务，可以使用commit work and chain语法。在autocommit为1的情况下，用begin启动事务，如果执行commit则提交事务。如果执行commit work and chain，则是提交事务并自动启动下一个事务，这样也省去了再次执行begin语句的开销。同时带来的好处是从程序开发的角度明确的知道每个语句是否处于事务中。
	
	可以在information_schema库的innodb_trx这个表中查询长事务，比如下边这条语句，用于查询持续时间超过60s的事务。

```sql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

#### 四、索引（上）：

索引常见模型：

哈希表：

![哈希表](https://github.com/g453030291/java-2/blob/master/images/哈希表.png)

	哈希表是一种以键-值（key-value）存储数据的结构，我们只要输入待查找的值即key，就可以找到对应的值即value。哈希思路很简单，把值放在数组里，用一个哈希函数对key换算成一个确定的位置，然后把value放在数组的这个位置。如果对多个key哈希函数换算成同一个值，会把他们拉成一个链表，会先将key哈希找到链表，然后遍历找到对应值。

适用场景：哈希表这种数据结构适用于只有等值查询的场景，如memcached以及其他一些NoSQL引擎。

数组：

![数组](https://github.com/g453030291/java-2/blob/master/images/数组.png)

	有序数组在等值查询和范围查询场景中的性能都非常优秀。查询场景中，有序数组是最好的数据结构，但是，在需要更新数据的时候，你往中间插一条数据，就必须挪动后面所有数据，成本太高。

使用场景：有序数组索引只适用于静态存储引擎，如你要保存上一年度某城市所有人口信息，这种不太变更的数据。

二叉搜索树：

![二叉搜索树](https://github.com/g453030291/java-2/blob/master/images/二叉搜索树.png)	

	二叉搜索树的特点是：每个节点的左儿子小于父节点，父节点又小于右儿子。这样，如果你需要查询数据，搜索最大的时间复杂度是树的高度。这个查询的时间复杂度是O(log(N))。当然，为了维持O(log(N))的查询复杂度，你需要保持这颗树是平衡二叉树。为了做这个保证，更新的时间复杂度也是O(log(N))。
	
	当然，树可以有二叉，也可以有多叉。多叉树就是每个节点有多个儿子，儿子之间的大小从左到右递增。二叉树是搜索效率最高的，但实际上大多数的数据库却并不使用二叉树，原因是，索引不仅存在内存中，还要保存在磁盘上。为了让一个查询尽可能的少读磁盘，就必须让查询过程访问尽量少的数据块。那么我们不应该使用二叉树，应该使用N叉树，N取决于数据块的大小。

InnoDB的索引类型：

	InnoDB中，使用的是B+树的索引类型。其结构保存大概如图：

![B+树](https://github.com/g453030291/java-2/blob/master/images/B+树.png)

	如图所示，索引类型分为主键索引和非主键索引。主键索引的叶子结点存的是整行数据。在InnoBD里，主键索引也被称为聚簇索引（clustered index）。非主键索引的叶子结点内容是主键的值。在InnoDB，非主键索引也被称为二级索引（secondary index）。
	
	根据上面的索引结构说明，我们来讨论一个问题：基于主键索引和普通索引的查询有什么区别？
	
	如果语句是select * from T where ID=500，即主键查询方式，则只需要搜索ID这颗B+树。
	
	如果语句是select * from T where k=500，即普通索引查询方式，则需要先搜索k索引树，得到ID的值为500，再到ID索引树搜索一次。这个过程称为回表。

索引维护：

	B+树为了维护索引有序性，插入新值时需要做必要的维护。以上图，如果在中间插入数据，需要挪动后面所有数据，来空出位置。更遭的是，如果插入位置的数据页已满，根据B+树算法，需要申请一个新的数据页，然后挪动部分数据过去。这个过程称为叶分裂。这个时候，性能自然会有影响。除了性能以外，叶分裂操作还影响数据页的利用率。原本放在一个页的数据，现在分裂到两个页中，整体空间利用率降低50%。
	
	所以，使用时，尽量使用主键为自增整形。这样占用空间小，且索引叶子结点也小，普通索引占用空间也小。从性能和存储空间方面考虑，自增主键往往是更合理的选择。有一种情况是适合业务字段直接做主键。1.只有一个索引；2.该索引必须是唯一索引。这就是典型的KV场景。由于没有其他索引，所以也就不用考虑其他索引的叶子结点大小问题。直接将这个索引设置为主键，可以避免每次查询需要搜索两棵树。

#### 五、索引（下）：

对于表：

```sql
mysql> create table T (
ID int primary key,
k int NOT NULL DEFAULT 0, 
s varchar(16) NOT NULL DEFAULT '',
index k(k))
engine=InnoDB;

insert into T values(100,1, 'aa'),(200,2,'bb'),(300,3,'cc'),(500,5,'ee'),(600,6,'ff'),(700,7,'gg');
```

查询`select * from T where k between 3 and 5`需要执行几次树的搜索？会扫描多少行？

![索引下1](https://github.com/g453030291/java-2/blob/master/images/索引下1.png)

语句执行流程：

1.在k索引树上找到k=3的记录，取到id=300

2.再到id索引树上查到id=300，对应的R3

3.在k索引树取下一个值k=5，取得k=500

4.再回到id索引树到id=500，对应的R4

5.在k索引树取下一个值k=6，不满足条件，循环结束

​	在上面的过程中，回到主键索引树搜索的过程，称为回表。上面的查询，读了k索引树3条记录，回表了两次。效率比较低。

覆盖索引：

​	如果执行上面的查询，这里只需要查询ID的值，而ID的值已经在k索引树上了，因此可以直接提供查询结果，不需要回表。也就是，在这个查询里，索引k已经“覆盖了”我们查询的需求，我们称为覆盖索引。由于覆盖索引可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段。注意：在引擎内部使用覆盖索引在索引k上其实读取了三个记录，R3～R5（对应索引k上的记录项），但是对于MySQL的Server层来说，它就是找引擎拿到了两条记录，因此MySQL认为扫描的行数是2。

最左前缀原则：

​	B+树这种索引结构，可以利用索引的”最左前缀“，来定位记录。

索引下推：





​	











