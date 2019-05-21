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

172.168.10.104 rocketmq-nameserver1
172.168.10.104 rocketmq-master1

:wq
````

