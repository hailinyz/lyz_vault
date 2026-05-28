# 实现业务功能
![](assets/Day3/file-20260526122413460.png)

## 注册DAO
### 顺序图
![](assets/Day3/file-20260526122607739.png)
### 参数要求
![](assets/Day3/file-20260526122730932.png)
![](assets/Day3/file-20260526122948005.png)
## 注册Service
![](assets/Day3/file-20260526132812392.png)

## 注册Controller
Controller层一般不处理异常，异常 处理在Servicce 层，直接返回**错误信息**就行
Controller还是推荐使用改成**标准 JSON + @RequestBody**
```java
@PostMapping("/register")
public AppResult register(@RequestBody User user) {
     ......
    // 下面逻辑不变
}
```

# 前端--注册
导入前端代码**放在static这个目录下**
![](assets/Day3/file-20260528235405123.png)
前端主要是发**AJAX**请求就行
