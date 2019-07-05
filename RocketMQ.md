#### RocketMQ概念模型：

Producer：消息生产者，负责产生消息，一般由业务系统负责生产消息。

Consumer：消息消费者，负责消费消息，一般是后台系统负责异步消费。

Push Consumer：Consumer的一种，需要向Consumer对象注册监听。

Pull Consumer：Consumer的一种，需要主动请求Broker拉取消息。

Producer Group：生产者集合，一般用于发送一类消息。

Consumer Group：消费者集合，一般用于接收一类消息进行消费。

Broker：MQ消息服务（中转角色，用于消息存储与生产消费转发）。

#### 搭建RocketMQ单节点环境：

一、编辑hosts文件

````shell
vim /etc/hosts

172.168.10.133 rocketmq-nameserver1
172.168.10.133 rocketmq-master1

:wq
````

二、上传apache-rocketmq.tar.gz

````shell
mkdir /usr/local/apache-rocketmq
tar -zxvf apache-rocketmq.tar.gz -C ../apache-rocketmq/
#创建软连接
ln -s apache-rocketmq rocketmq
#创建存储路径
mkdir /usr/local/rocketmq/store
mkdir /usr/local/rocketmq/store/commitlog
mkdir /usr/local/rocketmq/store/consumequeue
mkdir /usr/local/rocketmq/store/index
````

三、修改rocketmq配置文件

````properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker 名字，注意此处不同的配置文件填写的不一样 
brokerName=broker-a|broker-b
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer 地址，分号分割 
namesrvAddr=rocketmq-nameserver1:9876 
#在发送消息时，自动创建服务器不存在的 topic，默认创建的队列数 
defaultTopicQueueNums=4
#是否允许 Broker 自动创建 Topic，建议线下开启，线上关闭 
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭 
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4 点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog 每个文件的大小默认 1G 
mapedFileSizeCommitLog=1073741824
#ConsumeQueue 每个文件默认存 30W 条，根据业务情况调整 
mapedFileSizeConsumeQueue=300000 
#destroyMapedFileIntervalForcibly=120000 
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径 
storePathCommitLog=/usr/local/rocketmq/store/commitlog 
#消费队列存储路径存储路径 
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue 
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径 
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4 
#flushConsumeQueueLeastPages=2

#flushCommitLogThoroughInterval=10000 
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制 Master #- SYNC_MASTER 同步双写 Master #- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘 
#- SYNC_FLUSH 同步刷盘 
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false 
#发消息线程池数量 
#sendMessageThreadPoolNums=128 
#拉消息线程池数量 
#pullMessageThreadPoolNums=128
````

四、修改日志配置

```shell
mkdir -p /usr/local/rocketmq/logs
cd /usr/local/rocketmq/conf && sed -i 's#${user.home}#/usr/local/rocketmq#g' *.xml
```

五、修改脚本启动参数

```shell
vim /usr/local/rocketmq/bin/runbroker.sh
vim /usr/local/rocketmq/bin/runserver.sh
```

这里修改rocketmq启动的jvm参数，rocketmq比较吃内存，所以，生产环境最少8G内存，这里测试，最少内存也不能少于1G。

六、启动rocketmq

````shell
cd /usr/local/rocketmq/bin
````

启动nameserver

````shell
nohup sh mqnamesrv &
````

启动broker

````shell
nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-2s-async/broker-a.properties >/dev/null 2>&1 &
````

七、rocketmq控制台

Rocketmq-console是一个springboot项目，修改nameserveraddr配置，启动即可。

八、测试代码:

Producer

```java
public static void main(String[] args) throws Exception {
		DefaultMQProducer producer = new DefaultMQProducer("test_quick_producer_name");
		producer.setNamesrvAddr(Const.NAME_SERVER_ADDR);
		producer.start();
		for (int i= 0; i<5;i++){
			Message message = new Message("test_quick_topic", //主题
					"TagA", //标签
					"key"+i, //用户自定义的key，唯一标识
					("Hello RocketMQ"+i).getBytes()); //消息体
			SendResult sendResult = producer.send(message);
			System.out.println("消息发出"+sendResult);
		}
		producer.shutdown();
	}
```

consumer

````java
public static void main(String[] args) throws MQClientException {
		DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("test_quick_consumer_name");
		consumer.setNamesrvAddr(Const.NAME_SERVER_ADDR);
		consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);
		consumer.subscribe("test_quick_topic","*");
		consumer.registerMessageListener(new MessageListenerConcurrently() {
			@Override
			public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
				MessageExt me = msgs.get(0);
				try {
					String topic = me.getTopic();
					String tags = me.getTags();
					String keys = me.getKeys();
					String msgBody = new String(me.getBody(), RemotingHelper.DEFAULT_CHARSET);
	System.out.println("topic:"+topic+",tags:"+tags+",keys:"+keys+",body:"+msgBody);
				}catch (Exception e){
					e.printStackTrace();
					int reconsumeTimes = me.getReconsumeTimes();
					if(reconsumeTimes == 3){
						//重试3次消费,还不成功,就直接返回成功,记录日志,做补偿处理
						return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
					}
					return ConsumeConcurrentlyStatus.RECONSUME_LATER;
				}
				return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
			}
		});
		consumer.start();
		System.out.println("consumer start......");
	}
````

Const

```java
public static final String NAME_SERVER_ADDR = "172.168.10.166:9876";
```

#### 主从模式集群环境：

特点：

1.主从模式环境构建可以保障消息的即时性与可靠性

2.投递一条消息后，关闭主节点

3.从节点继续可以提供消费者数据进行消费，但是不能接受消息

4.主节点上线后进行消费进度offset同步

关闭之前启动的服务：

```shell
sh mqshutdown broker
sh mqshutdown namesrv
```

