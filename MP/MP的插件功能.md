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
