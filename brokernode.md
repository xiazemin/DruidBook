Broker节点扮演着历史节点和实时节点的查询路由的角色。

Broker节点知道发布于Zookeeper中的关于哪些segment是可查询的和这些segment是保存在哪里的，Broker节点就可以将到来的查询请求路由到正确的历史节点或者是实时节点，

Broker节点也会将历史节点和实时节点的局部结果进行合并，然后返回最终的合并后的结果给调用者

缓存：Broker节点包含一个支持LRU失效策略的缓存。这个缓存可以使用本地堆内存或者是一个外部的分布式 key/value 存储，例如Memcached

每次Broker节点接收到查询请求时，都会先将查询映射到一组segment中去。这一组确定的segment的结果可能已经存在于缓存中，而不需要重新计算。

对于那些不存在于缓存的结果，Broker节点会将查询转发到正确的历史节点和实时节点中去，一旦历史节点返回结果，Broker节点会将这些结果缓存起来以供以后使用，这个过程如下图所示

注意：实时数据永远不会被缓存，因此查询实时节点的数据的查询请求总是会被转发到实时节点上去。实时数据是不断变化的，因此缓存实时数据是不可靠的

![](/assets/历史数据缓存.png)

上图：结果会为每一个segment缓存。查询会合并缓存结果与历史节点和实时节点的计算结果

缓存也可作为数据可用性的附加级别。在所有历史节点都出现故障的情况下，对于那些命中已经在缓存中缓存了结果的查询，仍然是可以返回查询结果的

可用性：在所有的Zookeeper都中断的情况下，数据仍然是可以查询的。如果Broker节点不可以和Zookeeper进行通信了，它会使用它最后一次得到的整个集群的视图来继续将查询请求转发到历史节点和实时节点，Broker节点假定集群的结构和Zookeeper中断前是一致的。在实践中，在我们诊断Zookeeper的故障的时候，这种可用性模型使得Druid集群可以继续提供查询服务，为我们争取了更多的时间

说明：通常在ShareNothing的架构中,如果一个节点变得不可用了,会有一个服务将下线的这个节点的数据搬迁到其他节点，但是如果这个节点下线后又立即重启,而如果服务在一下线的时候就开始搬迁数据,是会产生跨集群的数据传输,实际上是没有必要的。因为分布式文件系统对同一份数据会有多个副本,搬迁数据实际上是为了满足副本数.而下线又重启的节点上的数据不会有什么丢失的，因此短期的副本不足并不会影响整体的数据健康状况.何况跨机器搬迁数据也需要一定的时间,何不如给定一段时间如果它真的死了,才开始搬迁



