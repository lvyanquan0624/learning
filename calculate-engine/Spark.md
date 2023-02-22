### Spark 临时表的生命周期
参考文档 https://support.huaweicloud.com/cmpntguide-mrs/mrs_01_2024.html    
Spark的表管理层次如下图所示，最底层是Spark的临时表，存储着使用DataSource方式的临时表，在这一个层面中没有数据库的概念，因此对于这种类型表，表名在各个数据库中都是可见的。   

上层为Hive的MetaStore，该层有了各个DB之分。在每个DB中，又有Hive的临时表与Hive的持久化表，因此在Spark中允许三个层次的同名数据表。  

查询的时候，Spark SQL优先查看是否有Spark的临时表，再查找当前DB的Hive临时表，最后查找当前DB的Hive持久化表。  

![image](https://user-images.githubusercontent.com/38547014/220511390-6a39d9a6-b653-4844-ae01-615cb6671e72.png)


当Session退出时，用户操作相关的临时表将自动删除。建议用户不要手动删除临时表。

删除临时表时，其优先级与查询相同，从高到低为Spark临时表、Hive临时表、Hive持久化表。如果想直接删除Hive表，不删除Spark临时表，您可以直接使用drop table DbName.TableName命令。
