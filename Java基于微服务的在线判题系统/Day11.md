问题：
想要根据开始/结束时间查竞赛列表有如下问题
+ 当Redis有缓存时，直接从Redis取数据，完全忽略了startTime和endTime参数
+ Redis缓存的是全部竞赛列表，没有按时间过滤

优化一下**查询竞赛列表接口**(如果有时间过滤条件，直接查数据库 )
```java
@Override  
public TableDataInfo redisList(ExamQueryDTO examQueryDTO) {  
    // 如果有时间过滤条件，直接查数据库  
    if (examQueryDTO.getStartTime() != null || examQueryDTO.getEndTime() != null) {  
        List<ExamVO> examVOList = list(examQueryDTO);  
        long total = new PageInfo<>(examVOList).getTotal();  
        return TableDataInfo.success(examVOList, total);  
    }  
  
    // 没有时间过滤，走缓存逻辑  
    //从redis中获取 竞赛列表数据  
    Long total = examCacheManager.getListSize(examQueryDTO.getType());  
    List<ExamVO> examVOList;  
    if (total == null || total <= 0){  
        //从数据库中获取 竞赛列表数据  
        examVOList = list(examQueryDTO);  
        //同步到redis中  
        examCacheManager.refreshCache(examQueryDTO.getType());  
        total = new PageInfo<>(examVOList).getTotal(); //获取总记录数  
    } else {  
        //从redis中获取 竞赛列表数据  
        examVOList = examCacheManager.getExamVOList(examQueryDTO);  
        total =  examCacheManager.getListSize(examQueryDTO.getType()); // 获取总记录数  
    }  
    if (CollectionUtil.isEmpty(examVOList)){ //使用hutool工具包判断集合是否为空  
        return TableDataInfo.empty(); //未查出任何数据时调用  
    }  
    return TableDataInfo.success(examVOList, total);  
}
```

## 竞赛报名功能

竞赛报名和比赛、竞赛排名、我的竞赛、竞赛列表有关

比赛：已经报名、已经开塞          哪些用户报名需要记录下来
