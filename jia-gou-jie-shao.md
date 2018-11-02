Druid 由上面介绍的角色组成的构架图：

![](/assets/druid构架.png)

查询路径：红色箭头:①客户端向Broker发起请求,Broker会将请求路由到②实时节点和③历史节点

Druid数据流转:黑色箭头：数据源包括实时流和批量数据. ④实时流经过索引直接写到实时节点，⑤批量数据通过IndexService存储到DeepStorage,⑥再由历史节点加载. ⑦实时节点也可以将数据转存到DeepStorage

Druid的集群依赖了ZooKeeper来维护数据拓扑. 每个组件都会与ZooKeeper交互，如下：

![](/assets/druidzookeeper.png)

