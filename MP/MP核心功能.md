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

## 3.IService接口

**Service提供的增删改查**
![](assets/MP核心功能/file-20251120111658281.png)

①我们Service接口需要去继承他的Service接口
②我们的实现类需要去继承他的实现类
![700](assets/MP核心功能/file-20251120112314564.png)
**实际上对于一些简单的增删改查，我们都可以直接在Controller当中调MP里面提供的方法，无需写任何的自定义Service或者Mapper，非常的方便；
只有在我们的业务逻辑相对复杂，需要自己写一些业务，MP只有增删改查没有业务，因此在这种情境下我们就需要去自定义Service方法，并且在里边编写我们的业务逻辑，当我们的BaseMapper不足以满足，就要自定义Mapper

## 4.IService的Lambda查询

**未使用Lambda之前写的SQL,这种复杂查询就可以用到Lambda**
```xml
<select id="queryUsers" resultType="com.itheima.mp.domain.po.User">
    SELECT *
    FROM tb_user
    <where>
        <if test="name != null">
            AND username LIKE #{name}
        </if>
        <if test="status != null">
            AND `status` = #{status}
        </if>
        <if test="minBalance != null and maxBalance != null">
            AND balance BETWEEN #{minBalance} AND #{maxBalance}
        </if>
    </where>
</select>
```
**用了之后**
```java
@Override  
public List<User> queryUsers(String name, Integer status, Integer minBalance, Integer maxBalance) {  
    return lambdaQuery()  
            .like(name != null, User::getUsername, name)  
            .eq(status != null, User::getStatus, status)  
            .ge(minBalance != null, User::getBalance, minBalance)  
            .le(maxBalance != null, User::getBalance, maxBalance)  
            .list();  
}
```
对比对比是不是==超级方便==
**但是在之前的扣减余额可能会有并发安全问题，这时候可以提升为事物，加锁**
```java
@Override  
@Transactional  
public void deductBalance(Long id, Integer money) {  
    //1.查询用户  
    User user = getById(id);  
    //2.校验用户状态  
    if (user == null || user.getStatus() == 2) {  
        throw new RuntimeException("用户状态异常！");  
    }  
    //3.校验余额是否充足  
    if (user.getBalance() < money) {  
        throw new RuntimeException("用户余额不足！");  
    }  
    //4.扣减余额 update tb_user set balance = balance - ?    int remainBalance = user.getBalance() - money;  
    lambdaUpdate()  
            .set(User::getBalance, remainBalance)  
            .set(remainBalance == 0, User::getStatus, 2)  
            .eq(User::getId, id)  
            .eq(User::getBalance,user.getBalance()) // 乐观锁  
            .update();  
}
```

## 5.IService的批量新增

**方式一：首先是一个一个新增,耗时将近20万ms**
```java
@Test
void testSaveOneByOne() {
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        userService.save(buildUser(i));
    }
    long e = System.currentTimeMillis();
    System.out.println("耗时：" + (e - b));
}
```
==为什么这么慢？==

> [!NOTE] 解释
> 我们有10万条数据，每一条数据都会分别去提交，每次往数据库去提交，都是一次网络请求，然后提交到MySQL之后，MySQL去执行，所以每次网络请求都需要耗时，所以这种方式是最慢的。

**方式二：批处理新增,耗时2万ms**
```java
@Test
void testSaveBatch() {
    // 我们每次批量插入1000条，插入100次即10万条数据
    // 1.准备一个容量为1000的集合
    List<User> list = new ArrayList<>(1000);
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        // 2.添加一个user
        list.add(buildUser(i));
        // 3.每1000条批量插入一次
        if (i % 1000 == 0) {
            userService.saveBatch(list);
            // 4.清空集合，准备下一批数据
            list.clear();
        }
    }
    long e = System.currentTimeMillis();
    System.out.println("耗时：" + (e - b));
}
```
**MP采用的是JDBC底层的预编译方案，这种方案他会在便利的过程中把你提交的这个user数据不是直接提交到数据库，而是先对他进行一个编译变成SQL语句。这里每1000条数据才用发了一次网络请求，，相当于只发了100次网络请求，所以性能得以提高。**
==但是==**这个方案因为打包了他是逐条执行的SQL，所以对性能也是有一定的影响

所以要打包成一条SQL语句就能新增的这样的形式：
```sql
INSERT INTO tb_user ( username, password, phone, info, balance, create_time, update_time )
VALUES
('user_1', 123, 18688190001, '', 2000, 2023-07-01, 2023-07-01),
('user_2', 123, 18688190002, '', 2000, 2023-07-01, 2023-07-01),
('user_3', 123, 18688190003, '', 2000, 2023-07-01, 2023-07-01),
('user_4', 123, 18688190004, '', 2000, 2023-07-01, 2023-07-01);
```

**方式三：开启==rewriteBatchedStatements=true参数==**
