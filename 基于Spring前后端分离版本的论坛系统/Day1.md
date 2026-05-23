# 技术选型
![](assets/Day1/file-20260521224715096.png)

# 数据库设计
```sql
# 创建数据库
create database forum_db character set utf8mb4 collate utf8mb4_general_ci;

# 创建表
# 用户表
create table t_user (  
    id bigint primary key auto_increment comment '编号，主键自增',  
    username varchar(20) not null unique comment '用户名，唯一',  
    password varchar(32) not null comment '加密后的密码',  
    nickname varchar(50) not null comment '昵称',  
    phoneNum varchar(20) comment '手机号',  
    email varchar(50) comment '电子邮箱',  
    gender tinyint not null default 2 comment '性别 0女，1男，2保密',  
    salt varchar(32) not null comment '为密码加盐',  
    avatarUrl varchar(255) comment '用户头像路径',  
    articleCount int not null default 0 comment '发帖数量',  
    isAdmin tinyint not null default 0 comment '是否管理员 0否，1是',  
    remark varchar(1000) comment '备注，自我介绍',  
    state tinyint not null default 0 comment '状态 0正常，1禁言',  
    deleteState tinyint not null default 0 comment '是否删除，0否，1是',  
    createTime datetime not null comment '创建时间，精确到秒',  
    updateTime datetime not null comment '更新时间，精确到秒'  
);

# 板块表
create table t_board (
    id bigint primary key auto_increment comment '编号，主键自增',
    name varchar(50) not null comment '版块名',
    articleCount int not null default 0 comment '帖子数量',
    sort int not null default 0 comment '排序优先级，升序',
    state tinyint not null default 0 comment '状态 0正常，1禁用',
    deleteState tinyint not null default 0 comment '是否删除，0否，1是',
    createTime datetime not null comment '创建时间，精确到秒',
    updateTime datetime not null comment '更新时间，精确到秒'
);

# 帖子表
create table t_article (
    id bigint primary key auto_increment comment '编号，主键自增',
    boardId bigint not null comment '帖子编号',
    userId bigint not null comment '发帖人，关联用户编号',
    title varchar(100) not null comment '帖子标题',
    content text not null comment '帖子正文',
    visitCount int not null default 0 comment '访问量',
    replyCount int not null default 0 comment '回复数',
    likeCount int not null default 0 comment '点赞数',
    state tinyint not null default 0 comment '状态 0正常，1禁用',
    deleteState tinyint not null default 0 comment '是否删除，0否，1是',
    createTime datetime not null comment '创建时间，精确到秒',
    updateTime datetime not null comment '更新时间，精确到秒'
);

# 帖子回复表
create table t_article_reply (
    id bigint primary key auto_increment comment '编号，主键自增',
    articleId bigint not null comment '关联帖子编号',
    postUserId bigint not null comment '楼主用户，关联用户编号',
    replyId bigint not null comment '关联回复编号，支持楼中楼',
    replyUserId bigint not null comment '楼主下的回复用户编号，支持楼',
    content varchar(500) not null comment '回贴内容',
    likeCount int not null comment '回贴内容',
    state tinyint not null default 0 comment '状态 0正常，1禁用',
    deleteState tinyint not null default 0 comment '是否删除，0否，1是',
    createTime datetime not null comment '创建时间，精确到秒',
    updateTime datetime not null comment '更新时间，精确到秒'
);

# 站内信
create table t_message (
    id bigint primary key auto_increment comment '编号，主键自增',
    postUserId bigint not null comment '发送者，关联用户编号',
    receiveUserId bigint not null comment '接收者，关联用户编号',
    content varchar(255) not null comment '内容',
    state tinyint not null default 0 comment '状态 0正常，1禁用',
    deleteState tinyint not null default 0 comment '是否删除，0否，1是',
    createTime datetime not null comment '创建时间，精确到秒',
    updateTime datetime not null comment '更新时间，精确到秒'
);

```

# 软件开发
## 搭建环境
