### 竞赛排名功能

![](assets/Day18/file-20260124161135110.png)
缓存结构：
数据结构：List
key：  e:r:examId
value:  json:    examRank   userId  score

查看竞赛排名接口
**因为之前在开发定时任务发送消息的时候**就已经有了得分和排名，直接开发接口查看排名列表即可
```java
/*  
 * 查询排名功能  
 */@Override  
public TableDataInfo rankList(ExamRankDTO examRankDTO) {  
    Long total = examCacheManager.getRankListSize(examRankDTO.getExamId());  
    List<ExamRankVO> examRankVOList;  
    if (total == null || total <= 0){  
        PageHelper.startPage(examRankDTO.getPageNum(),examRankDTO.getPageSize());  
        examRankVOList = userExamMapper.selectExamRankList(examRankDTO.getExamId());  
        //同步到redis中  
        examCacheManager.refreshExamRankCache(examRankDTO.getExamId());  
        total = new PageInfo<>(examRankVOList).getTotal(); //获取总记录数  
    } else {  
        //从redis中获取 竞赛列表数据  
        examRankVOList = examCacheManager.getExamRankList(examRankDTO);  
    }  
    if (CollectionUtil.isEmpty(examRankVOList)){ //使用hutool工具包判断集合是否为空  
        return TableDataInfo.empty(); //未查出任何数据时调用  
    }  
    assembleExamRankVOList(examRankVOList); //昵称封装  
    return TableDataInfo.success(examRankVOList, total);  
}
```

