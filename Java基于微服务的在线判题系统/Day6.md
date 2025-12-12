## 题库管理开发

![](assets/Day6/file-20251211094334988.png)

题目数据 --> 数据库MySQL --> **设计表结构**
+ 满足需求，避免冗余设计，考虑今后发展

建表sql
```sql
create table tb_question(  
    question_id bigint unsigned not null comment '题目id',  
    title varchar(50) not null comment '题目标题',  
    difficulty tinyint not null comment '题目难度:1:简单 2:中等 3:困难',  
    time_limit int not null comment '时间限制',  
    space_limit int not null comment '空间限制',  
    content varchar(1000) not null comment '题目内容',  
    question_case varchar(1000) comment '题目用例',  
    default_code varchar(500) not null comment '默认代码',  
    main_func varchar(500) not null comment 'main函数',  
    create_by bigint unsigned not null comment '创建人',  
    create_time datetime not null comment '创建时间',  
    update_by bigint unsigned comment '更新人',  
    update_time datetime comment '更新时间',  
    primary key(`question_id`)  
);
```

创建和表相互应的实体类
```java
@TableName("tb_question")  
@Getter  
@Setter  
public class Question extends BaseEntity {  
  
    @TableId(type = IdType.ASSIGN_ID) // 主键 雪花算法  
    private Long questionId;  
  
    private String title;  
  
    private Integer difficulty;  
  
    private Long timelimit;  
  
    private Long spacelimit;  
  
    private String content;  
  
    private String questionCase;  
  
    private String defaultCode;  
  
    private String mainFunc;  
}
```

## 列表功能

### 后端开发
定义一些实体类、VO

**分页PageHelper**
```xml
引入依赖
dependency>
 <groupId>com.github.pagehelper</groupId>
 <artifactId>pagehelper-spring-boot-starter</artifactId>
 <version>${pagehelper.boot.version}</version>
</dependency>
<pagehelper.boot.version>2.0.0</pagehelper.boot.version>
```
只需把页码和记录数提供给它，就会拼装在普通sql后完成分页效果
底层已经直接拿到总记录数了，直接想办法取出来就行了
![](assets/Day6/file-20251211230735876.png)



### 列表功能各文件代码:

Controller
```java
@RestController  
@RequestMapping("/question")  
@Tag(name = "题目管理接口")  
public class QuestionController extends BaseController {  
  
    @Autowired  
    private IQuestionService questionService;  
  
    /*  
    * 获取题目列表接口  
     */    @GetMapping("/list")  
    public TableDataInfo list(QuestionQueryDTO questionQueryDTO){  
        return getTableDataInfo(questionService.list(questionQueryDTO));  
    }  
  
}
```

QuestionQueryDTO
```java
@Getter  
@Setter  
public class QuestionQueryDTO extends PageQueryDTO {  
  
    private Integer difficulty;  
  
    private String title;  
      
}
```

PageQueryDTO
```java
@Getter  
@Setter  
public class PageQueryDTO {  
      
    private Integer pageSize = 10; // 每页显示的条数  
  
    private Integer pageNum = 1; // 当前页码  
    }
```

接口IQuestionService
```java
public interface IQuestionService {  
  
    /*  
    * 获取题目列表接口  
     */    List<QuestionVO> list(QuestionQueryDTO questionQueryDTO);  
  
}
```

实现类
```java
@Service  
public class QuestionServiceImpl implements IQuestionService {  
  
    @Autowired  
    private QuestionMapper questionMapper;  
  
    /*  
    获取题目列表接口  
     */    @Override  
    public List<QuestionVO> list(QuestionQueryDTO questionQueryDTO) {  
        PageHelper.startPage(questionQueryDTO.getPageNum(),questionQueryDTO.getPageSize());  
        return questionMapper.selectQuestionList(questionQueryDTO);  
    }  
  
  
}
```
OK了，现在应该没啥问题了，后端应该能拿到前端传过来的参数

### 前端开发

1. 根据用户需要携带请求参数向后端获取题目列表请求（请求头携带token）
2. 前端接收到后端的响应数据，根据响应结果，将题目列表展示在页面
					如果失败，向用户提示错误信息，页面保持原样


## 题目新增功能

### 思路
1. 登录成功后才能新增，切换至题目管理
2. 点击添加题目按钮，弹出抽屉
3. 根据所要添加的题目，输入题目相关信息，点击发布
4. 携带相关参数（信息）向后端发起添加题目请求
5. 后端接收到请求，对请求参数处理
6. 将数据存储到数据库中
7. 后端需要将题目添加结果返回前端
8. 前端接收到后端相应，
   如果成功：提示用户添加成功，并且将新题目展示在第一个
   如果失败：失败原因提示给用户
   