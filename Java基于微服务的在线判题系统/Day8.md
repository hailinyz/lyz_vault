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

有三个地方可以进行竞赛发布操作
1. 添加竞赛

2. 编辑竞赛

3. 竞赛列表当中