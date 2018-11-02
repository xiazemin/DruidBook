Druid 是一个为在大数据集之上做实时统计分析而设计的开源数据存储。这个系统集合了一个面向列存储的层，一个分布式、shared-nothing的架构，和一个索引结构，来达成在秒级以内对十亿行级别的表进行任意的探索分析。

由于Druid保存的是时间序列数据，按列的类型，上述数据可以分为以下三类：

时间序列\(timestamp\)：Druid中所有的查询以及索引过程都和时间维度息息相关。Druid底层使用绝对毫秒数保存时间戳，默认使用ISO-8601格式展示时间。\(如：yyyy-MM-ddThh:mm:sss.SSSZ，其中“Z”代表零时区，中国所在的东八区可表示为+08:00\)

维度列\(dimensions\)：Druid中的维度与Olap中的维度一致，一条记录中的字符类型（String）数据可以看作是维度列，维度列被用于过滤筛选，分组数据。

度量列\(Metrics\)：Druid中的Metric也与Olap中的Metric一致。度量列被用于聚合和计算操作。

这也是Druid要求的格式，第一列为时间。Appkey和area都是维度列。value为metric列。

Druid会在导入阶段对数据进行Rollup，将维度相同组合的数据进行聚合处理。Rollup会使用其设定的聚合器进行聚合。



按天聚合后的数据如下（聚合后的DataSource为AD\_areauser\):



