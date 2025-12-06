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

## 引入Redis

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
自定义序列化器 JsonRedisSerializer类，序列化和反序列化
```java
public class JsonRedisSerializer<T> implements RedisSerializer<T> {  
    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");  
    private Class<T> clazz;  
    public JsonRedisSerializer(Class<T> clazz) {  
        super();  
        this.clazz = clazz;  
    }  
    @Override  
    public byte[] serialize(T t) throws SerializationException {  
        if (t == null) {  
            return new byte[0];  
        }  
        return JSON.toJSONString(t).getBytes(DEFAULT_CHARSET);  
    }  
    @Override  
    public T deserialize(byte[] bytes) throws SerializationException {  
        if (bytes == null || bytes.length <= 0) {  
            return null;  
        }  
        String str = new String(bytes, DEFAULT_CHARSET);  
        return JSON.parseObject(str, clazz);  
    }  
}
```

加redis配置RedisConfig
```java
/**  
* redis配置  
 */  
@Configuration  
public class RedisConfig extends CachingConfigurerSupport {  
    @Bean  
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory  
                                                               connectionFactory) {  
        RedisTemplate<Object, Object> template = new RedisTemplate<>();  
        template.setConnectionFactory(connectionFactory);  
        JsonRedisSerializer serializer = new JsonRedisSerializer(Object.class);  
        // 使⽤StringRedisSerializer来序列化和反序列化redis的key值  
        template.setKeySerializer(new StringRedisSerializer());  
        template.setValueSerializer(serializer);  
        // Hash的key也采⽤StringRedisSerializer的序列化⽅式  
        template.setHashKeySerializer(new StringRedisSerializer());  
        template.setHashValueSerializer(serializer);  
        template.afterPropertiesSet();  
        return template;  
    }  
}
```

