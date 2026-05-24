## 添加依赖
```xml
<!-- 构建项目指定编码集 -->
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<!-- 数据库驱动 -->
<mysql-connector.version>5.1.49</mysql-connector.version>
<!-- mybatis依赖 -->
<mybatis-starter.version>2.3.0</mybatis-starter.version>
<!-- 数据源 -->
<druid-starter.version>1.2.15</druid-starter.version>
```
![](assets/Day2/file-20260524170659252.png)
```xml
<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>${mysql-connector.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.mybatis.spring.boot</groupId>  
    <artifactId>mybatis-spring-boot-starter</artifactId>  
    <version>${mybatis-starter.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid-spring-boot-starter</artifactId>  
    <version>${druid-starter.version}</version>  
</dependency>
```

## 关于数据库的配置 在application.yml配置

