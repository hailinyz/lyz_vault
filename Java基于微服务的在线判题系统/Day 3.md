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
```sql
GRANT CREATE, DROP, SELECT, INSERT, UPDATE, DELETE ON bitoj_nacos_dev.* TO 'ojtest'@'%';
```
3. 启动nacos容器
将以单例模式启动开放
```powershell
docker run -d -p 8848:8848 -p 9848:9848 --name oj-nacos -e MODE=standalone -e JVM_XMS=256m -e JVM_XMX=256m -e SPRING_DATASOURCE_PLATFORM=mysql -e MYSQL_SERVICE_HOST=${mysql_ip} -e MYSQL_SERVICE_PORT=${mysql_port} -e MYSQL_SERVICE_DB_NAME=${nacos_db_name} -e MYSQL_SERVICE_USER=${mysql_user} -e MYSQL_SERVICE_PASSWORD=${mysql_password} nacos/nacos-server:v2.2.3
```
找到IPAddress：docker inspect oj-mysql
![](assets/Day%203/file-20251203210024777.png)
```powershell
docker run -d -p 8848:8848 -p 9848:9848 --name oj-nacos -e MODE=standalone -e JVM_XMS=256m -e JVM_XMX=256m -e SPRING_DATASOURCE_PLATFORM=mysql -e MYSQL_SERVICE_HOST=172.17.0.2 -e MYSQL_SERVICE_PORT=3306 -e MYSQL_SERVICE_DB_NAME=bitoj_nacos_local -e MYSQL_SERVICE_USER=ojtest -e MYSQL_SERVICE_PASSWORD=123456 nacos/nacos-server:v2.2.3
```
172.17.0.2为什么用这个后面会说


```sql
create database bitoj_nacos_local;  
  
use bitoj_nacos_local;  
  
CREATE TABLE `config_info` (  
 `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',  
 `data_id` varchar(255) NOT NULL COMMENT 'data_id',  
 `group_id` varchar(128) DEFAULT NULL COMMENT 'group_id',  
 `content` longtext NOT NULL COMMENT 'content',  
 `md5` varchar(32) DEFAULT NULL COMMENT 'md5',  
 `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
 `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',  
 `src_user` text COMMENT 'source user',  
 `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',  
 `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',  
 `tenant_id` varchar(128) DEFAULT '' COMMENT '租⼾字段',  
 `c_desc` varchar(256) DEFAULT NULL COMMENT 'configuration description',  
 `c_use` varchar(64) DEFAULT NULL COMMENT 'configuration usage',  
 `effect` varchar(64) DEFAULT NULL COMMENT '配置⽣效的描述',  
 `type` varchar(64) DEFAULT NULL COMMENT '配置的类型',  
 `c_schema` text COMMENT '配置的模式',  
 `encrypted_data_key` text NOT NULL COMMENT '密钥',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';  
