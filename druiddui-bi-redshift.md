Redshift 内部使用了亚马逊取得了授权的ParAccel



实时注入数据

抛开可能的性能不同, 有功能性的不同



Druid 适合分析大数据量的流式数据, 也能够实时加载和聚合数据

一般来讲， 传统的数据仓库包括列式存储只摄入批量数据， 没有对流式数据做优化



Druid 是只读分析型数据仓库

Druid支持写语句， 但是数据是不变的， 也不支持join. ParAccel 是完全数据库， 支持SQL语法包括join, insert, update



 



分发数据

Druid的数据分发的单位是segment, segment的数据在高可用的深存储之中， 例如S3和HDFS. 扩展和收缩不会导致大量的复制工作和不可用. 实际上， 一些历史节点失效不会导致数据丢失，因为当历史节点启动的时候会从深存储中拉取数据



想反， ParAccel数据分发是基于hash算法的。 扩展集群会导致在所有节点上重新计算hash， 这就比较难控制可用性。亚马逊的redshif解决问题的变通方案使用多步操作

    设置集群只读

    扩集群并行复制数据

    重定向查询到新的集群





复制策略

Druid使用segment做数据分发， 使更多的节点可以加入和重新平衡数据而不用分步骤交换。 复制策略也是所有副本可以被用来查询。 



ParAccel’s hash-based distribution generally means that replication is conducted via hot spares. This puts a numerical limit on the number of nodes you can lose without losing data, and this 



replication strategy often does not allow the hot spare to help share query load.

复制策略不利于低访问的节点分享查询压力。



索引策略

和列式存储一起， Druid用索引来提高带过滤查询的速度。索引结构会增加存储负担（使修改更难）， 但是显著的增加速度。



ParAccel 没有使用索引

