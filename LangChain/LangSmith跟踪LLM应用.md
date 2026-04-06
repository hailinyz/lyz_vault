调试监控的平台，配置好环境变量和key就可以追踪了。零代码侵入
![](assets/LangSmith跟踪LLM应用/file-20260406103051461.png)
以上就是我们聊天模型核心能力

# 其他核心组件

并不是所有组件都实现了Runable接口

## 消息（Message）
 不管是什么消息，langchain都是继承了**BaseMessage**
 