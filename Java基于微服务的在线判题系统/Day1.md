# 技术选型

| 技术分类           | 具体技术栈                                  |
| -------------- | -------------------------------------- |
| 前端技术           | vue3、html、JavaScript、scss、element plus |
| 服务架构           | Spring Cloud 微服务架构                     |
| 代理服务器          | Nginx                                  |
| 分布式任务调度中心      | Xxl-Job                                |
| 注册与发现中心 / 配置中心 | Nacos                                  |
| 服务间调用          | OpenFeign                              |
| 网关             | Spring Cloud Gateway                   |
| 数据存储           | MySQL                                  |
| 数据库持久层         | MyBatis/MyBatis-Plus                   |
| 缓存             | Redis                                  |
| 消息队列           | rabbitMQ                               |
| 搜索引擎           | ElasticSearch                          |
| 加密算法           | Bcrypt                                 |
| 身份认证           | JWT                                    |
| 代码沙箱           | Docker                                 |
| 对象存储           | OSS                                    |
| 短信服务           | 阿里云短信服务                                |

# 开发环境

Windows - 10/11
后端：
+ idea 社区版 2022.1.4
+ JDK17
+ SpringBoot 3 （3.0.1）
+ SpringCloud
+ docker
前端：
+ Node.js 18.3 或以上
部署环境：
+ 开发完成后再说

## 系统架构-CS与BS架构
### CS架构
Client客户端-Socket服务器
![](assets/Day1/file-20251129215233658.png)
特点：
+ 必须要安装客户端程序，不同的应用需要安装不同的客户端程序
+ 客户端不能处理才会请求服务器
+ 客户端承担更多的业务逻辑，可以直接操作数据库等服务
例如：QQ、微信、王者荣耀

### BS架构
浏览器(browser)-服务器架构
![](assets/Day1/file-20251129215453739.png)
特点：
+ 客户端是浏览器，不同的BS架构的应用客户端是同一个，并且都是浏览器
+ 客户端不处理任何的业务逻辑，所有业务逻辑交给服务器处理，如果操作第三方的组件或者服务也是通过服务器进行操作的
