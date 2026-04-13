调试监控的平台，配置好环境变量和key就可以追踪了。零代码侵入
![](assets/LangSmith跟踪LLM应用/file-20260406103051461.png)
以上就是我们聊天模型核心能力

# 其他核心组件

并不是所有组件都实现了Runable接口

## 消息（Message）
 不管是什么消息，langchain都是继承了**BaseMessage**
 ![](assets/LangSmith跟踪LLM应用/file-20260406105144273.png)
## 消息--多轮对话
之前都是和大模型进行单次对话，怎么和大模型进行多轮对话

对于原本的大模型来说，发现大模型没有记忆功能
![](assets/LangSmith跟踪LLM应用/file-20260406105726570.png)
没有记忆功能怎么办，可以用**消息列表**![](assets/LangSmith跟踪LLM应用/file-20260406105838296.png)
实际上这一步可以做到**多轮对话**


这是让model有历史记忆的能力（内存级别的管理方法：**内存缓存**）
![](assets/LangSmith跟踪LLM应用/file-20260406111027499.png)
但是这种方法已经**不再推荐**了，实际上历史记忆管理在lang grarp会有很大作用（LangGraph持久性）

上下文太大，大模型可处理的Token数不够，可以控制输入（**管理消息列表**）

## 管理消息列表 -- 裁剪trim

## 过滤filter_message
![](assets/LangSmith跟踪LLM应用/file-20260406113552426.png)

## 消息合并merge_message_runs


# 提示词模板
![](assets/LangSmith跟踪LLM应用/file-20260412223826458.png)



有文本提示词模板，聊天消息补模板

# 少样本提示
![](assets/LangSmith跟踪LLM应用/file-20260413212202799.png)

