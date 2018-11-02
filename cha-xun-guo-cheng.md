Druid使用JSON over HTTP 作为底层的查询语言，不过强大的社区也为我们提供了多种查询方式，比如Python接口pydruid、R接口RDruid、JavaScript接口plywood、类SQL接口plyql、PHP接口druid-php等。

Druid查询目前只支持单表操作，基本涵盖了ANSISQL中常用的查询语句，包括：

聚合类\(Aggregation\)查询  
时间序列查询  
TopN查询  
GroupBy

元信息\(Metadata\)类查询  
时间范围查询（数据集最早和最近出现时间点）  
Segment元信息  
DataSource元信息

搜索类\(Search\)查询（包括Select查询）  
不过Druid目前还不支持JOIN类操作，以上已支持的各类查询的详细说明可以参见：[http://druid.io/docs/0.9.1.1/querying/searchquery.html](http://druid.io/docs/0.9.1.1/querying/searchquery.html)

查询过程  
在介绍BrokerNode中已基本概述了Druid是如何查询集群内的数据。BrokerNode作为集群内查询入口，需要了解数据在集群内的分布情况，才能将查询请求发送给对应的数据节点（包括HistoricalNode和Real-TimeNode），BrokerNode会merge每个节点返回的数据，最终返回给用户。

在这里主要说下带有过滤\(filter\)的查询请求，我们知道Segments内部存在位图索引，所以数据的过滤操作完全可以转换为bitmap的按位逻辑操作，所以无论是HistoricalNodes还是Real-TimeNodes，都不需要去查看原始数据，只需要通过位图索引的按位逻辑操作，获得符合过滤条件的行号，再取出需要的列返回给Broker即可。

基本sql 使用

。plysql--   [http://plywood.imply.io/plyql](http://plywood.imply.io/plyql)  （下面的端口应该是8082，我这个地方做了端口转换）

执行sql脚本（bin/plyql -h \*.\*.195.60:8085 -q ‘SHOW TABLES‘）

\[teld@Druid imply-1.3.1\]$ bin/plyql -h \*.\*.195.60:8085 -q ‘SHOW TABLES‘

http://druid.io/docs/0.10.1/querying/querying.html



其他：



Transforming Dimension Values



The following JSON fields can be used in a query to operate on dimension values.



http://druid.io/docs/0.10.1/querying/dimensionspecs.html



Query Context



The query context is used for various query configuration parameters. The following parameters apply to all querie



http://druid.io/docs/0.10.1/querying/query-context.html



Multi-value dimensions



http://druid.io/docs/0.10.1/querying/multi-value-dimensions.html



一、Querying

Queries 是以HTTP REST形式向nodes请求 \(Broker, Historical, or Realtime\). JSON形式标识请求的内容：



curl -X POST '&lt;queryable\_host&gt;:&lt;port&gt;/druid/v2/?pretty' -H 'Content-Type:application/json' -d @&lt;query\_json\_file&gt;

其他形式的第三方查询库： client libraries 



http://druid.io/libraries.html



calcite库

：http://calcite.apache.org/docs/druid\_adapter.html



Apache Calcite - SQL parser, planner and query engine whose Druid adapter can query data residing in Druid, and combine it with data in other locations; has local and remote JDBC drivers powered by Avatica

implydata/plyql - A command line and HTTP interface for issuing SQL queries to Druid

SQL Support for Druid

http://druid.io/docs/0.10.1/querying/sql.html