封装RedisService
```java
@Component  
public class RedisService {  
    @Autowired  
    public RedisTemplate redisTemplate;  
    //************************ 操作key ***************************  
    /**     * 判断 key是否存在  
     *  
     * @param key 键  
     * @return true 存在 false不存在  
     */  
    public Boolean hasKey(String key) {  
        return redisTemplate.hasKey(key);  
    }  
    /**  
     * 设置有效时间  
     *  
     * @param key Redis键  
     * @param timeout 超时时间  
     * @return true=设置成功；false=设置失败  
     */  
    public boolean expire(final String key, final long timeout) {  
        return expire(key, timeout, TimeUnit.SECONDS);  
    }/**  
     * 设置有效时间  
     *  
     * @param key Redis键  
     * @param timeout 超时时间  
     * @param unit 时间单位  
     * @return true=设置成功；false=设置失败  
     */  
    public boolean expire(final String key, final long timeout, final TimeUnit  
            unit) {  
        return redisTemplate.expire(key, timeout, unit);  
    }  
  
    /**  
     * 删除单个对象  
     *  
     * @param key  
     */  
    public boolean deleteObject(final String key) {  
        return redisTemplate.delete(key);  
    }  
    //************************ 操作String类型 ***************************    /**     * 缓存基本的对象，Integer、String、实体类等  
     *  
     * @param key 缓存的键值  
     * @param value 缓存的值  
     */  
    public <T> void setCacheObject(final String key, final T value) {  
        redisTemplate.opsForValue().set(key, value);  
    }  
    /**  
     * 缓存基本的对象，Integer、String、实体类等  
     *  
     * @param key 缓存的键值  
     * @param value 缓存的值  
     * @param timeout 时间  
     * @param timeUnit 时间颗粒度  
     */  
    public <T> void setCacheObject(final String key, final T value, final Long  
            timeout, final TimeUnit timeUnit) {  
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);  
    }/**  
     * 获得缓存的基本对象。  
     *  
     * @param key 缓存键值  
     * @return 缓存键值对应的数据  
     */  
    public <T> T getCacheObject(final String key, Class<T> clazz) {  
        ValueOperations<String, T> operation = redisTemplate.opsForValue();  
        T t = operation.get(key);  
        if (t instanceof String) {  
            return t;  
        }  
        return JSON.parseObject(String.valueOf(t), clazz);  
    }  
    //*************** 操作list结构 ****************    /**     * 获取list中存储数据数量  
     *  
     * @param key  
     * @return  
     */    public Long getListSize(final String key) {  
        return redisTemplate.opsForList().size(key);  
    }  
    /**  
     * 获取list中指定范围数据  
     *  
     * @param key  
     * @param start  
     * @param end  
     * @param clazz  
     * @param <T>  
     * @return  
     */    public <T> List<T> getCacheListByRange(final String key, long start, long  
            end, Class<T> clazz) {  
        List range = redisTemplate.opsForList().range(key, start, end);  
        if (CollectionUtils.isEmpty(range)) {  
            return null;  
        }  
        return JSON.parseArray(JSON.toJSONString(range), clazz);  
    }  
/*** 底层使⽤list结构存储数据(尾插 批量插⼊)  
 */public <T> Long rightPushAll(final String key, Collection<T> list) {  
    return redisTemplate.opsForList().rightPushAll(key, list);  
}  
    /**  
     * 底层使⽤list结构存储数据(头插)  
     */    public <T> Long leftPushForList(final String key, T value) {  
        return redisTemplate.opsForList().leftPush(key, value);  
    }  
    /**  
     * 底层使⽤list结构,删除指定数据  
     */  
    public <T> Long removeForList(final String key, T value) {  
        return redisTemplate.opsForList().remove(key, 1L, value);  
    }  
    //************************ 操作Hash类型 ***************************    public <T> T getCacheMapValue(final String key, final String hKey,  
                                  Class<T> clazz) {  
        Object cacheMapValue = redisTemplate.opsForHash().get(key, hKey);  
        if (cacheMapValue != null) {  
            return JSON.parseObject(String.valueOf(cacheMapValue), clazz);  
        }  
        return null;  
    }  
    /**  
     * 获取多个Hash中的数据  
     *  
     * @param key Redis键  
     * @param hKeys Hash键集合  
     * @param clazz 待转换对象类型  
     * @param <T> 泛型  
     * @return Hash对象集合  
     */  
    public <T> List<T> getMultiCacheMapValue(final String key, final  
    Collection<String> hKeys, Class<T> clazz) {  
        List list = redisTemplate.opsForHash().multiGet(key, hKeys);  
        List<T> result = new ArrayList<>();  
        for (Object item : list) {  
            result.add(JSON.parseObject(JSON.toJSONString(item), clazz));}  
        return result;  
    }  
    /**  
     * 往Hash中存⼊数据  
     *  
     * @param key Redis键  
     * @param hKey Hash键  
     * @param value 值  
     */  
    public <T> void setCacheMapValue(final String key, final String hKey, final  
    T value) {  
        redisTemplate.opsForHash().put(key, hKey, value);  
    }  
    /**  
     * 缓存Map  
     *     * @param key  
     * @param dataMap  
     */  
    public <K, T> void setCacheMap(final String key, final Map<K, T> dataMap) {  
        if (dataMap != null) {  
            redisTemplate.opsForHash().putAll(key, dataMap);  
        }  
    }  
    public Long deleteCacheMapValue(final String key, final String hKey) {  
        return redisTemplate.opsForHash().delete(key, hKey);  
    }  
}
```
存放文件如图
![](assets/Day4/file-20251205204752987.png)
为了让其他微服务能扫描到要完成下图操作在resource下创建
![](assets/Day4/file-20251205205614316.png)

别忘了配置redis的连接配置，密码地址等
```yaml
server:
  port: 9201
# Spring
spring:
  data:
    redis:
      host: localhost
      password: 123456

  datasource:
   url: jdbc:mysql://localhost:3307/bitoj_dev?useUnicode=true&characterEncoding=utf8&useSSL=true&serverTimezone=GMT%2B8
   username: ojtest
   password: 123456
   hikari:
    minimum-idle: 5 # 最⼩空闲连接数
    maximum-pool-size: 20 # 最⼤连接数
    idle-timeout: 30000 # 空闲连接存活时间（毫秒）
    connection-timeout: 30000 # 连接超时时间（毫秒
 
```

重启项目就可以了
![](assets/Day4/file-20251205212321107.png)

## 身份认证机制-功能实现

### 引入jwt
```xml
<dependency>
 <groupId>io.jsonwebtoken</groupId>
 <artifactId>jjwt</artifactId>
 <version>0.9.1</version>
</dependency>
<dependency>
 <groupId>javax.xml.bind</groupId>
 <artifactId>jaxb-api</artifactId>
 <version>2.4.0-b180830.0359</version>
</dependency>
<dependency>
 <groupId>cn.hutool</groupId>
 <artifactId>hutool-all</artifactId>
 <version>5.8.22</version>
</dependency>

```
### 创建jwt工具类

