## 竞赛删除

竞赛删除 = 删除竞赛基本信息（tb_exam） + 删除竞赛题目信息（tb_exam_question）
1. 找到要删除的竞赛，点击删除按钮。前端携带竞赛 id 向后端发起删除竞赛请求
2. 后端接收请求后，根据竞赛 id （先判断竞赛是否存在，竞赛是否开赛）删除竞赛基本信息 和 竞赛题目信息。并且返回删除结果
3. 前端接收到响应，如果成功/失败

```java
/*  
删除竞赛  
 */@Override  
public int delete(Long examId) {  
    //判断竞赛是否存在  
    Exam exam = getExam(examId);  
    //判断竞赛是否开始  
    checkExam(exam);  
    //删除竞赛中的题目  
    examQuestionMapper.delete(new LambdaQueryWrapper<ExamQuestion>()  
            .eq(ExamQuestion::getExamId, examId));  
    //删除竞赛  
    return examMapper.deleteById(exam);  
}
```

## 竞赛发布

竞赛发布的前提：竞赛存在、竞赛中有题目

有三个地方可以进行竞赛发布操作
A添加竞赛
1. 在满足发布竞赛前提后，点击发布竞赛。前端携带竞赛 id 向后端发起请求
2. 后端接收到请求后，根据竞赛 id 判断前提是否成立。
   如果不成立：将原因发给前端
   如果成立：将竞赛状态字段从0改为1，并且同步到数据库，将修改结果发给前端
3. 前端接收到后端响应之后
   如果失败，展示失败原因
   如果成功，跳转回列表页，当前状态变回已发布
   C端竞赛列表中要能找到已发布竞赛（可以将0的竞赛过滤掉，后面可能会传参数给接口）


B 编辑竞赛

C竞赛列表当中