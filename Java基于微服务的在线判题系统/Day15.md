
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

### 用户提交 & 题目答案比对

**业务逻辑**


**friend**
后端接收到前端请求之后，获取参数，根据programType判断用户提交代码的语言类型，根据语言类型进行不同的处理。

根据questionId从ES中查询出题目对应的main函数和测试用例（入参），将代码拼接完整。查询的时候还需查询题目时间、空间限制、难以程度。


**judge**
执行代码：javac 编译  java执行

javac  如果成功：继续执行后续逻辑
      如果失败：终止逻辑，失败原因返回前端
      
java   如果成功：继续执行后续逻辑
      如果失败：终止逻辑，失败原因返回前端

题目答案的比对：根据执行代码实际输出结果和测试用例的output进行比对。
	如果比对一致：题目作答正确，继续执行后续逻辑
     如果不一致：题目作答错误，错误的原因返回前端
 
时间限制&空间限制比对：代码执行使用实际时间&空间和期望时间&空间进行比对。
如果 <= 期望值：符合要求，判定题目作答正确，并将结果返回前端，否则......

对于用户答题结果，无论成功/失败，都应该存储到MYSQL中，供后续使用。
答题结果计算时，分值的计算和题目难易程度相关。


![](assets/Day15/file-20260111154259520.png)
处理方案:
- 分配有限的资源。限制cpu、内存等资源的使用。
- 运行时间限制。不允许用户代码长时间占用资源。
- 限制用户代码执行的权限。限制文件的读写、限制网络的访问等。
- 和系统运行环境隔离开，不同用户代码执行环境也隔离开。
- 如果发现恶意攻击的用户，随时将这个用户进行拉黑处理。

docker 容器相互隔离，相互干扰out；资源滥用数据泄露，可以限制；安全漏洞，在容器里面执行，大部分能化解，容器挂掉再拉一个就行了

这些操作都是Java代码执行，所以就要通过Java操作docker  judge服务主要是拿来判题,所以docker相关依赖应该引入到judge服务。

服务间调用（friend --> judge）的问题：openfign发起服务间调用（说白了就是发起HTTP请求）

创建oj-api的一个模块：把openfeign的客户端统一放在这。
