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

