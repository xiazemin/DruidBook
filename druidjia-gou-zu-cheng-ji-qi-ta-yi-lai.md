Druid是一个用于大数据实时查询和分析的高容错、高性能开源分布式系统，旨在快速处理大规模的数据，并能够实现快速查询和分析。尤其是当发生代码部署、机器故障以及其他产品系统遇到宕机等情况时，Druid仍能够保持100%正常运行。创建Druid的最初意图主要是为了解决查询延迟问题，当时试图使用Hadoop来实现交互式查询分析，但是很难满足实时分析的需要。而Druid提供了以交互方式访问数据的能力，并权衡了查询的灵活性和性能而采取了特殊的存储格式。

Druid功能介于PowerDrill和Dremel之间，它几乎实现了Dremel的所有功能，并且从PowerDrill吸收一些有趣的数据格式。Druid允许以类似Dremel和PowerDrill的方式进行单表查询，同时还增加了一些新特性，如为局部嵌套数据结构提供列式存储格式、为快速过滤做索引、实时摄取和查询、高容错的分布式体系架构等。从官方得知，Druid的具有以下主要特征：

为分析而设计——Druid是为OLAP工作流的探索性分析而构建，它支持各种过滤、聚合和查询等类；

快速的交互式查询——Druid的低延迟数据摄取架构允许事件在它们创建后毫秒内可被查询到；

高可用性——Druid的数据在系统更新时依然可用，规模的扩大和缩小都不会造成数据丢失；

可扩展——Druid已实现每天能够处理数十亿事件和TB级数据。

Druid应用最多的是类似于广告分析创业公司Metamarkets中的应用场景，如广告分析、互联网广告系统监控以及网络监控等。当业务中出现以下情况时，Druid是一个很好的技术方案选择：

需要交互式聚合和快速探究大量数据时；

需要实时查询分析时；

具有大量数据时，如每天数亿事件的新增、每天数10T数据的增加；

对数据尤其是大数据进行实时分析时；

需要一个高可用、高容错、高性能数据库时。

Historical节点  ：对非实时数据进行处理存储和查询

Realtime节：实时摄取数据、监听输入数据流

Coordinator节点：监控Historical节点

Broker节点：接收来自外部客户端的查询和将查询转发到Realtime和Historical节点

Indexer节点：负责索引服务

一个Druid集群有各种类型的节点（Node）组成，每个节点都可以很好的处理一些的事情，这些节点包括对非实时数据进行处理存储和查询的Historical节点、实时摄取数据、监听输入数据流的Realtime节、监控Historical节点的Coordinator节点、接收来自外部客户端的查询和将查询转发到Realtime和Historical节点的Broker节点、负责索引服务的Indexer节点。

查询操作中数据流和各个节点的关系如下图所示：

如下图是Druid集群的管理层架构，该图展示了相关节点和集群管理所依赖的其他组件（如负责服务发现的ZooKeeper集群）的关系：

一、Druid简介

二、Druid架构组成及相关依赖

三、Druid集群配置

四、Druid集群启动

五、Druid查询

六、后记

一、Druid简介

Druid是一个为大型冷数据集上实时探索查询而设计的开源数据分析和存储系统，提供极具成本效益并且永远在线的实时数据摄取和任意数据处理。

主要特性：

为分析而设计——Druid是为OLAP工作流的探索性分析而构建。它支持各种filter、aggregator和查询类型，并为添加新功能提供了一个框架。用户已经利用Druid的基础设施开发了高级K查询和直方图功能。

交互式查询——Druid的低延迟数据摄取架构允许事件在它们创建后毫秒内查询，因为Druid的查询延时通过只读取和扫描有必要的元素被优化。Aggregate和 filter没有坐等结果。

高可用性——Druid是用来支持需要一直在线的SaaS的实现。你的数据在系统更新时依然可用、可查询。规模的扩大和缩小不会造成数据丢失。

可伸缩——现有的Druid部署每天处理数十亿事件和TB级数据。Druid被设计成PB级别。

就系统而言，Druid功能位于PowerDrill和Dremel之间。它实现几乎所有Dremel提供的工具（Dremel处理任意嵌套数据结构，而Druid只允许一个基于数组的嵌套级别）并且从PowerDrill吸收一些有趣的数据格式和压缩方法。

Druid对于需要实时单一、海量数据流摄取产品非常适合。特别是如果你面向无停机操作时，如果你对查询查询的灵活性和原始数据访问要求，高于对速度和无停机操作，Druid可能不是正确的解决方案。在谈到查询速度时候，很有必要澄清“快速”的意思是：Druid是完全有可能在6TB的数据集上实现秒级查询。

二、Druid架构组成及其他依赖

海量数据实时OLAP分析系统-Druid.io安装配置和体验



2.1 Overlord Node \(Indexing Service\)

Overlord会形成一个加载批处理和实时数据到系统中的集群，同时会对存储在系统中的数据变更（也称为索引服务）做出响应。另外，还包含了Middle Manager和Peons，一个Peon负责执行单个task，而Middle Manager负责管理这些Peons。



2.2 Coordinator Node

监控Historical节点组，以确保数据可用、可复制，并且在一般的“最佳”配置。它们通过从MySQL读取数据段的元数据信息，来决定哪些数据段应该在集群中被加载，使用Zookeeper来确定哪个Historical节点存在，并且创建Zookeeper条目告诉Historical节点加载和删除新数据段。



2.3 Historical Node

是对“historical”数据（非实时）进行处理存储和查询的地方。Historical节点响应从Broker节点发来的查询，并将结果返回给broker节点。它们在Zookeeper的管理下提供服务，并使用Zookeeper监视信号加载或删除新数据段。



2.4 Broker Node

接收来自外部客户端的查询，并将这些查询转发到Realtime和Historical节点。当Broker节点收到结果，它们将合并这些结果并将它们返回给调用者。由于了解拓扑，Broker节点使用Zookeeper来确定哪些Realtime和Historical节点的存在。



2.5 Real-time Node

实时摄取数据，它们负责监听输入数据流并让其在内部的Druid系统立即获取，Realtime节点同样只响应broker节点的查询请求，返回查询结果到broker节点。旧数据会被从Realtime节点转存至Historical节点。



2.6 ZooKeeper

为集群服务发现和维持当前的数据拓扑而服务；



2.7 MySQL

用来维持系统服务所需的数据段的元数据；



2.8 Deep Storage

保存“冷数据”，可以使用HDFS。

