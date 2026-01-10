
## 用户答题功能

表设计(用户提交表)
```sql
CREATE TABLE tb_user_submit (
    submit_id BIGINT UNSIGNED NOT NULL COMMENT '提交记录id',
    user_id BIGINT UNSIGNED NOT NULL COMMENT '用户id',
    question_id BIGINT UNSIGNED NOT NULL COMMENT '题目id',
    exam_id BIGINT UNSIGNED COMMENT '竞赛id',
    program_type TINYINT NOT NULL COMMENT '代码类型 0 java 1 CPP',
    user_code TEXT NOT NULL COMMENT '用户代码',
    pass TINYINT NOT NULL COMMENT '0:未通过 1:通过',
    exe_message VARCHAR(500) NOT NULL COMMENT '执行结果',
    score INT NOT NULL DEFAULT '0' COMMENT '得分',
    create_by BIGINT UNSIGNED NOT NULL COMMENT '创建人',
    create_time DATETIME NOT NULL COMMENT '创建时间',
    update_by BIGINT UNSIGNED COMMENT '更新人',
    update_time DATETIME COMMENT '更新时间',
    PRIMARY KEY (`submit_id`)
);
```

点击开始答题之后向后端提交**获取题目详情请求**
如果ES查不到，再去数据库查，然后同步给ES
```java
/*  
获取题目详情  
 */@Override  
public QuestionDetailVO detail(Long questionId) {  
  
    // 从ES中查询  
    QuestionES questionES = questionRespository.findById(questionId).orElse(null); //获取的是options字段,所以需要.orElse(null)  
    QuestionDetailVO questionDetailVO = new QuestionDetailVO();  
    if (questionES != null){  
        BeanUtils.copyProperties(questionES,questionDetailVO);  
        return questionDetailVO;  
    }  
  
    // 从数据库中查询  
    Question question = questionMapper.selectById(questionId);  
    if (question == null){  
        return null;  
    }  
    refreshQuestion(); //将数据库中的数据同步到ES中  
    BeanUtils.copyProperties(question,questionDetailVO);  
    return questionDetailVO;  
}
```

然后是前端，因为有可以编写代码的编译器区域，可以跟之前B端的一样，引入一个库：
![](assets/Day14/file-20260110162316571.png)
##### ace-builds
![](assets/Day14/file-20260110161233166.png)
```powershell
npm install ace-builds@1.4.13
```

和B端同样的方式，将参数questionId传过去
然后另一个页面就可以了
```vue
function goQuestTest(questionId) {
  router.push(`/c-oj/anwser?questionId=${questionId}`)
}
```

然后在Answer.vue就能拿到questionId，就能向后端发起请求了
![](assets/Day14/file-20260110165933652.png)

和B端一样，获取到默认代码块之后放到aceCode里面
![](assets/Day14/file-20260110171924750.png)


#### 获取上一题&下一题

获取上一题  
题目的顺序列表  当前题目是哪个(questionId)  
redis  list数据类型(顺序,已经排好序了) key:  q:l   value:  questionId

需要把顺序存到**redis**里面，可以先从中获得顺序列表，然后查出来当前题目所在位置，
然后上一题、下一题也就清楚了
