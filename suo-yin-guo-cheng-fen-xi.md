Druid底层不保存原始数据，而是借鉴了Apache Lucene、Apache Solr以及ElasticSearch等检索引擎的基本做法，对数据按列建立索引，最终转化为Segment，用于存储、查询与分析。

首先，无论是实时数据还是批量数据在进入Druid前都需要经过Indexing Service这个过程。在Indexing Service阶段，Druid主要做三件事：第一，将每条记录转换为列式\(columnar format\)；第二，为每列数据建立位图索引；第三，使用不同的压缩算法进行压缩，其中默认使用LZ4，对于字符类型列采用字典编码\(Dictionary encoding\)进行压缩，对于位图索引采用Concise/Roaring bitmap进行编码压缩。最终的输出结果也就是Segment



