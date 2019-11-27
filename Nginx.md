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

配置反向代理：(官网http_proxy_model模块中都可以找到以下配置)

````shell
http{
	upstream local{
		server  127.0.0.1:8080;
	}
	server{
		server_name 域名;
		listen 80;
		
		location / {
			proxy_pass http://localhost;
			
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		}
	}
}
````

配置缓存：

````shell
http{
  #配置缓存,地址,名称,大小,开辟空间,等等
	proxy_cache_path	/temp/nginxcache	levels=1:2	keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
	upstream local{
		server  127.0.0.1:8080;
	}
	server{
		server_name 域名;
		listen 80;
		
		location / {
        proxy_pass http://localhost;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #配置缓存
        proxy_cache my_cache;
        proxy_cache_key $host$uri$is_args$args;
        proxy_cache_valid	200 304 302 1d;
		}
	}
}
````

用GoAccess图形化显示,分析nginx：

`goaccess geek.access.log -o ../html/report.html -real-time-html --time-format='%H:%M:%S' -date-format='%d/%b/%Y' --log-format=COMBINED`

指定accesslog的位置，指定输出html文件位置，指定实时更新，配置时间日期格式，指定日志格式

````shell
http{
	server{
		location /report.html{
			alias /usr/local/openresty/nginx/html/report.html;
		}
		location /{
		
		}
	}
}
````

OpenResty简单使用：

````shell
#1.下载、解压
#2.查看帮助文件
./configure --help | more
#3.编译
./configure
#4.安装
install
````

添加lua脚本L:

````shell
http{
	server{
		location /{
			content_by_lua '此处添加由openresty提供的lua脚本,处理http请求';
		}
	}
}
````

# 二、Nginx架构基础

![nginx进程结构](https://github.com/g453030291/java-2/blob/master/images/nginx进程结构.png)

work进程、master进程、cache manager进程、cache loader进程

![nginx信号](https://github.com/g453030291/java-2/blob/master/images/nginx信号.png)

使用nginx -s reload、stop......等等就是向nginx各进程发送linux进程间信号。（这是符合linux进程管理规则的）

![nginx-reload流程](https://github.com/g453030291/java-2/blob/master/images/nginx-reload流程.png)

work-shutdown-timeout：新版本nginx中，可以添加对旧的work子进程最大工作时间。也就是添加定时器，关闭work子进程。

![nginx热升级流程](https://github.com/g453030291/java-2/blob/master/images/nginx热升级流程.png)

work进程优雅的关闭：

1.设置定时器：worker_shutdown_timeout

2.关闭监听句柄

3.关闭空闲链接

4.循环中等待全部连接关闭

5.推出进程

nginx事件驱动模型：

![nginx异步非阻塞事件驱动模型epoll](https://github.com/g453030291/java-2/blob/master/images/nginx异步非阻塞事件驱动模型epoll.png)

nginx连接池：

nginx默认连接池大小为512.也就是512大小的一个connection数组。一个连接，大概会占用232+96*2个字节。如果修改连接池配置，要保证服务器有足够的内存空间处理这些连接。

nginx内存池：

nginx内存会提前申请一块较大内存。给nginx使用。需要分配小内存只和nginx申请，较大和系统申请。这样nginx对内存的优化非常好。减少内存碎片。

# 三、详解HTTP模块

Listen指令：

![nginx-listen指令](https://github.com/g453030291/java-2/blob/master/images/nginx-listen指令.png)

nginx接收请求事件模块：

![nginx-接收请求事件模块](https://github.com/g453030291/java-2/blob/master/images/nginx-接收请求事件模块.png)

nginx接收请求http模块：

![nginx-接收请求http模块](https://github.com/g453030291/java-2/blob/master/images/nginx-接收请求http模块.png)

正则表达式：

可以使用pcretest来测试自己写的nginx正则表达式。

server_name指令：

1.指令后可以跟多个域名，第一个是主域名

````shell
Syntax server_name_in_redirect on|off
Default server_name_in_redirect off;
Context http,server,location;
````

2.*泛域名：仅支持在最前或者最后

例如：server_name *.xxx.con

3.正则表达式：加～前缀

例如：server_name www.xxx.com ~^www\d+\.xxx$;

4.用正则表达式创建变量：用小括号()

例如：

````shell
server{
	server_name ~^(www\.)?(.+)$;
	location / {root /sites/$2;}#$2表示取出上边server_name后边的变量
}
````

````shell
server{
	server_name ~^(www\.)?(?<domain>.+)$;
	location / {root /sites/$domain;}#domain也表示对上边的变量取值
}
````

5.其它：

.xxx.com可以匹配xxx.com *.xxx.com

_匹配所有

"" 匹配没有传递Host头部



# 四、反向代理与负载均衡



# 五、Nginx的系统层性能优化



# 六、从源码视角深入使用Nginx与OpenResty

