Tranquility是托管在GitHub上的开源Scala Library，主要负责协调实时索引任务中创建Indexing Service Tasks、处理partition、副本、服务发现以及更新schema。在集群中，我们可以启动多个Tranquility-Kafka实例，所有实例均通过Zookeeper协同处理Indexing Service Tasks。



Tranquility的出现主要是因为Indexing Service API更偏向底层\(low-level\)，就如同Kafka Producer和Consumer在low-level API\(Scala API\)的基础上又封装了high-level API\(Java API\)，供开发者使用。



任务生命周期



Tranquility会为时间窗口内的每一个Segment启动一个Indexing Service Task，其中Tranquility将数据以POST请求 的方式提交给EventReceiverFirehose\(Firehose实现类，默认丢弃所有时间窗口外的数据\)，当到达任务最大时长（SegmentGranularity+WindowPeriod）时，TimedShutoffFirehose会自动关闭Firehose，此时Segment会进行合并、注册元信息、存储到Deep Storage中并等待Handoff，当某个Historical Node声明自己已加载该Segment后，Indexing Service Task会正常退出。所以，每个Indexing Service Task的生命周期包括SegmentGranularity + WindowPeriod+push to Deep Storage + wait forHandoff。



Schema更新 

Schema更新表示我们增加或减少了原始数据中的维度数或度量数。Tranquility可以自动检测Schema更新，并保存新老两份Schema，对于先前创建的任务依然使用老Schema，当到达新的SegmentGranularity时，Tranquility则会使用新Schema摄取数据。



高可用性 

Tranquility的所有操作都是尽最大努力\(best-effort\)，我们可以通过配置多个任务副本保证数据不丢失，但是在某些情况下，数据可能会丢失或出现重复：



早于或晚于时间窗口，数据一定会被丢弃。



失败的Middle Manager数目多于配置的任务副本数，部分数据可能会丢失。



IndexingService内部（Overlord、MiddleManager、Peon）通信长时间丢失，同时重试次数超过最大上限，或者运行周期已经超过了时间窗口，这种情况部分数据也会被丢弃。



Tranquility一直未收到IndexingService的确认请求，那么Tranquility会切换到批量加载模式，数据可能会出现重复。



所以，Druid官方建议，如果使用Tranquility作为Real-TimeNodes，那么可以采用如下解决方案减少数据丢失或者重复的风险，从而保证Druid中数据的exactly-oncesemantics：



将数据备份到S3或者HDFS等存储中；



晚间对备份数据运行Hadoopbatchindexingjob，从而对白天的数据重做Segment。



