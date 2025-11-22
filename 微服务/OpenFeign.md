## 1. 如何使用
步骤
1. 引入依赖，包括OpenFeign和负载均衡组件SpringCloudLoadBalancer
```xml
<!--OpenFeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--负载均衡-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId> 
</dependency>
<!-- 以前用的是Ribbon>
```

2. 通过 @EnableFeignClients注解，启用OpenFeign功能
```java
@EnableFeignClients
@SpringBootApplication
public class CartApplication { // ... 略 }
```
OpenFeign已经被SpringCloud自动装配，实现起来非常简单
3. 编写FeignClient

4. 使用FeignClient，实现远程调用
