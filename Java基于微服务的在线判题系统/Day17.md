### 我的消息功能

业务分析：
先登录，点击右上角小铃铛，有消息列表，竞赛结果通知，竞赛结束后，系统自动统计排名。

**站内信**：网站内部的通信方式。
1.用户和用户之间的通信（点对点）
2.管理员/系统   和   某个用户之间的通信（点对点）
3.管理员/系统   和   某个用户群（指的是满足某一条件的用户的群体）之间的通信。（点对面）

竞赛结果的通信消息（属于第二种点对点）


数据库表设计：
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
```sql
create table tb_message(  
    message_id bigint unsigned NOT NULL COMMENT '消息id（主键）', 
    text_id bigint unsigned NOT NULL COMMENT '消息内容id（主键）',  
    send_id bigint unsigned NOT NULL COMMENT '消息发送人id',  
    rec_id bigint unsigned NOT NULL COMMENT '消息接收人id',  
    create_by bigint unsigned not null comment '创建人',  
    create_time datetime not null comment '创建时间',  
    update_by bigint unsigned comment '更新人',  
    update_time datetime comment '更新时间',  
    primary key (message_id)  
);
```


消息发送：就是将产生的消息存入数据库和缓存就**OK**
1. 消息是如何产生的（竞赛结果通知消息）
每天凌晨，会对前一天结束竞赛进行用户排名统计。消息在统计过程中随即产生了。
竞赛结束时间不能超过晚上 10:00 凌晨统计可以一次统计完所有竞赛（xxl-job 每天1:00）。

2. 消息如何发送
只需要将产生的消息存储到数据库中 + redis
设计redis当中的缓存
![](assets/Day17/file-20260116000024593.png)

---
好的上面分析完成。
下面开始开发。

#### 我的消息
1. 消息的发送
通过**定时任务**生成消息 --> 将消息存储到**数据库**和**缓存**当中
我们现在做个定时任务**生成消息**
```java
    /*  
    * 生成排名消息  
     */    private void createMessage(List<Exam> examList, Map<Long, List<UserScore>> userScoreMap) {  
        List<MessageText> messageTextList = new ArrayList<>(); //消息详情信息  
        List<Message> messageList = new ArrayList<>(); //消息信息  
        for (Exam exam : examList) {  
            Long examId = exam.getExamId();  
            List<UserScore> userScoreList = userScoreMap.get(examId);  
            int totalUser = userScoreList.size();  
            int examRank = 1;  
            for (UserScore userScore : userScoreList) {  
                String msgTitle = exam.getTitle() + "——排名情况";  
                String msgContent = "您所参与的竞赛：" + exam.getTitle()  
                        + "，本次参与竞赛一共" + totalUser + "人，您排名第" + examRank + "名。";  
                MessageText messageText = new MessageText();  
                messageText.setMessageTitle(msgTitle);  
                messageText.setMessageContent(msgContent);  
                messageText.setCreateBy(Constants.SYSTEM_USER_ID);  
//                messageTextMapper.insert(messageText); //插入数据库，循环插入慢  
                messageTextList.add(messageText);  
                Message message = new Message();  
                message.setSendId(Constants.SYSTEM_USER_ID);  
                message.setCreateBy(Constants.SYSTEM_USER_ID);  
                message.setRecId(userScore.getUserId());  
                examRank++;  
            }  
        }  
        messageTextService.batchInsert(messageTextList); //批量插入  
        Map<String, MessageTextVO> messageTextVOMap = new HashMap<>();  
        for (int i = 0; i < messageTextList.size(); i++) {  
            MessageText messageText = messageTextList.get(i);  
            MessageTextVO messageTextVO = new MessageTextVO();  
            BeanUtil.copyProperties(messageText, messageTextVO);  
            String msgDetailKey = getMsgDetailKey(messageText.getTextId());  
            messageTextVOMap.put(msgDetailKey, messageTextVO); //为了批量缓存，这里现存入map中  
            Message message = messageList.get(i);  
            message.setTextId(messageText.getTextId());  
        }  
        messageService.batchInsert(messageList); //批量插入  
        //分组 划分成每个用户对应的消息列表  
        Map<Long, List<Message>> userMsgMap = messageList.stream().collect(Collectors.groupingBy(Message::getRecId));  
        Iterator<Map.Entry<Long, List<Message>>> iterator = userMsgMap.entrySet().iterator();  
        while (iterator.hasNext()) { //通过迭代器遍历  
            Map.Entry<Long, List<Message>> entry = iterator.next();  
            Long recId = entry.getKey(); //用户id  
            String userMsgListKey = getUserMsgListKey(recId); //缓存key  
            List<Long> userMsgTextIdList = entry.getValue().stream().map(Message::getTextId).collect(Collectors.toList()); //缓存Value 获取用户消息的textId  
            redisService.rightPushAll(userMsgListKey, userMsgTextIdList);  
        }  
        redisService.multiSet(messageTextVOMap);  
  
    }
```

