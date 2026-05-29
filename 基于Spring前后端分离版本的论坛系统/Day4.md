# 登录后端
## 实现流程
![](assets/Day4/file-20260529111442731.png)
![](assets/Day4/file-20260529121836908.png)

![](assets/Day4/file-20260529114556678.png)
![](assets/Day4/file-20260529114605972.png)
这些校验注解是为了减轻数据库的一些校验，能提升数据库性能
![](assets/Day4/file-20260529115045156.png)

# 登录前端
![](assets/Day4/file-20260529132631058.png)

# 获取用户信息后端
## 根据id的值判断User对象的获取方式

**如果id为空，则获取Session作用域中的User对象(当前登录用户)
如果id不为空，从数据库中按id查询出用户信息（查看别人的信息）**

## 修复响应结果中问题
![](assets/Day4/file-20260529140324199.png)
**这些敏感信息不应该再网络中传输也不应该返回给前端**
![](assets/Day4/file-20260529141610532.png)
![](assets/Day4/file-20260529142012980.png)
**处理日期格式**
```yml
# 在spring下加⼊⼦节点
spring:
 # JSON序列化配置
 jackson:
 date-format: yyyy-MM-dd HH:mm:ss # ⽇期格式
 default-property-inclusion: NON_NULL # 不为null时序列化

```
![](assets/Day4/file-20260529143203336.png)

**但是当我们在登录接口以及其他接口时，是没有data的，但是又想让他出现，可以这样配置**
![](assets/Day4/file-20260529143444784.png)
![](assets/Day4/file-20260529143901635.png)
**得到：**
![](assets/Day4/file-20260529143924041.png)

# 获取用户信息前端