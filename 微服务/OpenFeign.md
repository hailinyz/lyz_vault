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
3. 编写FeignClient(这其实是一个接口)
```java
@FeignClient(value = "item-service")
public interface ItemClient {
    @GetMapping("/items")
    List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
}
```
3. 使用FeignClient，实现远程调用
```java
List<ItemDTO> items = itemClient.queryItemByIds(List.of(1,2,3));
```

OpenFegn固然方便快捷丝滑，但是：
底层发起http请求用的是jdk默认的HttpURLConnection，每次请求都会重新创建连接，慢，所以可以依赖其他的框架：
+ HttpURLConnection： 默认，不支持连接池
+ Apache HttpClient 支持连接池
+ OKHttp: 支持连接池
这里我选泽的是OKHttp

步骤如下：
1. 引入依赖
```xml
<!--ok-http-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```
1. 开启连接池功能
```ymal
feign:
  okhttp:
    enabled: true #开启OKHttp连接池支持
```
