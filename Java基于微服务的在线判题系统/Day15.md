
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

### 用户提交

