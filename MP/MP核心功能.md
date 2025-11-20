## 1.条件构造器
MP支持各种负责的where条件，可以满足日常开发的所有需求
![](assets/MP核心功能/file-20251120103730572.png)
==Wapper==就是条件构造器
![](assets/MP核心功能/file-20251120103836341.png)
**使用例子**
需求：更新id为1，2，4的用户的余额，扣200
```sql
UPDATE user 
	SET balance = balance - 200 
	WHERE id in (1, 2, 4)
```

```java
@Test
void testUpdateWrapper() {
    List<Long> ids = List.of(1L, 2L, 4L);
    UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
            .setSql("balance = balance - 200")
            .in("id", ids);
    userMapper.update(null, wrapper);
}
```