在common的security的util包下创建
![](assets/Day4/file-20251205215050307.png)
```java
public class JwtUtils {  
    /**  
     * ⽣成令牌  
     *  
     * @param claims 数据  
     * @param secret 密钥  
     * @return 令牌  
     */  
    public static String createToken(Map<String, Object> claims, String secret)  
    {  
        String token =  
                Jwts.builder().setClaims(claims).signWith(SignatureAlgorithm.HS512,  
                        secret).compact();  
        return token;  
    }  
    /**  
     * 从令牌中获取数据  
     *  
     * @param token 令牌  
     * @param secret 密钥  
     * @return 数据  
     */  
    public static Claims parseToken(String token, String secret) {  
        return  
                Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();  
    }  
}
```

### 捋一下逻辑，完成用户身份认证的三大步骤

1. 用户登录成功后，调用createToken 生成令牌 并 发送给客户端

**放在nacos上就能满足secret的保密、随机、不硬编码、定期更换**
+ 在实现类里面声明注解@Value("${jwt.secret}")，并在实现类上面声明@RefreshScope，实现动态刷新，当更新配置文件时，会自动更新无需重启
```yaml
# 在oj-system-local.yaml下
jwt:
  secret: sdfghuijasxdjkawskuigy
```

+ 封装生成token的方法TokenService
```java
//操作用户登录token的方法  
@Service  
public class TokenService {  
  
    @Autowired  
    private RedisService redisService;  
  
  
    public String createToken(Long userId, String secret,Integer identity){  
  
        Map<String, Object> claims = new HashMap<>();  
        String userKey = UUID.fastUUID().toString();  
        claims.put(JwtConstants.LOGIN_USER_ID, userId);  
        claims.put(JwtConstants.LOGIN_USER_KRY, userKey);  
        String token = JwtUtils.createToken(claims, secret);  
  
        //第三方中存放敏感信息  
        //身份认证具体存储的信息，redis 表明用户身份字段 identity 1:用户 2:管理员 对象好扩展一点LoginUser  
        // 使用啥样的数据结构 String hash list set zset        // key必须唯一，便于维护  统一前缀：logintoken:userId 是通过雪花生成所以唯一  
        //也可以用糊涂工具生成UUID作为唯一标识，跟前缀拼接实现唯一  
        // ，过期时间咋记录，定多长  720分钟  
        String key = CacheConstants.LOGIN_TOKEN_KET + userKey;  
        LoginUser loginUser = new LoginUser();  
        loginUser.setIdentity(identity);  
        redisService.setCacheObject(key, loginUser,CacheConstants.EXP, TimeUnit.MINUTES);  
  
        return token;  
  
    }  
  
}
```

然后完成一系列封装，比如枚举、LoginUser类的封装、还有模块的相关依赖


