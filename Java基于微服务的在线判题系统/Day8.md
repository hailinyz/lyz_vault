## 竞赛删除

竞赛删除 = 删除竞赛基本信息（tb_exam） + 删除竞赛题目信息（tb_exam_question）
1. 找到要删除的竞赛，点击删除按钮。前端携带竞赛 id 向后端发起删除竞赛请求
2. 后端接收请求后，根据竞赛 id （先判断竞赛是否存在，竞赛是否开赛）删除竞赛基本信息 和 竞赛题目信息。并且返回删除结果
3. 前端接收到响应，如果成功/失败

```java
/*  
删除竞赛  
 */@Override  
public int delete(Long examId) {  
    //判断竞赛是否存在  
    Exam exam = getExam(examId);  
    //判断竞赛是否开始  
    checkExam(exam);  
    //删除竞赛中的题目  
    examQuestionMapper.delete(new LambdaQueryWrapper<ExamQuestion>()  
            .eq(ExamQuestion::getExamId, examId));  
    //删除竞赛  
    return examMapper.deleteById(exam);  
}
```

## 竞赛发布

竞赛发布的前提：竞赛存在、竞赛中有题目

有三个地方可以进行竞赛发布操作
A添加竞赛
1. 在满足发布竞赛前提后，点击发布竞赛。前端携带竞赛 id 向后端发起请求
2. 后端接收到请求后，根据竞赛 id 判断前提是否成立。
   如果不成立：将原因发给前端
   如果成立：将竞赛状态字段从0改为1，并且同步到数据库，将修改结果发给前端
3. 前端接收到后端响应之后
   如果失败，展示失败原因
   如果成功，跳转回列表页，当前状态变回已发布
   C端竞赛列表中要能找到已发布竞赛（可以将状态字段为0的竞赛过滤掉，后面可能会传参数给接口）

B 编辑竞赛
C竞赛列表当中

```java
/*  
发布竞赛  
 */@Override  
public int publish(Long examId) {  
    //判断竞赛是否存在  
    Exam exam = getExam(examId);  
    //判断竞赛中是否有题目 select count(*) from tb_exam_question where exam_id = #{examId}    Long count = examQuestionMapper.selectCount(new LambdaQueryWrapper<ExamQuestion>()  
            .eq(ExamQuestion::getExamId, examId));  
    if (count == null || count <= 0){  
        throw new ServiceException(ResultCode.EXAM_NOT_HAS_QUESTION);  
    }  
    //改变状态并同步到数据库  
    exam.setStatus(Constants.TRUE);  
    return examMapper.updateById(exam);  
}
```

## 竞赛撤销

竞赛撤销发布的前提：竞赛存在、竞赛还未开始

1. 找到要发布的竞赛，点击撤销发布按钮。前端携带竞赛 id 向后端发起请求
2. 后端接收到请求后，根据竞赛 id 判断前提条件是否成立
   如果不成立：返回前端不成立原因
   如果成立： 撤销发布对应的竞赛（将竞赛状态从1 设置 为 0，并同步数据库 ），将结果返回前端
3. 前端接收到响应后
   如果成功：展示失败原因
   如果失败：B端竞赛列表中当前竞赛状态变为未发布
		   当前竞赛C端竞赛列表中消失
		   
```java
/*  
撤销发布  
 */@Override  
public int cancelpublish(Long examId) {  
    //判断竞赛是否存在  
    Exam exam = getExam(examId);  
    //判断竞赛是否开始  
    checkExam(exam);  
    exam.setStatus(Constants.FALSE);  
    return examMapper.updateById(exam);  
}
```

![](assets/Day8/file-20251216154036734.png)
B端功能全部功能如图此前的这几个已经开发完毕，现在继续开发
是哪个用户提交的

## 用户管理

大概先列出这些

B端：列表、拉黑
C端：登录、注册、退出登录、个人中心

B端用密码登录、C端用验证码登录

### 建表
拉黑功能用AOP（面向切面）实现，咱有用户状态这个字段
```sql
create table tb_user(  
    user_id bigint unsigned NOT NULL COMMENT '用户id（主键）',  
    nick_name varchar(20) comment '用户昵称',  
    head_image varchar(100) comment '用户头像',  
    sex tinyint comment '用户状态1: 男 2: 女',  
    phone char(11) not null comment '手机号',  
    code char(6) comment '验证码',  
    email varchar(20) comment '邮箱',  
    wechat varchar(20) comment '微信号',  
    -- AAAAA varchar(50) comment '学校/专业', 学校 &专业  
    school_name varchar(20) comment '学校',  
    major_name varchar(20) comment '专业',  
    introduce varchar(100) comment '个人介绍',  
    status tinyint not null comment '用户状态0: 拉黑 1: 正常',  
    create_by  bigint unsigned not null  comment '创建人',  
    create_time datetime not null comment '创建时间',  
    update_by  bigint unsigned comment '更新人',  
    update_time datetime comment '更新时间',  
    primary key(`user_id`)  
)
```

创建表对应的实体类User
```java
@Getter  
@Setter  
@TableName("tb_user")  
public class User extends BaseEntity {  
  
    @JsonSerialize(using = ToStringSerializer.class)  
    @TableId(value = "USER_ID", type = IdType.ASSIGN_ID)  
    private Long userId;  
  
    private String nickName;  
  
    private String headImage;  
  
    private Integer sex;  
  
    private String phone;  
  
    private String code;  
  
    private String email;  
  
    private String wechat;  
  
    private String schoolName;  
  
    private String majorName;  
  
    private String introduce;  
  
    private Integer status;  
}
```

