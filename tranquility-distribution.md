tranquility-distribution，下载地址：

[http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.2.tgz](http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.2.tgz)

 tranquility-distribution提供了Server和Kafka两种通过流来加载数据方式，由于业务环境的因素，对于Server的方式没有做什么研究，感兴趣的伙伴可以前往Druid的官网进行围观。接下来，详细说一下我研究的Kafka加载数据方式。



1.  Druid的数据组成



    无论使用何种方式加载自己的数据，都需要明白Druid的数据组成，因为需要通过配置文件描述加载的数据如进行存储，存储的格式是什么样子的。



    在Druid中数据的存储为三个部分，分别为：timestampSpec、dimensionsSpec和metricsSpec，其中timestampSpec是时间戳字段，这个字段是进行Query时的主要字段，这也是Druid可以作为时序数据库使用的主要因素。dimensionsSpec是维度字段，可以理解为分组条件或MapReduce中的Key。metricsSpec为根据维度对数据进行加工进而产生的数据，比如：count、sum、max、min、first、last等等。

