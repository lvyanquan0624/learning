# 文档汇总，供个人学习

[社区会议同步文档](https://docs.google.com/document/d/1YuGhUdukLP5gGiqCbk0A5_Wifqe2CZWgOd3TbhY3UQg/edit)  
有月度会议和视频回放，记录了从 2020.05 至今版本迭代过程中的抉择讨论，非常值得细看     

## 企业实践

## 源码剖析

[Iceberg文件组织原理](https://mp.weixin.qq.com/s/QE-odbd5O2LBFg3RU1gJPQ)  
个人认为原理讲解得特别清楚的一篇文章   
本质上，一张表是由它的全部数据文件组成，Iceberg做的就是怎么跟踪这些数据文件和利用这些数据文件中记录的统计信息   
![image](https://user-images.githubusercontent.com/38547014/194792347-94c3a321-c3a5-4e6d-b641-5f8e829a3b62.png)  
 


[分析iceberg合并任务解决数据冲突](https://zhuanlan.zhihu.com/p/506740221)  


## 功能跟踪

[Apache Iceberg 1.0.0 RC0 发版](https://lists.apache.org/thread/cr53bdjssovscf79wzhjck9cqs7pt6y3)    
正在讨论中，正式版发布代表着稳定性的提升，一定会吸引更多人关注和使用这个项目  

[Nessie项目](https://github.com/projectnessie/nessie)    
与Iceberg相关的一个项目，在Table Format的基础上提供事务保证和Git式的使用体验

[Arctic项目](https://github.com/NetEase/arctic)
网易开源的数据湖平台项目，提供了文件自动治理的能力，不过目前是基于Iceberg0.12版本的

[支持hilbert curve(进行中)](https://github.com/apache/iceberg/pull/5824)  
hilbert曲线相比z-order曲线在多维查询中效果应该会更好，但是看讨论不太积极，可能zorder已经足够好了

