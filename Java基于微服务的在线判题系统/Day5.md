## Entity、DTO、VO

Entity：实体类
DTO：后端接收前端请求参数，通常只包含需要返回的数据
VO：后端返回给前端的数据

解耦，修改每一层的时候对其他层的影响降低
合理利用网络资源，在互相传输数据过程中，尽可能用到哪些封装哪些

如果划分太细可能未来维护困难

## 对参数的校验

引入依赖
```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