2. 获取当前用户消息列表（消息的展示）
我的消息列表，模仿竞赛那块的代码就行了。
```java
/*  
* 获取当前用户消息列表(消息的展示)  
 */@Override  
public TableDataInfo list(PageQueryDTO dto) {  
    //从ThreadLocal中获取用户id  
    Long userId = ThreadLocalUtil.get(Constants.USER_ID, Long.class);  
  
    //从redis中获取 竞赛列表数据  
    Long total = messageCacheManager.getListSize(userId);  
    List<MessageTextVO> messageTextVOList;  
    if (total == null || total <= 0){  
        //从数据库中获取 我的竞赛列表数据  
        PageHelper.startPage(dto.getPageNum(),dto.getPageSize());  
        messageTextVOList = messageTextMapper.selectUserMsgList(userId);  
        //同步到redis中  
        messageCacheManager.refreshCache(userId);  
        total = new PageInfo<>(messageTextVOList).getTotal(); //获取总记录数  
    } else {  
        //从redis中获取 竞赛列表数据  
        messageTextVOList = messageCacheManager.getMsgTextVOList(dto, userId);  
    }  
    if (CollectionUtil.isEmpty(messageTextVOList)){ //使用hutool工具包判断集合是否为空  
        return TableDataInfo.empty(); //未查出任何数据时调用  
    }  
    return TableDataInfo.success(messageTextVOList, total);  
}
```

执行定时任务生成消息的时候注意修改一下数据库的字符集，不然定时任务会执行失败，造成数据插入数据库失败。
```sql
ALTER TABLE tb_message_text CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;  
  
ALTER TABLE tb_message_text MODIFY COLUMN message_title VARCHAR(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;  
  
ALTER TABLE tb_message_text MODIFY COLUMN message_content TEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

//TODO删除我的消息功能还没做
思路：将数据库数据和缓存数据删除即可，操作的表是**tb_message_text** 和 **tb_message**

删除应该怎么删除呢？根据什么删除？
```java
/*  
 * 删除当前用户的消息（硬删除）  
 */@Override  
@Transactional(rollbackFor = Exception.class) //添加事务注解  
public void delete(Long textId) {  
    // 1. 获取当前用户ID  
    Long userId = ThreadLocalUtil.get(Constants.USER_ID, Long.class);  
    log.info("用户 {} 删除消息，textId: {}", userId, textId);  
  
    // 2. 删除数据库中的消息关联记录（物理删除）  
    int deleteCount = messageMapper.delete(new LambdaQueryWrapper<Message>()  
            .eq(Message::getRecId, userId)  
            .eq(Message::getTextId, textId));  
      
    if (deleteCount == 0) {  
        log.warn("消息不存在或已被删除，userId: {}, textId: {}", userId, textId);  
        throw new RuntimeException("消息不存在");  
    }  
      
    log.info("删除数据库记录数: {}", deleteCount);  
  
    // 3. 检查是否还有其他用户关联这条消息  
    Long count = messageMapper.selectCount(new LambdaQueryWrapper<Message>()  
            .eq(Message::getTextId, textId));  
      
    // 4. 如果没有其他用户关联，删除消息内容  
    if (count == 0) {  
        messageTextMapper.deleteById(textId);  
        log.info("消息内容已删除，textId: {}", textId);  
    } else {  
        log.info("消息内容保留，还有 {} 个用户关联", count);  
    }  
  
    // 5. 删除 Redis 缓存  
    // 5.1 从用户消息列表中移除该消息  
    String userMsgListKey = CacheConstants.USER_MESSAGE_LIST + userId;  
    redisService.listRemove(userMsgListKey, textId);  
      
    // 5.2 如果没有其他用户关联，删除消息详情缓存  
    if (count == 0) {  
        String msgDetailKey = CacheConstants.MESSAGE_DETAIL + textId;  
        redisService.deleteObject(msgDetailKey);  
    }  
      
    log.info("Redis 缓存清理完成");  
}
```


