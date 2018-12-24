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



#### 4.性能分析

#### 5.索引优化