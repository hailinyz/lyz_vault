## 个人中心 

### 查看用户详情 & 用户基本信息编辑

目前有个问题，就是C端用户登录/注册的时候，前端存不下token令牌，但是redis能存入token。
目前先这么将就着吧，可以先不通过网关。（理论上获取用户信息和用户详细信息都是要走到网关才行的）

###### !!!!!!!!!!!!!重磅，经过我的不断调试网关的过滤以及统一了nacos上的jwt配置
![](assets/Day13/file-20260109210634689.png)
注意，之前的nacos不统一，是这样的：
```cao
system：sdfghuijasxdjkawskuigy
friend：sdfghuijasxdjkawskuigysms
gateway：sdfghuijasxdjkawskuigy
job：sdfghuijasxdjkawskuigy
```

统一后才对，因为生成token和解析token的jwt应该是一样的才能正确解析出userId
```cao
system：sdfghuijasxdjkawskuigy
friend：sdfghuijasxdjkawskuigy
gateway：sdfghuijasxdjkawskuigy
job：sdfghuijasxdjkawskuigy
```

###### !!!!!!!!!!!!!!!!!!!!!!所以注意接口测试的时候切莫操之过急，每一步都不能忽略！！！！

核心问题：新用户注册时，ThreadLocalUtil 中没有 USER_ID（因为还没登录），但 MyMetaObjectHandler 会从 ThreadLocalUtil 获取 USER_ID 来填充 createBy 字段，结果获取到 null，导致数据库插入失败。

解决方案：在插入新用户之前，手动设置 createBy 字段。有两种方式：

方式 1（推荐）：手动设置 createBy 和 createTime
```java
if (user == null){ //新用户
    //注册逻辑
    user = new User();
    user.setPhone(phone);
    user.setStatus(UserStatus.Normal.getValue());
    // 新用户自注册，createBy设为0或-1表示系统创建
    user.setCreateBy(0L);
    user.setCreateTime(LocalDateTime.now());
    userMapper.insert(user);
}
```

--- 


#### 用户信息编辑的接口
```java
/*  
修改用户信息  
 */@Override  
public int edit(UserUpdateDTO userUpdateDTO) {  
    Long userId = ThreadLocalUtil.get(Constants.USER_ID, Long.class);  
    if (userId == null) {  
        throw new ServiceException(ResultCode.FAILED_USER_NOT_EXISTS);  
    }  
    User user = userMapper.selectById(userId);  
    if (user == null) {  
        throw new ServiceException(ResultCode.FAILED_USER_NOT_EXISTS);  
    }  
    user.setNickName(userUpdateDTO.getNickName());  
    user.setSex(userUpdateDTO.getSex());  
    user.setSchoolName(userUpdateDTO.getSchoolName());  
    user.setMajorName(userUpdateDTO.getMajorName());  
    user.setPhone(userUpdateDTO.getPhone());  
    user.setEmail(userUpdateDTO.getEmail());  
    user.setWechat(userUpdateDTO.getWechat());  
    user.setIntroduce(userUpdateDTO.getIntroduce());  
    ////更新用户缓存  
    userCacheManager.refreshUser(user); //用户详情的缓存  
    tokenService.refreshLoginUser(user.getNickName(), user.getHeadImage(), //刷新当前用户的登录信息  
            ThreadLocalUtil.get(Constants.USER_KEY, String.class));  
    return userMapper.updateById(user);  
}
```

更改的时候不仅仅是用户详情信息的缓存，注意还有当前登录用户的缓存。

vue中前后端交互的方法一般都写在js文件中
![](assets/Day13/file-20260109215441856.png)

然后views文件中调用，发起请求，别忘要引入。

###### 为了解决头像这个问题

要集成OSS到项目中,创建一个模块oj-common-file
```java
<dependency>  
    <groupId>com.aliyun.oss</groupId>  
    <artifactId>aliyun-sdk-oss</artifactId>  
    <version>${aliyun-sdk-oss}</version>  
</dependency>  
<dependency>  
    <groupId>javax.xml.bind</groupId>  
    <artifactId>jaxb-api</artifactId>  
    <version>${jaxb-api}</version>  
</dependency>  
<dependency>  
    <groupId>javax.activation</groupId>  
    <artifactId>activation</artifactId>  
    <version>${activation}</version>  
</dependency>  
<!-- no more than 2.3.3-->  
<dependency>  
    <groupId>org.glassfish.jaxb</groupId>  
    <artifactId>jaxb-runtime</artifactId>  
    <version>${jaxb-runtime}</version>  
</dependency>  
  
<dependency>  
    <groupId>com.bite</groupId>  
    <artifactId>oj-common-core</artifactId>  
    <version>${oj-common-core.version}</version>  
</dependency>  
<dependency>  
    <groupId>com.bite</groupId>  
    <artifactId>oj-common-redis</artifactId>  
    <version>${oj-common-redis.version}</version>  
</dependency>  
<dependency>  
    <groupId>com.bite</groupId>  
    <artifactId>oj-commin-security</artifactId>  
    <version>${oj-common-security.version}</version>  
</dependency>
```

配置凭证  &  nacos增加配置
```yaml
file:
  max-time: 3
  test: true
  oss:
    endpoint: oss-cn-beijing.aliyuncs.com
    region: cn-beijing
    accessKeyId: 你的accessKeyId
    accessKeySecret: f你的accessKeySecret
    bucketName: 你的bucketName
    pathPrefix: ojtest/
```

