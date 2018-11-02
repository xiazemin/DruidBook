Elasticsearch\(ES\) 是基于Apache Lucene的搜索服务器。它提供了全文搜索的模式，并提供了访问原始事件级数据。 Elasticsearch还提供了分析和汇总支持。根据研究，ES在数据获取和聚集用的资源比在Druid高。



Druid侧重于OLAP工作流程。Druid是高性能（快速聚集和获取）以较低的成本进行了优化，并支持广泛的分析操作。Druid提供了结构化的事件数据的一些基本的搜索支持。



 



Segment: Druid中有个重要的数据单位叫segment，其是Druid通过bitmap indexing从raw data生成的（batch or realtime）。segment保证了查询的速度。可以自己设置每个segment对应的数据粒度，这个应用中广告流量查询的最小粒度是天，所以每天的数据会被创建成一个segment。注意segment是不可修改的，如果需要修改，只能够修改raw data，重新创建segment了。

