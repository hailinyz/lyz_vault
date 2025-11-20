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

下面我们通过这个需求展开：将id在指定范围的用户（例如1、2、4）的余额扣减指定值

这是完全手写这个需求的SQL语句
```sql
<update id="updateBalanceByIds">
    UPDATE user
    SET balance = balance - #{amount}
    WHERE id IN
    <foreach collection="ids" separator="," item="id" open="(" close=")">
        #{id}
    </foreach>
</update>
```
我们可以利用MyBatisPlus的Wrapper来构建复杂的Where条件，然后自己定义SQL语句中剩下的部分。
![](assets/MP核心功能/file-20251120110748164.png)
 ① 基于Wrapper构建where条件
 ```java
 List<Long> ids = List.of(1L, 2L, 4L);
int amount = 200;
// 1.构建条件
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>().in(User::getId, ids);
// 2.自定义SQL方法调用
userMapper.updateBalanceByIds(wrapper, amount);
 ```
 ② 在 mapper 方法参数中用 Param 注解声明 wrapper 变量名称，必须是 ew
 ```java
 void updateBalanceByIds(@Param("ew") LambdaQueryWrapper<User> wrapper, @Param("amount") int amount);
 ```
③ 自定义 SQL，并使用 Wrapper 条件
```xml
<update id="updateBalanceByIds">
    UPDATE tb_user SET balance = balance - #{amount} ${ew.customSqlSegment}
</update>
```

## 3.Service接口

**Service提供的增删改查**
![](assets/MP核心功能/file-20251120111658281.png)
