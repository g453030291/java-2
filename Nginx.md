# 一、初识Nginx

Nginx三个主要应用场景：静态资源服务（通过本地文件系统提供服务）、反向代理服务（Nginx的强大性能、缓存、负载均衡）、API服务（OpenResty）

Nginx的主要组成：Nginx二进制可执行文件、nginx.conf配置文件、access.log访问日志、error.log错误日志

Nginx的版本：1.15单数是主干版本标记为Mainline，1.14双数为稳定版本标记为Stable。

开源版Nginx：nginx.org（web网站适用）

商业版Nginx：nginx.com

开源版OpenResty：openresty.org（api服务器和web防火墙）

商业版OpenResty：openresty.com（章亦春）

编译Nginx：

````shell
#1.复制官网链接,下载nginx
#2.解压缩下载的nginx压缩包
#3.查看config脚本支持的参数列表
./configure --help | more
#4.这里重点看两个地方
	#(1)确定需要编辑的文件路径,以PATH结尾的配置是注明各文件所在的默认路径.如果不变动,只需要指定--prefix即可
	#(2)确定需要编译的文件,nginx默认编译的模块以without开头,nginx默认不编译的模块以with开头.
	#(3)查看编译时需要添加的特定参数,如--with-cc=PATH.这里是配置使用gcc编译时需要加的特定参数.
#5.指定nginx安装目录在/home/geek/nginx路径下
./configure --prefix=/home/geek/nginx
#6.编译成功后,生成objs文件夹, 其中有一个ngx_modules.c决定了编译时有哪些模块被安装
#7.编译
make
#8.安装
make install
````

安装nginx提供的vim插件：`cp -r contrib/vim/* ~/.vim/`

Nginx配置语法：

1.配置文件由指令与指令块构成

2.每条指令以；分号结尾，指令与参数间以空格符号分割

3.指令块以{}大括号将多条指令组织在一起

4.include语句允许组合多个配置文件以提升可维护性

5.使用#富豪添加注释，提高可读性

6.使用$符号使用变量

7.部分指令的参数支持正则表达式

Nginx命令行格式：

Nginx -s reload

-? -h：帮助

-c：使用指定配置文件

-g：使用配置指令

-p：指定运行目录

-s：发送信号（stop：立刻停止服务；quit：优雅的停止服务；reload：重载配置文件；reopen：重新开始记录配置文件）

-t -T：测试配置文件语法错误

-v -V：打印版本信息

热部署（更新nginx、其实只更新二进制文件）：

````shell
#1.查看进程
ps -ef | grep nginx
#2.备份老的文件
cd /sbin;
cp nginx nginx.old
#3.拷贝新编译好的nginx文件,到旧的目录中,替换掉目前进程使用的旧的文件
cp nginx /usr/local/nginx/sbin/
#4.向nginx老的master进程发送一个信号,告诉master进程,现在要进行热部署
kill USR2 13195
#5.现在已经生成了新的master进程,目前新旧master进程都存在,等待切换
#6.告诉老的master进程,请关闭你的所有work进程
kill -WINCH 13195
#7.此时,老的work进程已经全部退出,新的master和work已经全部开始工作
#8.但,老的master进程依然存在,这是为了如果切换失败,可以向老的master发送reload拉起老的work进程,继续工作
````

日志切割：

````shell
#1.将老的日志文件备份
mv access.log bak.log
#2.使用reopen生成新的配置文件
../sbin/nginx -s reopen

##########################
#1.实际工作中,会定时将日志切割
#2.查看目前的crontab中的定时任务
crontab -l
#3.查看脚本文件
vim rotate.sh
#4.脚本中向nginx发送了 -USR1信号,和-s reopen是一样的
````

![nginx-rotate](https://github.com/g453030291/java-2/blob/master/images/nginx-rotate.png)

配置Nginx静态资源服务器示例：

nginx.conf文件

````shel
location / {
		alias dlib/;#配置/路径与dlib路径一一对应,这里没有使用route
		autoindex on;#配置显示资源目录结构
		set $limit_rate 1k;#limit_rate是nginx内置变量,这里限制了向浏览器相应的速度为1k
}
````

配置gzip压缩：

````shell
gzip on;
gzip_min_length 1;#压缩大小:小于1k文件不压缩
gzip_comp_level 2;#压缩级别:已经做过压缩的文件不压缩
gzip_types text/plain application/x-javascript text/css application/xml text/javascript image/jpeg image/gif mage/png
````

配置记录日志格式：

````shell
http{
		log_formate	main '$remote_addr - $remote_user [$time_local] "$request" '
										 '$status $body_bytes_sent "$http_referer" '
										 '"$http_usr_agent" "$http_x_forwarded_for"';
		server{
				access_log	log/geek.access.log	main;
		}
}
````



# 二、Nginx架构基础



# 三、详解HTTP模块



# 四、反向代理与负载均衡



# 五、Nginx的系统层性能优化



# 六、从源码视角深入使用Nginx与OpenResty

