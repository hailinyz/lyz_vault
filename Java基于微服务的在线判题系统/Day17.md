### 我的消息功能

业务分析：
先登录，点击右上角小铃铛，有消息列表，竞赛结果通知，竞赛结束后，系统自动统计排名。

**站内信**：网站内部的通信方式。
1.用户和用户之间的通信（点对点）
2.管理员/系统   和   某个用户之间的通信（点对点）
3.管理员/系统   和   某个用户群（指的是满足某一条件的用户的群体）之间的通信。（点对面）

竞赛结果的通信消息（属于第二种点对点）


数据库表设计

消息内容表
```sql
create table tb_message_text(  
    text_id bigint unsigned NOT NULL COMMENT '消息内容id（主键）',  
    message_title varchar(10) NOT NULL COMMENT '消息标题',  
    message_content varchar(200) NOT NULL COMMENT '消息内容',  
    create_by bigint unsigned not null comment '创建人',  
    create_time datetime not null comment '创建时间',  
    update_by bigint unsigned comment '更新人',  
    update_time datetime comment '更新时间',  
    primary key (text_id)  
);
```




消息表