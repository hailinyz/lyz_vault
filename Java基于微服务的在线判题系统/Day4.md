**我们在线oj项目用的就是jwt认证方式**

## 身份认证机制

### JWT组成
+ 头部，令牌类型和使用的算法，使用base64编码
+ 载荷，用户的信息（昵称、id，并不是明文直接存）和其他元数据，使用base64编码
+ 签名，验证令牌完整、真实性

通过 . 分割三个部分

**为什么使用jwt**
在gateway微服务进行jwt验证，因为jwt是自包含（减轻服务器负担）、安全（签名）、无状态（不存储），每个服务身份认证这块就能独立操作，扩展起来就很方便；支持跨域

**有没有什么坑，仅仅使用jwt就可以吗？**
+ jwt采用base64编码，不能存敏感信息，完全暴露出来的，能解码得到原始的数据，token泄露就完了
+ 因为无状态，想要修改里面的内容就必须重新签发（用户修改个人信息需重新登录）
+ 无法演唱jwt过期时间，用户正在操作时突然身份认证失败

**怎么既要又要呢**
+ payload中不能存放敏感信息
仅仅存储用户唯一标识信息，第三方集中存储敏感信息，根据唯一标识，从第三方中查出响应的敏感信息，第三方存储查询性能一定要高，没必要长期存储，用**Redis**
+ 用户修改个人信息，jwt不变
+ 控制jwt提供的过期时间
不能再使用jwt提供的过期时间的参数/方法，借助第三方维护过期时间，还是**Redis**

所以综上所述，记录jwt过期时间，并且支持过期时间的修改，最好还能有存储功能，那必然用的是**Redis**+JWT 实现身份认证机制

## 安装Redis

1. 拉取镜像
```powershell
docker pull redis
```
2. 启动redis容器
```powershell
docker run --name oj-redis -d -p 6379:6379 redis --requirepass "123456"
```
3. 在oj-common创建相应的redis工程，引入依赖
```xml
<!-- SpringBoot Boot Redis -->
 <dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-data-redis</artifactId>
 </dependency>
 <!-- Alibaba Fastjson -->
 <dependency>
 <groupId>com.alibaba.fastjson2</groupId>
 <artifactId>fastjson2</artifactId>
 <version>2.0.43</version>
 </dependency>
```
也要进行版本的管理
4. 对redis进行相应的配置
+ 写一个关于json序列化工具
JsonRedisSerializer类，序列化和反序列化
