
## 题目前端增删改

引入两个库

富文本编译器
还有代码编译器
```powershell
npm install @vueup/vue-quill@1.2.0

npm install ace-builds@1.4.13
```

还有抽屉
前端携带数据向后端发起请求
子组件要向父组件传递数据就要用到事件，代码里有，比如添加题目成功后就会立刻发起查询题目列表的请求就用到了子组件向父组件发消息

### 编辑题目
获取题目详情数据我们是在 Question.vue 这个组件下写的
但是展示题目详情数据的是在抽屉 QuestionDrawer.vue 这个子组件下

这就是获取数据在父组件，展示在子组件
**想办法将父组件获取到的数据放到子组件中去展示**

但是太麻烦了，直接让子组件发出请求自己显示就好了，得想办法将父组件获取的题目id传给子组件.

### 编辑题目
关于 async/await 的使用规则：
需要 async/await：当函数内部有异步操作（如 API 调用）且需要等待结果时

## 竞赛管理

**和竞赛相关的功能：**
B端：列表、新增、编辑、删除、发布、撤销发布
C端：列表（未开始、历史）、报名参赛、参加竞赛（竞赛倒计时、完成竞赛、竞赛内题目切换）、竞赛练习、查看排名、我的消息

建表
```sql
create table tb_exam (  
    exam_id bigint unsigned not null comment '竞赛id（主键）',  
    title varchar(50) not null comment '竞赛标题',  
    start_time datetime not null comment '竞赛开始时间',  
    end_time datetime not null comment '竞赛结束时间',  
    status tinyint not null default '0' comment '是否发布 0：未发布 1：已发布',  
    create_by bigint unsigned not null  comment '创建人',  
    create_time datetime not null comment '创建时间',  
    update_by bigint unsigned comment '更新人',  
    update_time datetime comment '更新时间',  
  
    primary key(exam_id)  
);  
  
-- 题目和竞赛关系表  
create table tb_exam_question (  
    exam_question_id bigint unsigned not null comment '竞赛题目关系id（主键）',  
    question_id bigint unsigned not null comment '题目id（主键）',  
    exam_id bigint unsigned not null comment '竞赛id（主键）',  
    question_order int not null comment '题目顺序',  
    create_by bigint unsigned not null  comment '创建人',  
    create_time datetime not null comment '创建时间',  
    update_by bigint unsigned comment '更新人',  
    update_time datetime comment '更新时间',  
  
    primary key(exam_question_id)  
);
```

**把这两个表对应的实体类创建出来**
![](assets/Day7/file-20251214150448287.png)
