
需求：
改造余额支付功能，不在同步调用交易服务的OpenFeign接口，而是采用异步MQ通知交易服务更新订单状态。

![](assets/业务改造%20-%20从OpenFeign的同步调用改为基于MQ的异步调用/file-20251129113618989.png)
先声明队列交换机（在消费者那里去写）
这里的pay-service是支付服务，他是发送者
消费者是trade-service，也就是交易微服务

1. 首先为用到MQ的异步调用的两个服务引入相应依赖
```xml
<!--amqp-->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-amqp</artifactId>  
</dependency>
```
2. MQ地址配置，配置在trade-service 和 pay-service 的 application.yaml就行
```yaml
spring:  
  rabbitmq:  
    host: 192.168.100.128  
    port: 5672  
    virtual-host: /hmall  
    username: hmall  
    password: 123
```
当然了，以上的配置也可以配置到共享配置里面，省点心也是可以的

3. 配置消息转换器
因为消息转换器不仅仅是发送方需要，消费方也需要，所以我们就不分别写在每个微服务里了，直接卸载common里面，之后每个微服务引入common就行了

