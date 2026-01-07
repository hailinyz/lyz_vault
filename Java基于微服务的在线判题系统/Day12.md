## C端题目列表功能

竞赛列表： 引入redis

数据结构： list                       key  q:l                          value   question_id                    

题目详情缓存： string类型     key   q:d:questionId      value    json题目详情

模糊查找：1.Java原生的方式， 通过for循环 进行模糊搜索
                 2. list结构 key  q:l:合并    q:l:有序   q:l:两 ...          valuse   qiestionId

#### 针对于这种搜索这块包含模糊查询我们业界会选择ES

ES解决 全文搜索（全部字段） 、模糊查询（搜索） 、数据分析（提供分析语法，例如聚合）