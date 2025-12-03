## 日志框架（SLF4J+logback）

故障排查，注意日志级别，可读性，避免过度日志记录
日志的滚动和归档（时间太久远的日志可以不要，结合业务场景）

是SpringBoot默认的日志框架，用它更方便，就说说明我们现在不需要再引入依赖了，在配置方面配置好就行了

### 配置文件
现在开发的是oj-system，所以先在他的resource目录下创建logback.xml配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
 <!-- ⽇志存放路径 -->
 <property name="log.path" value="logs/oj-system" />
 <!-- ⽇志输出格式 -->
 <property name="log.pattern" value="%d{HH:mm:ss.SSS} [%thread] %-5level 
%logger{20} - [%method,%line] - %msg%n" />
 <!-- 控制台输出 -->
<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
 <encoder>
 <pattern>${log.pattern}</pattern>
 </encoder>
 </appender>
 
```
