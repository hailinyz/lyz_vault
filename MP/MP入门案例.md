## 为什么使用MP

**MP即==MyBatisPlus==**，可以对Mybatis进行改造
```java
public interface UserMapper {

    void saveUser(User user);
    
    void deleteUser(Long id);
    
    void updateUser(User user);
    
    User queryUserById(@Param("id") Long id);
    
    List<User> queryUserByIds(@Param("ids") List<Long> ids);
}
```
+ 这些SQL语句对应的xml文件虽然不难，但是**繁琐**

## 怎么使用MP呢

1. 引入MP**依赖**
MP官方提供了starter，其中集成了Mybatis和MP的所有功能，并且实现了自动装配的效果。
因此我们可以用MybatisPlus代替Mybatis的starter:
```xml
<!--MybatisPlus--> 
<dependency> 
	<groupId>com.baomidou</groupId> 
	<artifactId>mybatis-plus-boot-starter</artifactId> 
	<version>3.5.3.1</version> 
</dependency>
```
2. 定义Mapper
自定义的Mapper继承MybatisPlus提供的BeanMapper接口：
**注意：在继承的时候要注意实体类为你的实体类的类型，只有这样才知道你的增删改查操作的是哪个实体**
```java
public interface UserMapper extends BaseMapper<User> {

}
```
继承之后这些方法就都可以直接用了
![700](assets/MP入门案例/file-20251120000848405.png)

## MP常见注解
**MP通过扫描实体类，并基于==反射==获取是实体信息作为数据库表信息。**
```java
// 就是这个
public interface UserMapper extends BaseMapper<User> {

}
```
通过==反射==拿到对应的字节码信息
```java
import lombok.Data;
import java.time.LocalDateTime;

@Data
public class User {
    private Long id;
    private String username;
    private String password;
    private String phone;
    private String info;
    private Integer status;
    private Integer balance;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```
有如下规定
+ 美名驼峰转下划线为表名
+ 名为id字段作为主键
+ 变量名驼峰转下划线作为表的字段名
若不符合以上规定，可以使用MP常用的几个注解
+ @TableName：用来指定表名 
+ @TableId：用来指定表中的主键字段信息 
+ @TableField：用来指定表中的普通字段信息
注意：
1. 一定要有主键，不然将来MP找不到主键，无法进行增删改查操作
2. 主键自增长
   + AUTO：数据库自增长
   + INPUT:通过set方法自行输入
   + ASSIGN_ID：分配ID 接口IdentifierGenerator的方法nextId来生成id，默认实现类为DefaultIdentifierGenerator雪花算法