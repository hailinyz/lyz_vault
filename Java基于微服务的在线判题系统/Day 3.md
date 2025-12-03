## 日志框架（SLF4J+logback）

故障排查，注意日志级别，可读性，避免过度日志记录
日志的滚动(滚动策略，每天都会生成新的文件)和归档（时间太久远的日志可以不要，结合业务场景）

是SpringBoot默认的日志框架，用它更方便，就说说明我们现在不需要再引入依赖了，在配置方面配置好就行了

### 配置文件
现在开发的是oj-system，所以先在他的resource目录下创建logback.xml配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 日志存放路径 -->
    <property name="log.path" value="logs/oj-system" />
    <!-- 日志输出格式 -->
    <property name="log.pattern" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{20} - [%method,%line] - %msg%n" />

    <!-- 控制台输出 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

    <!-- 系统日志输出 -->
    <appender name="file_info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/info.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件名格式 -->
            <fileNamePattern>${log.path}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志最大的历史 10天 -->
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>INFO</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="file_error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/error.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件名格式 -->
            <fileNamePattern>${log.path}/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志最大的历史 10天 -->
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>ERROR</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--日志级别-->
    <root level="info">
        <appender-ref ref="file_info" />
        <appender-ref ref="console" />
        <appender-ref ref="file_error" />
    </root>
</configuration>
```
启动项目之后配置就能生效
![](assets/Day%203/file-20251203152616694.png)

**现在所有配置都写在bootstrap.yml文件下，有没有什么问题呢**
+ 每修改一个配置，要重新打包上线
+ 团队协作困难，你能改他也能改，用git之类的，但是麻烦
+ 环境隔离不足（开发 测试 生产），这三个配置肯定是不同的
### Nacos解决这个问题
微服务项目得要服务发现、注册中心，又可以当注册中心，所以用它
1. 拉取镜像
```powershell
docker pull nacos/nacos-server:v2.2.3
```
2. 配置外部数据库
+ 数据持久性
+ 高可用，支持集群部署
+ 性能优化，易于管理
创建库
```sql
create database bitoj_nacos_local;
```
创建表sql[nacos/distribution/conf/mysql-schema.sql at master · alibaba/nacos (github.com)](https://github.com/alibaba/nacos/blob/master/distribution/conf/mysql-schema.sql)
现在root下创建库表，再赋予用户权限