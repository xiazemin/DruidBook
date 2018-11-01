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

    "Justin BIeber": 0,

    "Ke$ha":         1

}



2. 值的列表

\[0,

 0,

 1,

 1\]



3. bitMap

value="Justin Bieber": \[1, 1, 0, 0\]

value="Ke$ha":         \[0, 0, 1, 1\]



