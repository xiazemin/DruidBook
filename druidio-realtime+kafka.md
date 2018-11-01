kafka

kafka 消息缓存作为一个 firehorse（druid 的数据源，输入源）， druid集成的很好。官网上给出的建议是如果在集群部署的话需要自己定制 kafka的comsumer api作为druid的输入源，否则在一致性方面可能会有问题，但是单点 druid和测试用直接用druid集成的 kafka配置就可以了，\[2\]给出了详细的运行步骤。在 druid要运行realtime 和historical必须要让它们知道你的数据源，数据的格式等信息，这些信息由 .spec为后缀的文件来指定，也是我们定制自己的数据源的重要文件，详细的各个字段在文档 \[3\]中dataSchema 都有提到。以 druid自带的examples/indexing/wikipedia.spec 文件来解释几个重要的字段。

dataSource指定数据源的名字，可以自定义，后面在做查询的时候你的 json文件需要提供一个dataSource数据源的字段，就是你所写的这个值。

将 zookeeper.connect修改为你自己的zookeeper的主机号和端口号， feed等。feed 这个字段非常重要，它就是指定去 kafka中哪个topic 中拉取数据的 topic名，必须跟flume 输出的topic， kafka作为缓存的topic 是同一个。另外一些参数是性能调优方面的，即箭头所指的持久化周期大小和 realtime服务的窗口大小， 没列举出来的可调优的字段，其他配置 dimensionsSpec和metricsSpec 等属性的都可以在文档 中找到。

