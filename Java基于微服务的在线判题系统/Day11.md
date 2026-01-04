问题：
想要根据开始/结束时间查竞赛列表有如下问题
+ 当Redis有缓存时，直接从Redis取数据，完全忽略了startTime和endTime参数
+ Redis缓存的是全部竞赛列表，没有按时间过滤

