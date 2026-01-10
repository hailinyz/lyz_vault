
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


