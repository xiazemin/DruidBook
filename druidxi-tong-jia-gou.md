Druid是一个高效的数据查询系统，主要解决的是对于大量的基于时序的数据进行聚合查询。数据可以实时摄入，进入到Druid后立即可查，同时数据是几乎是不可变。通常是基于时序的事实事件，事实发生后进入Druid，外部系统就可以对该事实进行查询。

Druid是一组系统，按照职责分成不同的角色。目前存在五种节点类型：

Historical： 历史节点的职责主要是对历史的数据进行存储和查询，历史节点从Deep Storage下载Segment，然后响应Broker对于Segment的查询将查询结果返回给Broker节点，它们通过Zookeeper来声明自己存储的节点，同时也通过zookeeper来监听加载或删除Segment的信号。

Coordinator：协调节点监测一组历史节点来保证数据的可用和冗余。协调节点读取元数据存储来确定哪些Segment需要load到集群中，通过zk来感知Historical节点的存在，通过在Zookeeper上创建entry来和Historical节点通信来告诉他们加载或者删除Segment

Broker：节点接收外部客户端的查询，并且将查询路由到历史节点和实时节点。当Broker收到返回的结果的时候，它将结果merge起来然后返回给调用者。Broker通过Zook来感知实时节点和历史节点的存在。

Indexing Service: 索引服务是一些worker用来从实时获取数据或者批量插入数据。

Realtime：获取实时数据

![](/assets/druid数据流向.png)除了上述五个节点，Druid还有三个外部依赖：

Zookeeper集群

元数据存储实例：Mysql

Deep Storage：HDFS

Segments

Druid 把它的索引存储到一个Segment文件中，Segment文件是通过时间来分割的。

Segment数据结构

对于摄入到Druid的数据的列，主要分三种类型，时间列，指标列和维度列。

![](/assets/数据结构.png)对于时间列和指标列处理比较简单，直接用LZ4压缩存起来就ok，一旦查询知道去找哪几行，只需要将它们解压，然后用相应的操作符来操作它们就可以了。维度列就没那么简单了，因为它们需要被过滤和聚合，因此每个维度需要下面三个数据结构。

一个map，Key是维度的值，值是一个整型的id

一个存储列的值得列表，用1中的map编码的list

对于列中的每个值对应一个bitmap，这个bitmap用来指示哪些行包含这个个值。

对于上图的Page列，它的存储是这样的

1: 字典

{

```
"Justin BIeber": 0,

"Ke$ha":         1
```

}

1. 值的列表

\[0,

0,

1,

1\]

1. bitMap

value="Justin Bieber": \[1, 1, 0, 0\]

value="Ke$ha":         \[0, 0, 1, 1\]

历史节点

每个历史节点维持一个和Zookeeper的长连接监测一组path来获取新的Segment信息。历史节点互相不进行通信，他们依靠zk来等待协调节点来协调。

协调节点负责把新的Segment分发给历史节点，协调节点通过在zk的指定路径下创建一个entry来向历史节点做分发。

当历史节点发现一个新的entry出现在path中，它首先会检查本地文件缓存看有有没Segment信息，如果没有Segment信息，历史节点会从zk上下载新的Segment的元信息。Segment的元信息包括Segment存在Deep Storage的位置和如何解压和处理Segment。一旦一个历史节点完成对一个Segment的处理，这个历史节点会在zk上的一个路径声明对这个Segment提供查询服务，此刻这个Segment就可以查询了。

查询节点

Broker节点负责将查询路由到历史节点和实时节点，Broker节点通过zk来知道哪些Segment存在哪个节点上。Broker也会把查询的结果进行Merge

大多数Druid查询包含一个区间对象，这个对象用来指定查询所要查的区间段。Druid的Segment也通过时间段进行分割散落在整个集群中。假设有一个简单的数据源，这个数据源有七个Segment，每个Segment包含一周中的某一天的数据。任何一个时间范围超过一天的查询都会落到不止一个Segment上。这些Segment可能分布在集群中不同的节点上。因此这种查询就会涉及到多个节点。

为了确定发送到哪个节点上，Broker会从Historial和RealTime的节点来获取他们提供查询的Segment的信息，然后构建一个时间轴，当收到特定的时间区间的查询时，Broker通过时间轴来选择节点。

Broker节点会维护一个LRU缓存，缓存存着每个Segment的结果，缓存可以是一个本地的缓存或者多个节点共用的外部的缓存如 memcached。当Broker收到查询时候，它首先将查询映射成一堆Segment的查询，其中的一个子集的结果可能已经存在缓存中，他们可以直接从缓存中拉出来，那些没在缓存中的将被发送到相应节点。

协调节点

协调节点负责Segment的管理和分发，协调节点指挥历史节点来加载或者删除Segment，以及Segment的冗余和平衡Segment。协调节点会周期性的进行扫描，每次扫描会根据集群当前的状态来决定进一步的动作。和历史节点和Broker一样，协调节点通过zk来获取Segment信息，同时协调节点还通过数据库来获取可用的Segment信息和规则。在一个Segment提供查询之前，可用的历史节点会按照容量去排序，容量最小的具有最高的优先级，协调节点就会让它去加载这个Segment然后提供服务。

清理Segment，Druid会将集群中的Segment和数据库中的Segment进行对比，如果集群有的的数据库中没有的会被清理掉。同事那些老的被新的替换的Segment也会被清理掉。

Segment可用性, 历史节点可能因为某种原因不可用，协调节点会发现节点不可用了，会将这个节点上的Segment转移到其他的节点。Segment不会立即被转移，如果在配置的时间段内节点恢复了，历史节点会从本地缓存加载Segment。恢复服务

Segment负载均衡，协调节点会找到Segment最多的节点和Segment最少的节点，当他们的比例超过一个设定的值的时候，协调节点会从Segment最多的节点转移到Segment最少的节点。

索引服务

索引服务是一个高可用的，分布式的服务来运行索引相关的Task。索引服务会创建或者销毁Segment。索引服务是一个Master/Slave架构。索引服务是三个组件的集合

peon组件用来跑索引任务。

Middle Manager组件用来管理peons

Overlord向MiddleManager分发任务。

索引服务

![](/assets/索引服务.png)

Overlord节点负责接受任务，协调任务分发，创建锁，和返回状态给调用者。Overlord节点可以以本地模式或者远程模式运行。本地模式会直接创建Peon，远程模式会通过Middle Manager创建任务。

实时节点

实时节点提供实时索引服务，通过实时节点索引的数据立即可查。实时节点会周期性的构建Segment，并且把这些Segment推到历史节点并修改元数据。

![](/assets/时序图.png)

