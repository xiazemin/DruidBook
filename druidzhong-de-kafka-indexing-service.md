Kafka Indexing Service开发的目的是为了增强Kafka中数据的实时摄入。其特性如下：



保障数据摄入的Exactly Once。

可以摄入任意时间戳的数据，而不仅仅是当前数据。

可以根据Kafka分区的变化二调整任务的数量。

影响数据摄入Exactly Once的主要因素是Kafka的Offset管理。Kafka Indexing Service为了实现Exactly Once语义，去掉了windowPeriod，并采用底层API实现了对Offset的事务管理。



