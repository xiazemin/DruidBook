Caravel是由Airbnb开发的数据可视化平台，提供Druid和Python DBAPI支持的关系型数据库，包括Mysql、Postgres、Presto、Kylin等。提供简单直观的配置界面，配置一个漂亮的Dashboard非常简单。











针对Druid使用中遇到的问题都提供了比较完善的解决方案：



查询不好写 -&gt; Caravel可以通过拖拽和选择配置报表



明细数据无法导出 -&gt; Caravel同时支持Presto和Druid，通过改造Caravel使其支持明细数据走Presto查询，汇总走Druid



踩过的坑



时区问题Druid切片依赖时间戳，默认时间戳为UTC，可能会导致切片不正确。需要通过指定JVM默认时区的方法指明时区。Caravel对关系数据库的默认时区支持同样有问题，计划解决后给官方提PR。



Druid官方文档中有对protobuf的支持，实际使用中不支持include，不支持嵌套结构。基本没有达到可用状态。



对非JSON的数据，尽量使用标准CSV格式。否则各种转转义问题会很麻烦。



不要把订单号之类的唯一键作为维度，没有意义。



适当选择分片大小。使用中发现通过MR索引，实际的索引会在reduce步进行，每个reducer处理一个分片。所以适当增加reduce内存。数据倾斜时可以考虑用多次MR索引。



结论



Druid有下面的优缺点：



优点：



高可用，水平扩展。靠堆硬件就能解决性能问题；



高效和压缩和索引算法，能做到PB级数据量下的过滤汇总10ms以下的相应时间；



实时导入和批量索引结合；



schema灵活。同一个datasource下的不同segment可以有不同的指标与维度；



缺点：



数据经过初步聚合索引，无法看到明细数据；



无法更新已导入的数据。更新需要重新索引整个segment；



维度和指标需要预先定义好，添加新指标需要重新索引，才能对历史数据生效；



经常有人将Druid和其他的OLAP引擎比较，和常见的可以用作OLAP的引擎比较：



对比ES：



ES偏向于文本检索。虽然ES目前支持聚合查询，但是聚合操作占用的资源非常高。



Druid更适合聚合分析。



对比Spark/Hive/Impala/Presto等查询引擎：



这些查询引擎重点是更灵活的查询方式和完整的SQL支持。



Druid专注于对热点数据的探索分析。



对比Kylin



在Kylin的开发者邮件组里，对这两个引擎做过非常深入的比较，主要有下面几点：



Kylin更适合复杂的OLAP场景，Druid适合实时分析场景；



Druid能实时从kafka拉取数据，Kylin的近实时分析功能还处在开发阶段；



Druid使用倒排Bitmap索引，Kylin对历史数据使用Olap cube存储；



Kylin支持SQL，Druid不支持SQL查询；

