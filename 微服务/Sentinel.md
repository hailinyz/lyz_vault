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