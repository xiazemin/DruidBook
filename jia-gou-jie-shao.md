Druid 由上面介绍的角色组成的构架图：

![](/assets/druid构架.png)

查询路径：红色箭头:①客户端向Broker发起请求,Broker会将请求路由到②实时节点和③历史节点

Druid数据流转:黑色箭头：数据源包括实时流和批量数据. ④实时流经过索引直接写到实时节点，⑤批量数据通过IndexService存储到DeepStorage,⑥再由历史节点加载. ⑦实时节点也可以将数据转存到DeepStorage

Druid的集群依赖了ZooKeeper来维护数据拓扑. 每个组件都会与ZooKeeper交互，如下：

![](/assets/druidzookeeper.png)实时节点在转存Segment到DeepStorage, 会写入自己转存了什么Segment

协调节点管理历史节点,它负责从ZooKeeper中获取要同步/下载的Segment,并指派任务给具体的历史节点去完成

历史节点从ZooKeeper中领取任务,任务完成后要将ZooKeeper条目删除表示完成了任务

Broker节点根据ZooKeeper中的Segment所在的节点, 将查询请求路由到指定的节点

对于一个查询路由路径,Broker只会将请求分发到实时节点和历史节点, 因此元数据存储和DeepStorage都不会参与查询中\(看做是后台的进程\).

MetaData Storage 与 Zookeeper

MetaStore和ZooKeeper中保存的信息是不一样的. ZooKeeper中保存的是Segment属于哪些节点. 而MetaStore则是保存Segment的元数据信息

为了使得一个Segment存在于集群中,MetaStore存储的记录是关于Segment的自描述元数据: Segment的元数据,大小,所在的DeepStorage

元数据存储的数据会被协调节点用来知道集群中可用的数据应该有哪些\(Segment可以通过实时节点转存或者批量数据直接写入\).

除了上面介绍的节点角色外，Druid还依赖于外部的三个组件：ZooKeeper, Metadata Storage, Deep Storage，数据与查询流的交互图如下：