然后创建user相关的包和类，把框架搭起来
![](assets/Day8/file-20251216172826346.png)

1. 前端携带响应参数向后端发起 **用户列表** 请求
2. 后端接收到请求后，将参数转换为查询条件，从数据中查出符合查询条件的数据。并且将查询结果返回给前端。（成功/失败）
3. 前端接收到后端响应
   如果成功：页面展示用户的数据
   如果失败：提示失败原因

**获取用户列表接口**
```java
/*  
获取用户列表  
 */@Override  
public List<UserVO> list(UserQueryDTO userQueryDTO) {  
    PageHelper.startPage(userQueryDTO.getPageNum(),userQueryDTO.getPageSize());  
    return userMapper.selectUserList(userQueryDTO);  
}
```

**修改用户状态（用户拉黑/解禁功能）**

1. 前端携带参数（userId、status）向后端发起修改用户状态请求
2. 后端收到请求之后，从数据库中查询出要修改状态的用户，修改状态，并将修改后的状态同步到数据库中。再向前端返回更新结果（成功/失败）
3. 前端接收后端响应。
   如果失败：用户状态保持不变，提示失败原因
   如果成功：如果拉黑用户，限制用户的一些操作
             如果是解禁，放开之前限制

为了安全着想我们的参数可以用@RequestBody ，所以下面代码中的userId、status最好定义在一个DTO里。
```java
/*  
* 修改用户状态  
 */@PutMapping("updateStatus")  
public R<Void> updateStatus(@RequestBody Long userId, Integer status){  
    return userService.toR(userService.updateStatus(userId,status));  
}
```
应该是这样
```java
@Getter  
@Setter  
public class UserDTO {  
  
    private Long userId;  
  
    private Integer status;  
  
}

/*  
* 修改用户状态  
 */@PutMapping("updateStatus")  
public R<Void> updateStatus(@RequestBody UserDTO userDTO){  
    return toR(userService.updateStatus(userDTO));  
}
```

接口代码
```java
/*  
修改用户状态  
 */@Override  
public int updateStatus(UserDTO userDTO) {  
    User user = userMapper.selectById(userDTO.getUserId());  
    if (user == null){  
        throw new ServiceException(ResultCode.FAILED_USER_NOT_EXISTS);  
    }  
    user.setStatus(userDTO.getStatus());  
    return userMapper.updateById(user);  
}
```
好的，到这里修改用户状态完成，就差**限制/放开**用户的一些功能了
```java
// todo 拉黑/接近：限制/放开用户操作票
```

进行接口测试的时候注意：
![](assets/Day8/file-20251216190344222.png)
传的是这个，根据DTO传的
```java
{
    "userId": 1796100478535168002,
    "status": 1
}
```

还需要注意
![](assets/Day8/file-20251216191918450.png)
![](assets/Day8/file-20251216191932288.png)
### 错了错了，返回给前端的是VO，所以这个注解应该在UserVO上加


![](assets/Day8/file-20251216195212189.png)
好的以上的B端功能都已经差不多了

### 现在开始开发C端
![](assets/Day8/file-20251216195238875.png)

C端登录注册功能：
A老用户
1. 输入手机号，点击获取验证码按钮。前端向后端发起获取验证码请求。
2. 后端接收到请求后，随机生成验证码，并将验证码发送到用户的手机上。



3. 用户收到验证码后，输入收到的验证码，点击登录/注册按钮，发起登录/注册请求。
4. 后端接收到请求后，根据用户手机号判断是新/老用户，如果是老用户，直接执行登录逻辑，并将登录成功/失败返回前端
5. 前端接收到响应后，
   如果成功：跳转到系统首页（并在右上角展示用户的头像和昵称）
   如果失败：停留在当前登录页，提示用户失败原因。

既然前端发起了两次请求，分析出有两个接口
![](assets/Day8/file-20251216201857669.png)
之前开发B端都是在system微服务下，现在C端应该在用户端（friend微服务）

**获取验证码接口**
```java
/*  
获取验证码  
 */@Override  
public void sendCode(UserDTO userDTO) {  
    //先判断传过来的是不是一个真的手机号  
    if (!checkPhone(userDTO.getPhone())){  
        throw new ServiceException(ResultCode.FAILED_USER_PHONE);  
    }  
    //生成验证码  
    String code = RandomUtil.randomNumbers(6);  
  
}
```
这里使用糊涂的生成验证码之后，怎么将验证码发送给用户手机，借助第三方**阿里云短信服务**将验证码发送到用户手机上

**集成阿里云短信服务**
创建AcessKey并保存好ID 和 Secret
![](assets/Day8/file-20251216234202752.png)
搜索短信服务自行购买下单即可
截至我写这篇文章之前，短信服务已经**不支持个人资质**，所以我只能用**号码认证**了

创建oj-common-message短信微服务
引入依赖
```xml
<!-- 阿里云号码认证 -->  
<dependencies>  
    <dependency>  
        <groupId>com.aliyun</groupId>  
        <artifactId>dypnsapi20170525</artifactId>  
        <version>2.0.0</version>  
    </dependency>  
    
    <dependency>
	  <groupId>com.aliyun</groupId>
	  <artifactId>dysmsapi20170525</artifactId>
	  <version>4.3.1</version>
	</dependency>

    <dependency>  
        <groupId>com.alibaba.fastjson2</groupId>  
        <artifactId>fastjson2</artifactId>  
        <version>${fastjson.version}</version>  
    </dependency>  
</dependencies>


```


浇给sqpring容器处理 
配置类、所看到的参数用nacos进行管理，用@Value注解






B新用户
