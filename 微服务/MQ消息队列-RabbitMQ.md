
RabbitMQ的整体架构以及核心概念：
+ VirtualHost：虚拟主机，起到数据隔离的作用（类似于Mysql中的一个个的数据库）
+ publisher：消息发送者
+ consumer：消息的消费者
+ queue：队列，存储消息
+ exchange：交换机，负责路由消息
![](assets/MQ消息队列-RabbitMQ/file-20251128211953142.png)

## Spring AMQP

AMQP是用于应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求。

而Spring AMQP 就是基于AMQP定义的一套API规范，提供了模板来发送和接收消息。包含两部分，其中**spring-amqp是基础抽象，spring-rabbit是底层的默认实现。** 废话太多，其实这个东西就是让我们收发消息更加简单的

## 