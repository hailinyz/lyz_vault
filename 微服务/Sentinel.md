Sentinel 是阿里巴巴开源的一款微服务控制组件。官网地址：[[https://sentinelguard.io/zh-cn/index.html]]

启动准备好的sentinel jar包
```powershell
java -Dserver.port=8090 -Dcsp.sentinel.dashboard.server=localhost:8090 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```

![](assets/Sentinel/file-20251128074607549.png)
用户密码默认是sentinel

## 整合到微服务

1. 引入依赖
```xml
<!--sentinel-->
<dependency>
    <groupId>com.alibaba.cloud</groupId> 
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```
1. 配置控制台
```yaml
spring:
  cloud: 
    sentinel:
      transport:
        dashboard: localhost:8090
```

### 簇点链路

簇点链路就是单机调用链路。是一次请求进入服务后经过的每一个被Sentinel监控的资源链。默认Sentinel会监控SpringMVC的每一个Endpoint（http接口）。限流、熔断等都是针对簇点链路中的**资源**设置的。而资源名默认就是接口的请求路径。

Restful风格的API请求路径一般都相同，这会导致簇点资源名称重复。因此我们要修改配置，把请求方式+请求路径作为簇点资源名称:
```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8090
      http-method-specify: true # 开启请求方式前缀
```

## 请求限流

如图操作即可
![](assets/Sentinel/file-20251128081322904.png)

## 线程隔离

当商品服务出现阻塞或故障时，调用商品服务的购物车服务可能因此而被拖慢，甚至资源耗尽。所以必须限制购物车服务中查询这个业务的可用线程数，实现线程隔离。

现在我们假设说这个商品服务出现了**故障**
![](assets/Sentinel/file-20251128083457666.png)