2. 后续的所有请求，在调用具体接口前，都要先通过token进行身份认证
身份认证代码
```java
package com.bite.gateway;  
import cn.hutool.core.util.StrUtil;  
import com.alibaba.fastjson2.JSON;  
import com.bite.common.core.constants.CacheConstants;  
import com.bite.common.core.constants.HttpConstants;  
import com.bite.common.core.domain.R;  
import com.bite.common.core.enums.ResultCode;  
import com.bite.common.core.enums.UserIdentity;  
import com.bite.common.redis.service.RedisService;  
import com.bite.common.security.domain.LoginUser;  
import com.bite.common.security.util.JwtUtils;  
import io.jsonwebtoken.Claims;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.cloud.gateway.filter.GatewayFilterChain;  
import org.springframework.cloud.gateway.filter.GlobalFilter;  
import org.springframework.core.Ordered;  
import org.springframework.core.io.buffer.DataBuffer;  
import org.springframework.http.HttpHeaders;  
import org.springframework.http.HttpStatus;  
import org.springframework.http.MediaType;  
import org.springframework.http.server.reactive.ServerHttpRequest;  
import org.springframework.http.server.reactive.ServerHttpResponse;  
import org.springframework.stereotype.Component;  
import org.springframework.util.AntPathMatcher;  
import org.springframework.util.CollectionUtils;  
import org.springframework.web.server.ServerWebExchange;  
import reactor.core.publisher.Mono;  
import java.util.List;  
/**  
 * 网关鉴权  
 *  
 */@Slf4j  
@Component  
public class AuthFilter implements GlobalFilter, Ordered {  
    // 排除过滤的 uri 白名单地址，在nacos自行添加  
    @Autowired  
    private IgnoreWhiteProperties ignoreWhite;  
    @Value("${jwt.secret}")  
    private String secret;  
    @Autowired  
    private RedisService redisService;  
    @Override  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain  
            chain) {  
        ServerHttpRequest request = exchange.getRequest();  
        String url = request.getURI().getPath(); //请求接口地址  
        // 跳过不需要验证的路径  
        if (matches(url, ignoreWhite.getWhites())) { //拿到nacos上的白名单  
            return chain.filter(exchange);  
        }  
        //从http请求头中获取token  
        String token = getToken(request);  
        if (StrUtil.isEmpty(token)) {  
            return unauthorizedResponse(exchange, "令牌不能为空");  
        }  
        Claims claims;  
        try {  
            claims = JwtUtils.parseToken(token, secret); //获取令牌中信息 解析payload中信息  
            if (claims == null) {  
                return unauthorizedResponse(exchange, "令牌已过期或验证不正确！");  
            }  
        } catch (Exception e) {  
            return unauthorizedResponse(exchange, "令牌已过期或验证不正确！");  
        }  
        String userKey = JwtUtils.getUserKey(claims); //获取jwt中的key  
        boolean isLogin = redisService.hasKey(getTokenKey(userKey));  
        if (!isLogin) {  
            return unauthorizedResponse(exchange, "登录状态已过期");  
        }  
        String userId = JwtUtils.getUserId(claims); //判断jwt中的信息是否完整  
        if (StrUtil.isEmpty(userId)) {  
            return unauthorizedResponse(exchange, "令牌验证失败");  
        }  
  
        //到这里就判断redis中存储的关于用户身份认证的信息是否是对的  
        LoginUser user = redisService.getCacheObject(getTokenKey(userKey),  
                LoginUser.class);  
        if (url.contains(HttpConstants.SYSTEM_URL_PREFIX) &&  
                !UserIdentity.ADMIN.getValue().equals(user.getIdentity())) {  
            return unauthorizedResponse(exchange, "令牌验证失败");  
        }  
        if (url.contains(HttpConstants.FRIEND_URL_PREFIX) &&  
                !UserIdentity.ORDINARY.getValue().equals(user.getIdentity())) {  
            return unauthorizedResponse(exchange, "令牌验证失败");  
        }  
        return chain.filter(exchange);  
    }  
    /**  
     * 查找指定url是否匹配指定匹配规则链表中的任意一个字符串  
     *  
     * @param url 指定url  
     * @param patternList 需要检查的匹配规则链表  
     * @return 是否匹配  
     */  
    private boolean matches(String url, List<String> patternList) {  
        if (StrUtil.isEmpty(url) || CollectionUtils.isEmpty(patternList)) {  
            return false;}  
        //接口地址如果和白名单其中一个匹配，则返回true，否则返回false  
        for (String pattern : patternList) {  
            if (isMatch(pattern, url)) {  
                return true;  
            }  
        }  
        return false;  
    }  
    /**  
     * 判断url是否与规则匹配  
     * 匹配规则中：  
     * ? 表示单个字符;  
     * * 表示一层路径内的任意字符串，不可跨层级;  
     * ** 表示任意层路径;  
     *     * @param pattern 匹配规则  
     * @param url 需要匹配的url  
     * @return 是否匹配  
     */  
    private boolean isMatch(String pattern, String url) {  
        AntPathMatcher matcher = new AntPathMatcher();  
        return matcher.match(pattern, url);  
    }  
    /**  
     * 获取缓存key  
     */    private String getTokenKey(String token) {  
        return CacheConstants.LOGIN_TOKEN_KET + token;  
    }  
    /**  
     * 从请求头中获取请求token  
     */    private String getToken(ServerHttpRequest request) {  
        String token =  
                request.getHeaders().getFirst(HttpConstants.AUTHENTICATION);  
        // 如果前端设置了令牌前缀，则裁剪掉前缀  
        if (StrUtil.isNotEmpty(token) &&  
                token.startsWith(HttpConstants.PREFIX)) {  
            token = token.replaceFirst(HttpConstants.PREFIX, StrUtil.EMPTY);  
        }  
        return token;  
    }  
    private Mono<Void> unauthorizedResponse(ServerWebExchange exchange, String  
            msg) {  
        log.error("[鉴权异常处理]请求路径:{}", exchange.getRequest().getPath());  
        return webFluxResponseWriter(exchange.getResponse(), msg,  
                ResultCode.FAILED_UNAUTHORIZED.getCode());  
    }  
    //拼装webflux模型响应  
    private Mono<Void> webFluxResponseWriter(ServerHttpResponse response,  
                                             String msg, int code) {  
        response.setStatusCode(HttpStatus.OK);  
        response.getHeaders().add(HttpHeaders.CONTENT_TYPE,  
                MediaType.APPLICATION_JSON_VALUE);  
        R<?> result = R.fail(code, msg);  
        DataBuffer dataBuffer =  
                response.bufferFactory().wrap(JSON.toJSONString(result).getBytes());  
        return response.writeWith(Mono.just(dataBuffer));  
    }  
    @Override  
    public int getOrder() {  
        return -200;  
    }  
    public static void main(String[] args) {  
        AuthFilter authFilter = new AuthFilter();  
// 测试 ?        String pattern = "/sys/?bc";  
        System.out.println(authFilter.isMatch(pattern,"/sys/abc"));  
        System.out.println(authFilter.isMatch(pattern,"/sys/cbc"));  
        System.out.println(authFilter.isMatch(pattern,"/sys/acbc"));  
        System.out.println(authFilter.isMatch(pattern,"/sdsa/abc"));  
        System.out.println(authFilter.isMatch(pattern,"/sys/abcw"));  
// 测试*  
// String pattern = "/sys/*/bc";  
// System.out.println(authFilter.isMatch(pattern,"/sys/a/bc"));  
//  
        System.out.println(authFilter.isMatch(pattern,"/sys/sdasdsadsad/bc"));  
// System.out.println(authFilter.isMatch(pattern,"/sys/a/b/bc"));  
// System.out.println(authFilter.isMatch(pattern,"/a/b/bc"));  
// System.out.println(authFilter.isMatch(pattern,"/sys/a/b/"));  
// 测试**  
// String pattern = "/sys/**/bc";  
// System.out.println(authFilter.isMatch(pattern, "/sys/a/bc"));// System.out.println(authFilter.isMatch(pattern,  
//"/sys/sdasdsadsad/bc"));  
//// System.out.println(authFilter.isMatch(pattern, "/sys/a/b/bc"));  
//// System.out.println(authFilter.isMatch(pattern,  
//"/sys/a/b/s/23/432/fdsf///bc"));  
//// System.out.println(authFilter.isMatch(pattern,  
//"/a/b/s/23/432/fdsf///bc"));  
//// System.out.println(authFilter.isMatch(pattern,  
//"/sys/a/b/s/23/432/fdsf///"));  
 }  
}
```

