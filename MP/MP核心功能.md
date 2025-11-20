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
**使用Lambda的形式，用对应的get函数避免字符串硬编码**
```java
@Test
void testLambdaQueryWrapper() {
    // 1.构建查询条件
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
            .select(User::getId, User::getUsername, User::getInfo, User::getBalance)
            .like(User::getUsername, "o")
            .ge(User::getBalance, 1000);
    // 2.查询
    List<User> users = userMapper.selectList(wrapper);
    users.forEach(System.out::println);
}
```
**条件构造器的用法小结**
 - QueryWrapper和LambdaQueryWrapper通常用来构建select、delete、update的where条件部分 
 - UpdateWrapper和LambdaUpdateWrapper通常只有在set语句比较特殊才使用
 - 尽量使用LambdaQueryWrapper和LambdaUpdateWrapper，避免硬编码

## 2.自定义SQL

**我们可以利用MyBatisPlus的Wrapper来构建复杂的Where条件，然后自己定义SQL语句中剩下的部分。**

