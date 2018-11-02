Druid在不同场景下，有很多的查询类型。对于各种类型的查询类型的配置可以json属性文件设置。Druid查询类型，概括一下为3大类：



聚合查询 - 时间序列查询（Timeseries）、排名查询（TopN）、分组查询（GroupBy）

元数据查询 - 时间范围\(Time Boundary\) 、段元数据\(Segment Metadata\)、数据源\(Datasource\)

Search查询 - Search以聚合查询为主，与其它查询类型比较相对简单，使用上相对比较少，暂不介绍。

如何对查询进行选择呢？

在可能的情况下，我们建议使用的时间序列和TopN查询代替分组查询，分组查询是Druid最灵活的的查询，但是性能最差。时间序列查询是明显快于GROUPBY查询，因为聚合不需要分组尺寸。对于分组和排序在一个单一的维度，TopN查询更优于GROUPBY。

Druid Json查询属性

Druid json查询比较重要的几个属性是：queryType、dataSource、granularity、filter、aggregator等。



查询类型（queryType）：对应聚合查询下的3种类型值：timeseries、topN、groupBy

数据源（dataSource）：数据源，类似数据库中表的概念，对应数据导入时Json配置属性dataSource值。

聚合粒度（granularity）： 粒度决定如何得到数据块在跨时间维度，在配置查询聚合粒度里有三种配置方法：

简单聚合粒度 - 支持字符串值有：all、none、second、minute、five\_minute，ten\_minute，fifteen\_minute、thirty\_minute、hour、day、week、month、quarter、year，all - 将所有块变成一块， none - 不使用块数据（它实际上是使用最小索引的粒度，none意味着为毫秒级的粒度）；按时间序列化查询时不建议使用none，因为所有的毫秒不存在，系统也将尝试生成0值，这往往是很多。

时间段聚合粒度 - Druid指定一精确的持续时间（毫秒）和时间戳返回UTC（世界标准时间）。

常用时间段聚合粒度 - 与时间段聚合粒度差不多，但是常用时间指平时我们常用时间段，如年、月、周、小时等。下面对3种聚合粒度配置举例说明。

简单聚合粒度

查询粒度比数据采集时配置的粒度小，则不合理，也无意义，因较小粒度（相比）者无索引数据；如查询粒度小于采集时配置的查询粒度时，则Druid的查询结果与采集数据配置的查询粒度结果一样。



