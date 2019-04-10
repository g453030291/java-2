#### RabbitMQ基础概念及常用命令：  

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

关闭应用：`rabbitmqctl stop_app`

启动应用：`rabbitmqctl start_app`

节点状态：`rabbitmqctl status`

添加用户：`rabbitmqctl add_user username password`

列出所有用户：`rabbitmqctl list_users`

删除用户：`rabbitmqctl delete_user username`

清除用户权限：`rabbitmqctl clear_permissions -p vhostpath username`

列出用户权限：`rabbitmqctl list_user_permissions username`

修改密码：`rabbitmqctl change_password username newpassword`

设置用户权限：`rabbitmqctl set_permissions -p vhostpath username ".*"".*"".*"`

创建虚拟主机：`rabbitmqctl add_vhost chostpath`

列出所有虚拟主机：`rabbitmqctl list_vhosts`

列出虚拟主机上所有权限：`rabbitmqctl list_permissions -p vhostpath`

删除虚拟主机：`rabbitmqctl delete_vhost vhostpath`

查看所有队列信息：`rabbitmqctl list_queues`

清除队列里的消息：`rabbitmqctl -p vhostpath purge_queue blue`

移除所有数据，要在rabbitmqctl stop_app之后使用：`rabbitmqctl reset`

组成集群命令：`rabbitmqctl join_cluster <clusternode> [--ram]`

查看集群状态：`rabbitmqctl cluster_status`

修改集群节点的存储形式：`rabbitmqctl change_cluster_node_type disc|ram`

忘记节点：`rabbitmqctl forget_cluster_node [--offline]`

修改节点名称：`rabbitmqctl rename_cluster_node oldname1 newnode1 [oldnode2] [newnode2]`

注意：

启动rabbitmq之前，需要修改配置文件`vim  /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app`，修改启动loopback_users，后面为一个json数组，需要去掉无关的符号。

安装完图形界面插件，默认端口号为15672。访问地址：http://192.168.0.108:15672。默认用户名、密码为：guest

#### Hello World:

````java
public class Procuder {

	public static void main(String[] args) throws IOException, TimeoutException {
		//1.创建一个ConnectionFactory
		ConnectionFactory connectionFactory = new ConnectionFactory();

		connectionFactory.setHost("192.168.0.110");
		connectionFactory.setPort(5672);
		connectionFactory.setVirtualHost("/");

		//2.通过连接工厂创建连接
		Connection connection = connectionFactory.newConnection();

		//3.通过connection创建channel
		Channel channel = connection.createChannel();

		//4.通过channel发送消息
		for (int i = 0;i < 9999 ; i++){
			String msg = "hello rabbitmq";
			channel.basicPublish("","test001",null,msg.getBytes());
		}

		channel.close();
		connection.close();

	}
}
````

````java
public class Consumer {

	public static void main(String[] args) throws Exception{
		//1.创建一个ConnectionFactory
		ConnectionFactory connectionFactory = new ConnectionFactory();

		connectionFactory.setHost("192.168.0.110");
		connectionFactory.setPort(5672);
		connectionFactory.setVirtualHost("/");

		//2.通过连接工厂创建连接
		Connection connection = connectionFactory.newConnection();

		//3.通过connection创建channel
		Channel channel = connection.createChannel();

		//4.声明（创建）一个队列
		String queueName = "test001";
		channel.queueDeclare(queueName,true,false,false,null);

		//5.创建消费者
		QueueingConsumer queueingConsumer = new QueueingConsumer(channel);

		//6.设置channel
		channel.basicConsume(queueName,true,queueingConsumer);

		//7.获取消息
		while (true){
			QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
			String msg = new String(delivery.getBody());
			System.out.println("消费端："+msg);
			//Envelope envelope = delivery.getEnvelope();
		}

	}

}
````

#### Exchange(交换机):

​	接受消息，并根据路由键转发消息所绑定的队列。

![rabbitmq交换机](https://github.com/g453030291/java-2/blob/master/images/rabbitmq交换机.png)

Name：交换机名称

Type：交换机类型direct、topic、fanout、headers

Durability：是否需要持久化，true为持久化

Auto Delete：当最后一个绑定到Exchange上的队列删除后，自动删除该Exchange

Internal：当前Exchange是否用于RabbitMQ内部使用，默认为False

Arguments：扩展参数，用户扩展AMQP协议自定制化使用

Direct Exchange：

![Direct Exchange](https://github.com/g453030291/java-2/blob/master/images/Direct Exchange.png)

​	所有发送到Direct Exchange的消息被转发到RouteKey中指定的Queue。注意：Direct模式可以使用RabbitMQ自带的Exchange：default Exchange，所以不需要将Exchange进行任何绑定（binding）操作，消息传递时，RouteKey必须完全匹配才会被队列接受，否则该消息会被放弃。

Topic Exchange：

![Topic Exchange](https://github.com/g453030291/java-2/blob/master/images/Topic Exchange.png)

​	所有发送到Topic Exchange的消息被转发到所有关心RouteKey中指定Topic的Queue上。Exchange将RouteKey和某Topic进行模糊匹配，此时队列需要绑定一个Topic。

注意：可以使用通配符进行模糊匹配。符号"#"匹配一个或多个词，符合"*"匹配不多不少一个词。

例如："log.#"能够匹配到"log.info.oa"

​	    "log.*"只会匹配到"log.erro"

Fanout Exchange：

![Fanout Exchange](https://github.com/g453030291/java-2/blob/master/images/Fanout Exchange.png)

​	不处理路由键，只需要简单的将队列绑定到交换机上。发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。Fanout交换机转发消息是最快的。

#### Binding（绑定）：

​	Exchange和Exchange、Queue之间的连接关系。Binding中可以包含Routing Key。

#### Queue（消息队列）：

​	消息队列，实际存储消息数据。Durability，是否持久化（Durable：是，Transient：否）。Auto delete：如选yes，代表当最后一个监听被移除后，该Queue会自动被删除。

#### Message（消息）：

​	服务器和应用程序之间传送的数据。本质上就是一段数据，由Properties和Payload（Body）组成。常用属性：delivery model、headers（自定义属性）。其他属性，content_type、content_encoding、priority、correlation_id、reply_to、expiration、message_id、timestamp、type、user_id、app_id、cluster_id。

#### Virtual host（虚拟主机）：

​	虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个Virtual Host里面可以有若干个Exchange和Queue。同一个Virtual Host里面不能有相同名称的Exchange或Queue。

#### 消息如何保障100%的投递成功？

生产端-可靠性投递：

消息落库，对消息状态进行打标。

![消息落库](https://github.com/g453030291/java-2/blob/master/images/消息落库.png)

消息的延迟投递，做二次确认，回调检查。

![消息的延迟投递](https://github.com/g453030291/java-2/blob/master/images/消息的延迟投递.png)

#### 幂等性概念： 