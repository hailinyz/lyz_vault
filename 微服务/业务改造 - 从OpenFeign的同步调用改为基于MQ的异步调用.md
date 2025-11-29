
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

在config包下创建一个配置类,注意MessageConverter要是amqp包下的
```java
@Configuration  
public class MqConfig {  
  
    @Bean  
    public MessageConverter messageConverter() {  
        return new Jackson2JsonMessageConverter();  
    }  
  
}
```
现在这个配置类是没有生效的，想要使它生效，就得被扫描包扫描到，但是他是在common模块下的，我们是在trade模块或者pay模块下的，他们的包名都不一样，所以根本不可能被扫描到。

因此我们采用springboot自动装配的原理：在common模块下的 spring.factories这个文件下添加MQ的这个config。
```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\  
  com.hmall.common.config.MyBatisConfig,\  com.hmall.common.config.MvcConfig,\  com.hmall.common.config.MqConfig,\  com.hmall.common.config.JsonConfig
```
这种方式让jspringboot能够扫描到它，从而让他生效。

4. 接着我们就可以去编写消息的消费者代码了
注解部分不仅要指定队列，还要指定交换机以及绑定关系

方法部分是具体的业务代码，就是pay-service发消息，我们在这收消息，收到消息以后去吧这个订单状态标记成”已支付“

```java
@Component  
@RequiredArgsConstructor  
public class PayStatusListener {  
  
    private final IOrderService orderService;  
  
    @RabbitListener(bindings = @QueueBinding(  
            value = @Queue(name = "trade.pay.success.queue",durable = "true"),  
            exchange = @Exchange(name = "pay.direct"),  
            key = "pay.success"  
    ))  
    public void listenPaySuccess(Long orderId) {  
        orderService.markOrderPaySuccess(orderId);  
    }  
  
}
```

5. 重启消费者服务

6. 然后就可以去编写pay-service消息发送者的代码
