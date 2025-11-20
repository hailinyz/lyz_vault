## 1.分页插件
首先，要在配置类中注册MyBatisPlus的核心插件，同时添加分页插件：
```java
@Configuration  
public class MyBatisConfig {  
      
    @Bean  
    public MybatisPlusInterceptor mybatisPlusInterceptor(){  
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();  
        //1.创建分页插件  
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);  
        paginationInnerInterceptor.setMaxLimit(1000L); // 设置单页最大数量  
        //2.添加分页插件  
        interceptor.addInnerInterceptor(paginationInnerInterceptor);  
  
        return interceptor;  
    }  
      
}
```

接着，就可以使用分页的API了：
```java
@Test
void testPageQuery() {
    int pageNo = 1, pageSize = 2;
    // 1.准备分页条件
    // 1.1.分页条件
    Page<User> page = Page.of(pageNo, pageSize);
    // 1.2.排序条件
    page.addOrder(new OrderItem("balance", true));
    page.addOrder(new OrderItem("id", true));

    // 2.分页查询
    Page<User> p = userService.page(page);

    // 3.解析
    long total = p.getTotal();
    System.out.println("total = " + total);
    long pages = p.getPages();
    System.out.println("pages = " + pages);
    List<User> users = p.getRecords();
    users.forEach(System.out::println);
}
```

## 2.通用分页实体

**需求:遵循下面的接口规范，编写一个UserController接口，实现User的分页查询**

| 参数   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 请求方式 | GET                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| 请求路径 | /users/page                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 请求参数 | {<br>  "pageNo": 1,<br>  "pageSize": 5,<br>  "sortBy": "balance",<br>  "isAsc": false,<br>  "name": "jack",<br>  "status": 1<br>}                                                                                                                                                                                                                                                                                                                                                                                                      |
| 返回值  | {<br>  "total": 1005,<br>  "pages": 201,<br>  "list": [<br>    {<br>      "id": 1,<br>      "username": "Jack",<br>      "info": {<br>        "age": 21,<br>        "gender": "male",<br>        "intro": "佛系青年"<br>      },<br>      "status": "正常",<br>      "balance": 2000<br>    },<br>    {<br>      "id": 2,<br>      "username": "Rose",<br>      "info": {<br>        "age": 20,<br>        "gender": "female",<br>        "intro": "文艺青年"<br>      },<br>      "status": "冻结",<br>      "balance": 1000<br>    }<br>  ]<br>} |
| 特殊说明 | - 如果排序字段为空，默认按照更新时间排序<br>- 排序字段不为空，则按照排序字段排序                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
