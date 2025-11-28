
RabbitMQ的整体架构以及核心概念：
+ VirtualHost：虚拟主机，起到数据隔离的作用（类似于Mysql中的一个个的数据库）
+ publisher：消息发送者
+ consumer：消息的消费者
+ queue：队列，存储消息
+ exchange：交换机，负责路由消息
![](assets/MQ消息队列-RabbitMQ/file-20251128211953142.png)
