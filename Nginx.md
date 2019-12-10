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

#### realip模块：

功能：修改客户端地址

编译进nginx：`--with-http_realip_module`

指令：

````shell
set_real_ip_from
real_ip_header
real_ip_recursive
````

变量：

````shell
realip_remote_addr
realip_remote_port
````

指令用法：

![nginx-realip指令](https://github.com/g453030291/java-2/blob/master/images/nginx-realip指令.png)

#### rewrite模块:

return指令：

![nginx-return](https://github.com/g453030291/java-2/blob/master/images/nginx-return.png)

rewrite指令：

![nginx-rewrite指令](https://github.com/g453030291/java-2/blob/master/images/nginx-rewrite指令.png)

rewrite模块if指令：

可检查变量值是否为空或者0，与字符串做匹配，与正则表达式做匹配，检查文件、目录是否存在，是否为可执行文件等。

Context：server，location

location指令：

Context：server，location

限制每个客户端的并发连接数：

#### ngx_http_limit_conn_module模块：

基于共享内存，全部worker进程都会生效，限制的有效性基于key的设计，依赖postread阶段的realip模块取到的真实ip

limit_conn指令：定义共享内存大小，设计key关键字（给开辟的共享内存取名），限制并发连接数。

限制发生时向客户端返回错误码。

限制每个客户端每秒处理请求数：

#### ngx_http_limit_req_module模块：

limit_req指令，限制并发连接数。

限制某些IP地址的访问权限：access模块

#### auth_basic模块：

提供用户名密码访问权限

#### auth_request模块:

统一的用户权限验证系统：auth_request模块。默认未编译进nginx，`--with-http_auth_request_modult`，使用以上指令编译。原理为，收到请求后，生成一个一模一样的子请求，访问上游服务器。如果返回2xx，就继续访问，如果返回错误，就返回给浏览器。

#### 限制所有access阶段模块的satisfy指令：

#### precontent阶段的try_files指令：

nix_http_try_files_module模块。可以判断文件是否存在，不存在执行何种操作。

#### precontent阶段的mirror模块：

实时拷贝流量：precontent阶段的mirror模块。nix_http_mirror_module模块。处理请求时，生成子请求访问其他服务，对子请求的返回值不做处理。

content阶段：root和alias指令：root会将完整的url映射到文件路径，alias只会将location后的url映射到文件路径。

重定向跳转域名：

````shell
server_name_int_redirect on|off;
Context:http,server,location
port_in_redirect on|off;
Context:http,server,location
absolute_redirect on|off;
Context:http,server,location
````

#### 对content阶段的index模块：

ngx_http_index_module

Context：http，server，location

#### content阶段的autoindex模块：

选择autoindex模块，以/结尾的返回root/alias中目录的目录结构，作为/路径返回的文件内容（可选择返回json，html）。

指令：autoindex，autoindex_exact_size，autoindex_format，autoindex_localtime

#### content阶段的concat模块:

提升性能：content阶段的concat模块。当页面需要访问多个小文件时，把他们的内容合并到一次http相应中返回，提升性能。

ngx_http_concat_module：`add-module=../nginx-http-concat/`

#### log阶段log模块：

记录请求访问日志ngx_http_log_module,无法禁用

access日志格式：log_format

配置日志文件路径：access_log

对日志文件名包含变量时的优化：open_log_file_cache

#### 替换响应中的字符串：sub模块

将响应中的字符串替换成新的字符串。ngx_http_sub_filter_module模块，默认未编译进nginx,通过`--with-http_sub_module`启用

sub模块的指令：sub_filter,sub_filter_last_modified,sub_filter_once,sub_filter_types

#### 在响应前后添加内容：addition模块

不是修改响应本身，而是增加新的url访问子请求，添加内容到响应前后。ngx_http_addition_filter_module模块默认未编译进nginx。通过`--with-http_addition_module`启用。add_before_body，add_after_body，addition_types

#### Nginx中的变量：

![nginx-变量惰性求值](https://github.com/g453030291/java-2/blob/master/images/nginx-变量惰性求值.png)

nginx变量的特性：1.惰性求值：变量在有代码读取前，并不会进行任何编译、取值、存储等操作，这样设计可以增加性能。因为走的代码行数变少了。2.惰性求值带来的问题是，一次http请求中，变量的值会多次发生改变，如何取出对应阶段想要的值。nginx使用了hash表来存储各个阶段的值。供其他模块调用。

#### Http框架提供的变量：

http请求相关变量：

| 变量名                                    | 返回值含义                                                   |
| ----------------------------------------- | ------------------------------------------------------------ |
| arg_参数名                                | url中某个具体参数的值                                        |
| query_string                              | 与args变量完全相同                                           |
| args                                      | 全部url参数                                                  |
| is_args                                   | 如果请求url中有参数则返回？否则返回空                        |
| content_length                            | http请求中标识包体长度的Content-Length头部的值               |
| content_type                              | 标识请求包体类型的Content-Type头部的值                       |
| uri                                       | 请求的uri（不同于url，不包括？后的参数）                     |
| document_uri                              | 与uri完全相同                                                |
| request_uri                               | 请求的url（包括uri以及完整的参数）                           |
| scheme                                    | 协议名，例如http或者https                                    |
| request_method                            | 请求方法，例如get或者post                                    |
| request_length                            | 所有请求内容的大小，包括请求行、头部、包体等                 |
| remote_user                               | 由HTTP Basic Authentication协议传入的用户名                  |
| request_body_file                         | 临时存放请求包体的文件:1.如果包体非常小则不会存文件。2.client_body_in_file_only强制所有包体存入文件，且可决定是否删除 |
| request_body                              | 请求中的包体，这个变量当且仅当使用反向代理，且设定用内存暂存包体时才有效 |
| request                                   | 原始的url请求，含有方法与协议版本，例如GET /?a=1&b=22 HTTP/1.1 |
| Host                                      | 1.先从请求行中获取。2.如果含有Host头部，则用其值替换掉请求行中的主机名。3.如果前两者都取不到，则使用匹配上的server_name |
| http_头部名字（返回一个具体请求头部的值） | 1.特殊值处理：http_host;http_user_agent;http_referer;http_via;http_x_forwarded_for;http_cookie。2.通用值就使用http_头部名字，来取出 |

Tcp连接的相关变量：

| 变量名              | 返回值含义                                                   |
| ------------------- | ------------------------------------------------------------ |
| binary_remote_addr  | 客户端地址的整型格式，对于IPv4是4字节，对于IPv6是16字节      |
| connection          | 递增的连接序号                                               |
| connection_requests | 当前连接上执行过的请求数，对keepalive连接有意义              |
| remote_addr         | 客户端地址                                                   |
| remote_port         | 客户端端口                                                   |
| proxy_protocol_addr | 若使用了proxy_protocol协议则返回协议中的地址，否则返回空     |
| proxy_protocol_port | 若使用了proxy_protocol协议则返回协议中的端口，否则返回空     |
| server_addr         | 服务器端地址                                                 |
| server_port         | 服务器端端口                                                 |
| TCP_INFO            | tcp内核层参数，包括`$tcpinfo_rtt;$Tcpinfo_rttvar;$tcpinfo_snd_cwnd;$tcpinfo_rcv_space` |
| server_protocol     | 服务器端协议，例如HTTP/1.1                                   |

Nginx处理请求过程中产生的变量：

| 变量名             | 返回值含义                                                   |
| ------------------ | ------------------------------------------------------------ |
| request_time       | 请求处理到现在的耗时，单位为秒，精确到毫秒                   |
| server_name        | 匹配上请求的server_name值                                    |
| https              | 如果开启了TLS/SSL，则返回on，否则返回空                      |
| request_completion | 若请求处理完则返回OK，否则返回空                             |
| request_id         | 以16进制输出的请求标识id，该id共含有16字节，是随机生成的     |
| request_filename   | 待访问文件的完整路径                                         |
| document_root      | 由URI和root/alias规则生成的文件夹路径                        |
| realpath_root      | 将document_root中的软连接等换成真实路径                      |
| limit_rate         | 返回客户端响应时的速度上线，单位为每秒字节数。可以通过set指令修改对请求产生效果 |

发送HTTP响应时相关的变量：

| 变量名             | 返回值含义                                                   |
| ------------------ | ------------------------------------------------------------ |
| body_bytes_sent    | 响应中body包体的长度                                         |
| bytes_sent         | 全部http响应的长度                                           |
| status             | http响应中的返回码                                           |
| sent_trailer_名字  | 把响应结尾内容里值返回                                       |
| 特殊处理           | Sent_http_content_type;sent_http_content_length;sent_http_location_sent_http_location;sent_http_last_modified;sent_http_connection;sent_http_keep_alive;sent_http_transfer_encoding;sent_http_cache_control;sent_http_link; |
| sent_http_头部名字 | 响应中某个具体头部的值                                       |

Nginx系统变量：

| 变量名        | 返回值含义                                             |
| ------------- | ------------------------------------------------------ |
| Time_local    | 以本地时间标准输出的当前时间                           |
| Time_iso8601  | 使用ISO 8601标准输出的当前时间                         |
| Nginx_version | Nginx版本号                                            |
| pid           | 所属worker进程的进程id                                 |
| pipe          | 使用了管道则返回p，与hostname命令输出一致              |
| hostname      | 所在服务器主机名，与hostname命令输出一致               |
| msec          | 1970年1月1日到现在的时间，单位为秒，小数点后精确到毫秒 |

### 防盗链：

#### referer模块：

通过referer模块，用invalid_referer变量根据配置判断referer头部是否合法。目的，拒绝非正常的网站访问我们站点的资源。默认编译进nginx，通过`--without-http_referer_module`禁用。

指令：valid_referers,referer_hash_bucker_size,referer_hash_max_size

![nginx-referers模块](https://github.com/g453030291/java-2/blob/master/images/nginx-referers模块.png)

#### 另一种处理防盗链的方式：secure_like模块

第一种：是较为复杂的，可以对用户ip，时间戳，访问uri等等做限制，识别等

![nginx-secure_link模块](https://github.com/g453030291/java-2/blob/master/images/nginx-secure_link模块.png)

第二种：比较简单，但只能做简单md5匹配

![nginx-secure_link模块2](https://github.com/g453030291/java-2/blob/master/images/nginx-secure_link模块2.png)

#### map模块：

通过映射新变量提供更多的可能性。

#### split_clients模块：

实现AB测试

![nginx-split_clients模块](https://github.com/g453030291/java-2/blob/master/images/nginx-split_clients模块.png)

#### geo模块：

需要使用IP地址，或者子网掩码创建新变量。

![nginx-geo模块](https://github.com/g453030291/java-2/blob/master/images/nginx-geo模块.png)

#### geoip模块：

基于MaxMind数据库从客户端地址获取变量(获取用户地理位置)：

![nginx-geoip模块](https://github.com/g453030291/java-2/blob/master/images/nginx-geoip模块.png)

geoip_country指令：geoid_country;geoip_proxy

#### 对客户端keepalive行为控制的指令：

指令集：keepalive_disable;keepalive_requests;keepalive_timeout;

# 四、反向代理与负载均衡

负载均衡：

![nginx-负载均衡](https://github.com/g453030291/java-2/blob/master/images/nginx-负载均衡.png)

#### 与上游服务相关的stream模块：

指定上游服务地址的upstream与server。

````shell
http {
	upstream name {
			server address [parameters(backup,down)];
	}
}
````

![nginx-upstream模块](https://github.com/g453030291/java-2/blob/master/images/nginx-upstream模块.png)

#### 加权Round-Robin负载均衡算法：

![nginx-round-robin算法](https://github.com/g453030291/java-2/blob/master/images/nginx-round-robin算法.png)

#### 对上游服务使用keepalive长连接：

![nginx-upstream-keepalive指令](https://github.com/g453030291/java-2/blob/master/images/nginx-upstream-keepalive指令.png)

指令集：keepalive;keepalive_requests;keepalive_timeout

#### 指定上游服务域名解析的resolver指令：

#### 基于客户端IP地址的Hash算法实现负载均衡,upstream_ip_hash模块：

以客户端的IP地址作为hash算法的关键字,映射到特定的上游服务器中。可以基于realip模块修改用于执行算法的IP地址。模块：`ngx_http_upstream_ip_hash_module`，指令：`ip_hash`。

#### 基于任意关键字实现Hash算法的负载均衡,upstream_hash模块：

通过指定关键字作为hash key，基于hash算法映射到特定的上游服务器中。模块：`ngx_http_upstream_hash_module`,指令：`hash key [consistent]`。

问题：宕机或者扩容时，hash算法引发大量路由变更，可能导致缓存大范围失效。可以使用一致性hash算法。

![nginx-一致性hash算法](https://github.com/g453030291/java-2/blob/master/images/nginx-一致性hash算法.png)

#### upstream_hash模块，使用一致性hash算法：

指令：`hash key [consistent]`

#### 优先选择连接最少的上游服务器：upstream_least_conn模块：

从所有上游服务器中，找出当前并发连接数最少的一个，将请求转发到他。模块：`ngx_http_upstream_least_conn_module`，指令：`least_conn`

问题：这样只能对一个work进程生效，无法对所有进程共享。

#### 使用共享内存使负载均衡策略对所有worker进程生效，upstream_zone模块：

分配出共享内存，将其他upstream模块定义的负载均衡策略数据、运行时每个上游服务的状态数据存放在共享内存上，以对所有nginx worker进程生效。模块:`ngx_http_upstream_zone_module`，指令：`zone name [size]`。

upstream模块提供的变量（不含cache）:

| 变量                     | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| upstream_addr            | 上游服务器的IP地址，格式为可读字符串，例如127.0.0.1:8012     |
| upstream_connect_time    | 与上游服务建立连接消耗的时间，单位为秒，精确到毫秒           |
| upstream_header_time     | 接收上游服务发回响应中http头部所消耗的时间，单位为秒，精确到毫秒 |
| upstream_response_time   | 接收完整的上游服务响应所消耗的时间，单位为秒，精确到毫秒     |
| upstream_http_名称       | 从上游服务返回的响应头部的值                                 |
| upstream_bytes_received  | 从上游服务接收到的响应长度，单位为字节                       |
| upstream_response_length | 从上游服务返回的响应包体长度，单位为字节                     |
| upstream_status          | 上游服务返回的HTTP响应中的状态码。如果未连接上，该变量值为502 |
| upstream_cookie_名称     | 从上游服务发回的响应头Set-Cookie中取出cookie值               |
| upstream_trailer_名称    | 从上游服务的响应尾部取到的值                                 |

proxy处理请求的流程：

![nginx-http反向代理流程](https://github.com/g453030291/java-2/blob/master/images/nginx-http反向代理流程.png)

#### proxy模块：

功能：对上游服务使用http/https协议进行反向代理。模块：`ngx_http_proxy_module`。指令：`proxy_pass URL`。

url参数规则：

![nginx-proxy-url参数规则](https://github.com/g453030291/java-2/blob/master/images/nginx-proxy-url参数规则.png)

生成发往上游的请求行：指令：`proxy_method (method)`、`proxy_http_version (1.0|1.1)`

生成发往上游的请求头部：指令：`proxy_set_header (field value)`,(若value为空字符串，则整个header都不会向上游发送)，`proxy_pass_request_headers (on|off)`，

生成发往上游的包体：指令：`proxy_pass_request_body (on|off)`、`proxy_set_body (value)`

接收客户顿请求的包体：指令:`proxy_request_buffering (on|off)`,on:客户端网速较慢，上游服务并发处理能力低，适应高吞吐量场景。off：更及时的响应，降低nginx读写磁盘的消耗，一旦开始发送内容proxy_next_upstream功能失效。

![nginx-proxy客户端包体接收](https://github.com/g453030291/java-2/blob/master/images/nginx-proxy客户端包体接收.png)

最大包体长度限制：`client_max_body_size 1m`。仅对请求头部中含有Content-Length有效超出最大长度后，返回413错误。

临时文件路径格式：`client_body_temp_path (path)`、`client_body_in_file_only (on|clean|off)`

读取包体超时：`client_body_timeout (60s)`

nginx向上游服务建立连接：

`proxy_connect_timeout (time)`:超时后回向客户端生成http响应502

`proxy_next_upstream http_502|...`

`proxy_socket_keepalive (on|off)`：上游连接启用tcp keepalive

`keepalive connections;`

`keepalive_requests (100)`

`proxy_vind address[transparent]|off`：修改tcp连接中的local address

`proxy_ignore_client_abort (on|off)`:当客户端关闭连接时

`proxy_send_timeout 60s`:向上游发送http请求

nginx接收上游http响应头部：

`proxy_buffer_size 4k`:接收上游的http响应头部

nginx接收上游http响应包体：

`proxy_buffers 8 4k`

`proxy_buffering (on|off)`

`proxy_max_temp_file_size 1024m`

`proxy_temp_file_write_size 8k`

`proxy_temp_path path[level1[level2[level3]]]`

`proxy_busy_buffers_size 8k`:及时转发包体

`proxy_read_timeout 60s`:接收上游时网络速度相关指令

`proxy_limit_rate 0`:接收上游时网络速度相关指令

`proxy_store_access users:rw;`：上游包体的持久化

`proxy_store on|off|string;`：上游包体的持久化

处理上游的响应头：

禁用上游响应头部功能：`proxy_ignore_headers (field...)`

功能：某些响应头部可以改变nginx的行为。

转发上游的响应：`proxy_hide_header (field)`

修改返回的Set-Cookie头部：`proxy_cookie_domain off`、`proxy_cookie_domain (domain replacement)`、`proxy_cookie_path off`、`proxy_cookie_path (path replacement)`

修改返回的Location头部：`proxy_redirect (default|off|redirect replacement)`

上游返回失败时的处理方法：

模块：`proxy_next_upstream (err|timeout|......)`

限制`proxy_next_upstream_timeout (0)`的时间与次数

`proxy_next_upstream_tries (0)`重试次数

用error_page拦截上游失败响应：

当上游响应的响应码大于等于300时，应将响应返回客户端还是按error_page指令处理:`proxy_intercept_errors (on|off)`

nginx上下游双向使用ssl：

![nginx-双向认证时指令示例](https://github.com/g453030291/java-2/blob/master/images/nginx-双向认证时指令示例.png)

浏览器和nginx正确使用缓存：

![nginx-浏览器请求缓存过程](https://github.com/g453030291/java-2/blob/master/images/nginx-浏览器请求缓存过程.png)

nginx作为静态资源服务器，如何与浏览器沟通缓存的过期、存储、继续使用、重新请求？

Etag头部：比较etags能快速确定资源是否变化，也可能被跟踪服务器永久存留。

etag指令：`etag (on|off)`

If-None-Match：是一个条件式请求首部。根据给定资源的日期之后进行过修改才会将资源返回。

not_modified过滤模块：客户端拥有缓存，但不确认缓存是否过期。

expires指令：`expires [modified] time`可以设置具体的时间单位

not_modified过滤模块：`if_modified_sice off|exact|before`

nginx缓存：定义存放缓存的载体：`proxy_cache zone|off`、`proxy_cache_path path`、`proxy_cache_key (string)`、`proxy_cache_valid [code...] time`、`proxy_no_cache string ...`、`proxy_cache_bypass string...`、`proxy_cache_convert_head (on|off)`、`upstream_cache_status`、`proxy_cache_methods GET HEAD`

http头部：`X-Accel-Expires`、`Vary:*`、`Set-Cookie`

nginx如何解决缓存穿透问题？

`proxy_cache_lock on|off`、`proxy_cache_lock_timeout time;`、`proxy_cache_lock_age time`、`proxy_cache_use_stale updating`、`proxy_cache_background_update on|off`、`proxy_cache_background_update on|off`、`proxy_cache_revalidate on|off`

及时清除缓存：

商业版有相关功能，开源版可以使用第三方模块：`ngx_cache_purge`，使用`--add_module=`指令添加模块到nginx中，功能：接收到指定http请求后立刻清除缓存。

七层反向代理对照：uwsgi反向代理、fastcgi反向代理、scgi反向代理、http反向代理

memcached反向代理。

websocket反向代理：

由ngx_http_proxy_module模块实现。

````shell
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade; 
proxy_set_header Connection "upgrade";
````

上游服务大文件传输，使用slice模块，将一个大文件分解为多个小文件，更好地用缓存为客户端的range协议服务。slice模块对客户端使用断点续传、多线程下载有更好的支持。

模块:`http_slice_module`，通过`--with-http_slice_module启用`

指令：`slice size`

Open_file_cache指令：

`open_file_cache off;`、`open_file_cache_errors on|off`、`open_file_cache_min_uses number`、`open_file_cache_valid time`

#### HTTP2.0

主要特性：传输量大幅减少（以二进制方式传输，表头压缩）、多路复用及相关功能（消息优先级）、服务器消息推送（并行推送）。

![http2.0核心概念](https://github.com/g453030291/java-2/blob/master/images/http2.0核心概念.png)

模块：`ngx_http_v2_module`,通过`--with-http_v2_module`编译nginx加入http2协议的支持。

前提：开启TLS/SSL协议

使用方法：`listen 443 ssl http2`

指令：`http2_push_preload on|off`,`http2_push uri|off`,`http2_max_concurrent_pushes number`,`http2_recv_timeout time`,`http2_idle_timeout time`,`http2_max_concurrent_pushes number`,`http2_max_concurrent_streams number`,`http2_max_field_size size`

#### gRPC反向代理

模块：`ngx_http_grpc_module`,依赖`ngx_http_v2_module`模块

#### TCP反向代理和UDP反向代理：

# 五、Nginx的系统层性能优化

#### 如何高效使用CPU？

设置work进程数量：`worker_processes number`

如何查看nginx上下午切换次数？

`vmstat 1`、`dstat 1`、查找nginxpid:`ps -ef | grep nginx`、查看进程间切换：`pidstat -w -p pid 1`

什么决定CPU时间片大小？

Nice静态优先级：-20 -- 19

Priority动态优先级：0 -- 139

设置work静态优先级：`worker_proiority number`设置上下文：main。一般将nginx的work进程设置为19，也就是最不友好。不主动给其它进程切换CPU。



# 六、从源码视角深入使用Nginx与OpenResty

