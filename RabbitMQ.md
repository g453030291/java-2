RabbitMQ高可用负载均衡集群：

![RabbitMQ高可用负载均衡集群](https://github.com/g453030291/java-2/blob/master/images/RabbitMQ高可用负载均衡集群.png)

AMQP协议：

![AMQP协议](https://github.com/g453030291/java-2/blob/master/images/AMQP协议.png)

Server：又称Broker，接受客户端的连接，实现AMQP实体服务

Connection：连接，应用程序与Broker的网络连接

Channel：网络信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道。客户端可建立多个Channel，每个Channel代表一个会话任务

Message：消息，服务器和应用程序之间传送的数据，由Properties和Body组成。Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body则就是消息体内容

Virtual host：虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个Virtual Host里面可以有若干个Exchange和Queue，同一个Virtual Host里面不能有相同名称的Exchange或Queue

Exchange：交换机，接受消息，根据路由键转发消息到绑定的队列

Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key

Routing key：一个路由规则，虚拟机可用它来确定如何路由一个特定消息

Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者

RabbitMQ架构：

![rabbitmq架构](https://github.com/g453030291/java-2/blob/master/images/rabbitmq架构.png)

rabbitmq消息流转：

![rabbitmq消息流转](https://github.com/g453030291/java-2/blob/master/images/rabbitmq消息流转.png)

常用命令：

服务启动：

```shell
rabbitmq-server start &
```

服务停止：

```shell
rabbitmqctl stop_app
```

管理插件：

```shell
rabbitmq-plugins enable rabbitmq_management
```

注意：

启动rabbitmq之前，需要修改配置文件`vim  /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app`，修改启动loopback_users，后面为一个json数组，需要去掉无关的符号。

安装完图形界面插件，默认端口号为15672。访问地址：http://192.168.0.108:15672。默认用户名、密码为：guest

