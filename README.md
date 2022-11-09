# Iceberg

[社区会议同步文档](https://docs.google.com/document/d/1YuGhUdukLP5gGiqCbk0A5_Wifqe2CZWgOd3TbhY3UQg/edit).  
有月度会议和视频回放，记录了从 2020.05 至今版本迭代过程中的讨论.  
[1.0.0 RC0 发版投票](https://lists.apache.org/thread/cr53bdjssovscf79wzhjck9cqs7pt6y3).     
已经Release.   
[Apache Iceberg 0.14 特性介绍](https://mp.weixin.qq.com/s/AZxNAe9pXpLYyx5BlG35bw).     
[Apache Iceberg 0.13 特性介绍](https://mp.weixin.qq.com/s/n5YmWEwyEf5nFVGxSjJUxA).  
[Apache Iceberg 0.11 特性介绍](https://mp.weixin.qq.com/s/-BnuH_uVM4SRhXSTcz75yg).  

## 学习资料
[实践数据湖iceberg](https://blog.csdn.net/spark_dev/category_11588525.html) 新手入门参考，有很多实践案例.  
<img width="756" alt="Screen Shot 2022-10-12 at 11 53 00 AM" src="https://user-images.githubusercontent.com/38547014/195246091-09b60c59-5895-40b8-84c5-a8299031090d.png">


## 企业实践
[2021-04-06 Flink 集成 Iceberg 在同程艺龙的生产实践](https://mp.weixin.qq.com/s/PvW6KAL6Z8_0-PIDX2blfw).   
[2022-07-12 Iceberg在腾讯微视实时场景的应用](https://mp.weixin.qq.com/s/aEhZJUXOmNilHuImf7KB0A).  
[2022-08-06 Apache Iceberg在小红书的探索与实践](https://mp.weixin.qq.com/s/rw3iKYiHoGvTQG04nK0Y8w).  
[2022-08-29 Apache Iceberg在网易严选批流一体的实践](https://mp.weixin.qq.com/s/Ui2WRyu2eV3gqTh-kyGupw).  
[2022-09-22 业内首个基于Iceberg的“云端仓转湖”生产实践探索](https://mp.weixin.qq.com/s/v-VKjt_kDCXh3fj5Xvm_dQ).  
[2022-10-29 Spark读写Iceberg在腾讯的实践和优化](https://mp.weixin.qq.com/s/KK0pMt40dMOWevF8t6pdtw).  

## 源码剖析

[Iceberg文件组织原理](https://mp.weixin.qq.com/s/QE-odbd5O2LBFg3RU1gJPQ).  
个人认为原理讲解得特别清楚的一篇文章.   
本质上，一张表是由它的全部数据文件组成，Iceberg做的就是怎么跟踪这些数据文件和利用这些数据文件中记录的统计信息.     
![image](https://user-images.githubusercontent.com/38547014/194792347-94c3a321-c3a5-4e6d-b641-5f8e829a3b62.png).    
[Flink读取Iceberg表的实现源码解读](https://mp.weixin.qq.com/s/-rgbEyIteqx8txXuKOy4lw).     
[Flink 流式写入Iceberg实现原理解析](https://mp.weixin.qq.com/s/jIcQbpj1OtQ71m6YFCItng).   
[Iceberg事务特性解读](https://mp.weixin.qq.com/s/ppDm1SzmfHKDd585i6TGZQ).  
[分析iceberg合并任务解决数据冲突](https://zhuanlan.zhihu.com/p/506740221).   

### 二级索引
[Puffin索引文件](https://iceberg.apache.org/puffin-spec/).          
[Puffin and Iceberg：海雀与冰山齐飞](https://mp.weixin.qq.com/s/VrAQHzFevOL2JDjnFM3Low).   
Puffin Format是一种Iceberg自定义的文件格式，在理解上可以认为是和orc、parquet同一级别的，用于保存表的索引信息。目前Puffin文件的读/写/序列化/反序列化都已经实现好了。  

基于Puffin文件，Iceberg定义了表信息统计的接口[StatisticsFile](https://github.com/apache/iceberg/blob/master/api/src/main/java/org/apache/iceberg/StatisticsFile.java)，包含了一组BlobMetadata信息，BlobMetadata的定义如下：
```

/** A metadata about a statistics or indices blob. */
public interface BlobMetadata {
  /** Type of the blob. Never null */
  String type();

  /** ID of the Iceberg table's snapshot the blob was computed from */
  long sourceSnapshotId();

  /** Sequence number of the Iceberg table's snapshot the blob was computed from */
  long sourceSnapshotSequenceNumber();

  /** Ordered list of fields the blob was calculated from. Never null */
  List<Integer> fields();

  /** Additional properties of the blob, specific to the blob type. Never null */
  Map<String, String> properties();
}

```
其中properties是允许自定义的统计信息。这个统计信息也会随着Snapshot的变更而改变，通过sourceSnapshotId跟踪对应的Snapshot。具体使用可以参考这个单元测试
[TestSetStatistics](https://github.com/apache/iceberg/blob/master/core/src/test/java/org/apache/iceberg/TestSetStatistics.java)。注意到读取的索引文件格式就是.puffin结尾的。  

### 扫描指标
统计文件级别的skip信息
```
ScanMetrics.java
public static final String TOTAL_PLANNING_DURATION = "total-planning-duration";
public static final String RESULT_DATA_FILES = "result-data-files";
public static final String RESULT_DELETE_FILES = "result-delete-files";
public static final String SCANNED_DATA_MANIFESTS = "scanned-data-manifests";
public static final String SCANNED_DELETE_MANIFESTS = "scanned-delete-manifests";
public static final String TOTAL_DATA_MANIFESTS = "total-data-manifests";
public static final String TOTAL_DELETE_MANIFESTS = "total-delete-manifests";
public static final String TOTAL_FILE_SIZE_IN_BYTES = "total-file-size-in-bytes";
public static final String TOTAL_DELETE_FILE_SIZE_IN_BYTES = "total-delete-file-size-in-bytes";
public static final String SKIPPED_DATA_MANIFESTS = "skipped-data-manifests";
public static final String SKIPPED_DELETE_MANIFESTS = "skipped-delete-manifests";
public static final String SKIPPED_DATA_FILES = "skipped-data-files";
public static final String SKIPPED_DELETE_FILES = "skipped-delete-files";
public static final String INDEXED_DELETE_FILES = "indexed-delete-files";
public static final String EQUALITY_DELETE_FILES = "equality-delete-files";
public static final String POSITIONAL_DELETE_FILES = "positional-delete-files";

```

### 分区隐藏

### 多维数据排序
[Zorder curve](https://iceberg.apache.org/docs/latest/spark-procedures/#rewrite_data_files).  
[Hilbert curve](https://github.com/apache/iceberg/pull/5824).  

### Partial Update
https://github.com/apache/iceberg/pull/6043.  

### Flink
[Flink Iceberg Source 并行度推断源码解析](http://www.54tianzhisheng.cn/2022/05/02/flink-iceberg-source-parallelism/).  

### Spark

## 其他相关项目
[Nessie](https://github.com/projectnessie/nessie).        
提供事务保证和Git式的使用体验.  
[Arctic](https://github.com/NetEase/arctic).    
网易开源的数据湖平台项目，不过目前是基于Iceberg0.12版本的.   
[基于 Flink 实现 Kafka + Iceberg 的流批一体 Hybrid Storage](https://github.com/flink-china/flink-forward-asia-hackathon-2021/issues/28).  
flink-forward-asia-hackathon-2021 参赛作品.  