/******************************************/  
/* 表名称 = config_info_aggr *//******************************************/  
CREATE TABLE `config_info_aggr` (  
 `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',  
 `data_id` varchar(255) NOT NULL COMMENT 'data_id',  
 `group_id` varchar(128) NOT NULL COMMENT 'group_id',  
 `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',  
 `content` longtext NOT NULL COMMENT '内容',  
 `gmt_modified` datetime NOT NULL COMMENT '修改时间',  
 `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',  
 `tenant_id` varchar(128) DEFAULT '' COMMENT '租⼾字段',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum`  
(`data_id`,`group_id`,`tenant_id`,`datum_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租⼾字段';  
/******************************************/  
/* 表名称 = config_info_beta *//******************************************/  
CREATE TABLE `config_info_beta` (  
 `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',  
 `data_id` varchar(255) NOT NULL COMMENT 'data_id',  
 `group_id` varchar(128) NOT NULL COMMENT 'group_id', `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',  
 `content` longtext NOT NULL COMMENT 'content',  
 `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',  
 `md5` varchar(32) DEFAULT NULL COMMENT 'md5',  
 `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
 `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',  
 `src_user` text COMMENT 'source user',  
 `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',  
 `tenant_id` varchar(128) DEFAULT '' COMMENT '租⼾字段',  
 `encrypted_data_key` text NOT NULL COMMENT '密钥',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_configinfobeta_datagrouptenant`  
(`data_id`,`group_id`,`tenant_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin  
COMMENT='config_info_beta';  
/******************************************/  
/* 表名称 = config_info_tag *//******************************************/  
CREATE TABLE `config_info_tag` (  
 `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',  
 `data_id` varchar(255) NOT NULL COMMENT 'data_id',  
 `group_id` varchar(128) NOT NULL COMMENT 'group_id',  
 `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',  
 `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',  
 `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',  
 `content` longtext NOT NULL COMMENT 'content',  
 `md5` varchar(32) DEFAULT NULL COMMENT 'md5',  
 `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
 `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',  
 `src_user` text COMMENT 'source user',  
 `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_configinfotag_datagrouptenanttag`  
(`data_id`,`group_id`,`tenant_id`,`tag_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin  
COMMENT='config_info_tag';  
/******************************************/  
/* 表名称 = config_tags_relation *//******************************************/  
CREATE TABLE `config_tags_relation` (  
 `id` bigint(20) NOT NULL COMMENT 'id',  
 `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',  
 `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',  
 `data_id` varchar(255) NOT NULL COMMENT 'data_id',  
 `group_id` varchar(128) NOT NULL COMMENT 'group_id',`tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',  
 `nid` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'nid, ⾃增⻓标识',  
 PRIMARY KEY (`nid`),  
 UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),  
 KEY `idx_tenant_id` (`tenant_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin  
COMMENT='config_tag_relation';  
/******************************************/  
/* 表名称 = group_capacity *//******************************************/  
CREATE TABLE `group_capacity` (  
 `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',  
 `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表⽰整个集  
群',  
 `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表⽰使⽤默认值',  
 `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使⽤量',  
 `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置⼤⼩上限，单位  
为字节，0表⽰使⽤默认值',  
 `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合⼦配置最⼤  
个数，，0表⽰使⽤默认值',  
 `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的⼦  
配置⼤⼩上限，单位为字节，0表⽰使⽤默认值',  
 `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最⼤变更历史  
数量',  
 `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
 `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_group_id` (`group_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量  
信息表';  
/******************************************/  
/* 表名称 = his_config_info *//******************************************/  
CREATE TABLE `his_config_info` (  
 `id` bigint(20) unsigned NOT NULL COMMENT 'id',  
 `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'nid, ⾃增标识',  
 `data_id` varchar(255) NOT NULL COMMENT 'data_id',  
 `group_id` varchar(128) NOT NULL COMMENT 'group_id',  
 `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',  
 `content` longtext NOT NULL COMMENT 'content',  
 `md5` varchar(32) DEFAULT NULL COMMENT 'md5',  
 `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
 `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',  
 `src_user` text COMMENT 'source user',  
 `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip', `op_type` char(10) DEFAULT NULL COMMENT 'operation type',  
 `tenant_id` varchar(128) DEFAULT '' COMMENT '租⼾字段',  
 `encrypted_data_key` text NOT NULL COMMENT '密钥',  
 PRIMARY KEY (`nid`),  
 KEY `idx_gmt_create` (`gmt_create`),  
 KEY `idx_gmt_modified` (`gmt_modified`),  
 KEY `idx_did` (`data_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租⼾改造';  
/******************************************/  
/* 表名称 = tenant_capacity *//******************************************/  
CREATE TABLE `tenant_capacity` (  
 `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',  
 `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',  
 `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表⽰使⽤默认值',  
 `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使⽤量',  
 `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置⼤⼩上限，单位  
为字节，0表⽰使⽤默认值',  
 `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合⼦配置最⼤  
个数',  
 `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的⼦  
配置⼤⼩上限，单位为字节，0表⽰使⽤默认值',  
 `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最⼤变更历史  
数量',  
 `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',  
 `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_tenant_id` (`tenant_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租⼾容量信息表';  
CREATE TABLE `tenant_info` (  
 `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',  
 `kp` varchar(128) NOT NULL COMMENT 'kp',  
 `tenant_id` varchar(128) default '' COMMENT 'tenant_id',  
 `tenant_name` varchar(128) default '' COMMENT 'tenant_name',  
 `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',  
 `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',  
 `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',  
 `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',  
 PRIMARY KEY (`id`),  
 UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),  
 KEY `idx_tenant_id` (`tenant_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';CREATE TABLE `users` (  
 `username` varchar(50) NOT NULL PRIMARY KEY COMMENT 'username',  
 `password` varchar(500) NOT NULL COMMENT 'password',  
 `enabled` boolean NOT NULL COMMENT 'enabled'  
);  
CREATE TABLE `roles` (  
 `username` varchar(50) NOT NULL COMMENT 'username',  
 `role` varchar(50) NOT NULL COMMENT 'role',  
 UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE  
);  
CREATE TABLE `permissions` (  
 `role` varchar(50) NOT NULL COMMENT 'role',  
 `resource` varchar(128) NOT NULL COMMENT 'resource',  
 `action` varchar(8) NOT NULL COMMENT 'action',  
 UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE  
);  
INSERT INTO users (username, password, enabled) VALUES ('nacos',  
'$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);  
INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

http://localhost:8848/nacos/
随随便便就进来了，不安全，到服务器上部署再提这个安全

既然nacos已经配置好了，直接将bootstrap.yml文件往nacos移动