初始化 - 创建OSSClient实例 浇给spring容器处理（创建在config包下面）
```java
@Slf4j
@Configuration
public class OSSConfig {

    @Autowired
    private OSSProperties prop;

    public OSS ossClient;

    @Bean
    public OSS ossClient() throws ClientException {
        DefaultCredentialProvider credentialsProvider = CredentialsProviderFactory.newDefaultCredentialProvider(
                prop.getAccessKeyId(), prop.getAccessKeySecret());

        // 创建ClientBuilderConfiguration
        ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
        clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);

        // 使用内网endpoint进行上传
        ossClient = OSSClientBuilder.create()
                .endpoint(prop.getEndpoint())
                .credentialsProvider(credentialsProvider)
                .clientConfiguration(clientBuilderConfiguration)
                .region(prop.getRegion())
                .build();
        return ossClient;
    }

    @PreDestroy
    public void closeOSSClient() {
        ossClient.shutdown();
    }
}
```

```java
@Data  
@Component  
@ConfigurationProperties(prefix = "file.oss")  
public class OSSProperties {  
  
    private String endpoint;  
  
    private String region;  
  
    private String accessKeyId;  
  
    private String accessKeySecret;  
  
    private String bucketName;  
  
    /**  
     * 路径前缀，加在 endPoint 之后  
     */  
    private String pathPrefix;  //ojtest  
}
```

上传文件（简单上传 - 上传文件流）
跟短信服务一样，抽象出一个Service封装好，以后哪个服务用直接调用就行了。
```java
@Slf4j  
@Service  
@RefreshScope  
public class OSSService {  
  
    @Autowired  
    private OSSProperties prop;  
  
    @Autowired  
    private OSSClient ossClient;  
  
    @Autowired  
    private RedisService redisService;  
  
    @Value("${file.max-time}")  
    private int maxTime;  
  
    @Value("${file.test}")  
    private boolean test;  
  
    public OSSResult uploadFile(MultipartFile file) throws Exception {  
        if (!test) {  
            checkUploadCount();  
        }  
        InputStream inputStream = null;  
        try {  
            String fileName;  
            if (file.getOriginalFilename() != null) {  
                fileName = file.getOriginalFilename().toLowerCase();  
            } else {  
                fileName = "a.png";  
            }  
            String extName = fileName.substring(fileName.lastIndexOf(".") + 1);  
            inputStream = file.getInputStream();  
            return upload(extName, inputStream);  
        } catch (Exception e) {  
            log.error("OSS upload file error", e);  
            throw new ServiceException(ResultCode.FAILED_FILE_UPLOAD);  
        } finally {  
            if (inputStream != null) {  
                inputStream.close();  
            }  
        }  
    }  
  
    private void checkUploadCount() {  
        Long userId = ThreadLocalUtil.get(Constants.USER_ID, Long.class);  
        Long times = redisService.getCacheMapValue(CacheConstants.USER_UPLOAD_TIMES_KEY, String.valueOf(userId), Long.class);  
        if (times != null && times >= maxTime) {  
            throw new ServiceException(ResultCode.FAILED_FILE_UPLOAD_TIME_LIMIT);  
        }  
        redisService.incrementHashValue(CacheConstants.USER_UPLOAD_TIMES_KEY, String.valueOf(userId), 1);  
        if (times == null || times == 0) {  
            long seconds = ChronoUnit.SECONDS.between(LocalDateTime.now(),  
                    LocalDateTime.now().plusDays(1).withHour(0).withMinute(0).withSecond(0).withNano(0));  
            redisService.expire(CacheConstants.USER_UPLOAD_TIMES_KEY, seconds, TimeUnit.SECONDS);  
        }  
    }  
  
    private OSSResult upload(String fileType, InputStream inputStream) {  
        // key pattern: file/id.xxx, cannot start with /  
        String key = prop.getPathPrefix() + ObjectId.next() + "." + fileType;  
        ObjectMetadata objectMetadata = new ObjectMetadata();  
        objectMetadata.setObjectAcl(CannedAccessControlList.PublicRead);  
        PutObjectRequest request = new PutObjectRequest(prop.getBucketName(), key, inputStream, objectMetadata);  
        PutObjectResult putObjectResult;  
        try {  
            putObjectResult = ossClient.putObject(request);  
        } catch (Exception e) {  
            log.error("OSS put object error: {}", ExceptionUtil.stacktraceToOneLineString(e, 500));  
            throw new ServiceException(ResultCode.FAILED_FILE_UPLOAD);  
        }  
        return assembleOSSResult(key, putObjectResult);  
    }  
  
    private OSSResult assembleOSSResult(String key, PutObjectResult putObjectResult) {  
        OSSResult ossResult = new OSSResult();  
        if (putObjectResult == null || StrUtil.isBlank(putObjectResult.getRequestId())) {  
            ossResult.setSuccess(false);  
        } else {  
            ossResult.setSuccess(true);  
            ossResult.setName(FileUtil.getName(key));  
        }  
        return ossResult;  
    }  
}
```

一个**接口上传文件**，一个接口存，上传完成之后要返回唯一标识给 前端 再根据这个唯一标识再去请求后端。（解决超时问题）

存储头像不是最终目的，能让用户查询展示头像才是
查的时候要有**对应关系**  需要文件的唯一标识  -- > 取文件名就行了

用户表：headImage   oss的唯一标识（文件名）存储到数据库的headImage
