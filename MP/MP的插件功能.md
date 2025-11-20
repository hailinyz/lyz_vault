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
    // 1.查询
    int pageNo = 1, pageSize = 5;
    // 1.1.分页参数
    Page<User> page = Page.of(pageNo, pageSize);
    // 1.2.排序参数，通过OrderItem来指定
    page.addOrder(new OrderItem("balance", false));
    // 1.3.分页查询
    Page<User> p = userService.page(page);
    // 2.总条数
    System.out.println("total = " + p.getTotal());
    // 3.总页数
    System.out.println("pages = " + p.getPages());
    // 4.分页数据
    List<User> records = p.getRecords();
    records.forEach(System.out::println);
}
```