```java
package com.bite.common.core.constants;  
  
public class HttpConstants {  
    /**  
     * 服务端url标识  
     */  
    public static final String SYSTEM_URL_PREFIX = "system";  
    /**  
     * 用户端url标识  
     */  
    public static final String FRIEND_URL_PREFIX = "friend";  
    /**  
     * 令牌自定义标识  
     */  
    public static final String AUTHENTICATION = "Authorization";  
    /**  
     * 令牌前缀  
     */  
    public static final String PREFIX = "Bearer ";  
}
```
+ 登录的时候是不需要进行身份认证的，跳过不需要验证的路径，接口白名单中接口均不需认证
+ 一些列判断，判断redis中存储的关于用户身份认证的信息是否是对的，我觉得好麻烦
+ C端用户和管理端用户判断
创建IgnoreWhiteProperties配置类
```java
@Configuration  
@RefreshScope  
@ConfigurationProperties(prefix = "security.ignore") //从nacos读取所有前缀为：security.ignore  
public class IgnoreWhiteProperties  
{  
    /**  
     * 放行白名单配置，网关不校验此处的白名单  
     */  
    private List<String> whites = new ArrayList<>();  
    public List<String> getWhites()  
    {  
        return whites;  
    }  
    public void setWhites(List<String> whites)  
    {  
        this.whites = whites;  
    }  
}
```
在nacos配置**白名单**
```yaml
security:
  ignore:
    whites:
      - /**/login
```

注意springboot自动装配了一个RedisAutoConfiguration，我们要加上一个注解使自己的bean优先级更高 **@AutoConfigureBefore(RedisAutoConfiguration.class)**
```java
/**  
* redis配置  
 */  
@Configuration  
@AutoConfigureBefore(RedisAutoConfiguration.class)// 优先配置RedisAutoConfiguration 
public class RedisConfig extends CachingConfigurerSupport {  
 //.......
}
```

然后记得配置网关在注册中心的配置jwt
```yaml
server:
  port: 19090
spring:
  cloud:
    gateway:
      routes:
        - id: oj-system
          uri: lb://oj-system
          predicates:
            - Path=/system/**
          filters:
            - StripPrefix=1
jwt:
  secret: sdfghuijasxdjkawskuigy
  
security:
  ignore:
    whites:
      - /**/login
```

注意全局异常处理器和我们网关微服务是不相容的，注意一些模块的依赖

3. 用户使用系统的过程中进行适当的延长jwt过期时间（防止用户在编码过程中过期）
**重新登陆后用户之前的key要清除掉，虽然说到期会自动清除，但是不好**
