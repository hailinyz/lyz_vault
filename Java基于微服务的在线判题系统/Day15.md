
### 竞赛中获取上一题/下一题

```java
/*  
 * 获取上一题（竞赛内）  
 */@Override  
public String preQuestion(Long examId, Long questionId) {  
    checkAndRefresh(examId);  
    //到这里才去redis中获取上一题的 id    return examCacheManager.preQuestion(examId, questionId).toString();  
}  
  
  
/*  
 * 获取下一题(竞赛内)  
 */@Override  
public String nextQuestion(Long examId, Long questionId) {  
    checkAndRefresh(examId);  
    //到这里才去redis中获取上一题的 id    return examCacheManager.nextQuestion(examId, questionId).toString();  
}
```

### 用户提交 & 判题

**业务逻辑**

后端接收到前端请求之后，获取参数，根据programType判断用户提交代码的语言类型，根据语言类型进行不同的处理。

执行代码：

