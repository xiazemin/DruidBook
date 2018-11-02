tranquility-distribution，下载地址：

[http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.2.tgz](http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.2.tgz)

tranquility-distribution提供了Server和Kafka两种通过流来加载数据方式，由于业务环境的因素，对于Server的方式没有做什么研究，感兴趣的伙伴可以前往Druid的官网进行围观。接下来，详细说一下我研究的Kafka加载数据方式。

1. Druid的数据组成

无

论使用何种方式加载自己的数据，都需要明白Druid的数据组成，因为需要通过配置文件描述加载的数据如进行存储，存储的格式是什么样子的。

在Druid中数据的存储为三个部分，分别为：timestampSpec、dimensionsSpec和metricsSpec，其中timestampSpec是时间戳字段，这个字段是进行Query时的主要字段，这也是Druid可以作为时序数据库使用的主要因素。dimensionsSpec是维度字段，可以理解为分组条件或MapReduce中的Key。metricsSpec为根据维度对数据进行加工进而产生的数据，比如：count、sum、max、min、first、last等等。

原始数据是这个样子的

{"unit": "milliseconds", "http\_method": "GET", "value": 70, "timestamp": "2017-11-28T03:16:20Z", "http\_code": "200", "page": "/list", "metricType": "request/latency", "server": "www2.example.com"}

{"unit": "milliseconds", "http\_method": "GET", "value": 116, "timestamp": "2017-11-28T03:16:20Z", "http\_code": "200", "page": "/list", "metricType": "request/latency", "server": "www4.example.com"}

对这些数据进行加工，其中timestampSpec为timestamp，dimensionsSpec包括page、server，metricsSpec为count和value的sum，加工之后Druid中存储的数据格式将变成下面的样子

{"timestamp":"2017-11-28T03:16:20Z", "page":"/list", "server":"www2.example.com", "count":1, "valueSum":70}

{"timestamp":"2017-11-28T03:16:20Z", "page":"/list", "server":"www4.example.com", "count":2, "valueSum":253}

2.通过Kafka加载数据，并完成上面例子的加工操作

```
在导入数据的过程中，我们需要对获取的数据如何加工进行配置，具体kafka.json配置如下：
```

{

"dataSources" : {

```
"test1" : { --数据源名称，与下方的dataSource的值要完全相同

  "spec" : { --开始描述

    "dataSchema" : { --开始数据表描述

      "dataSource" : "test1", --数据源名称，注意上下对应

      "parser" : {

        "type" : "string",

        "parseSpec" : {

          "timestampSpec" : {

            "column" : "timestamp",

            "format" : "auto"

          },

          "dimensionsSpec" : { --维度字段都有那些

            "dimensions" : \["page", "server"\]

          },

          "format" : "json" --格式化工具类型，对应从kafka中加载的数据的格式

        }

      },

      "granularitySpec" : { --合并、分卷设置

        "type" : "uniform",

        "segmentGranularity" : "hour", --按照小时进行分卷设置

        "queryGranularity" : "none" --默认合并模式，采用按秒进行合并的方式进行

      },

      "metricsSpec" : \[ --合并计算列

        {

          "type" : "count", --count操作

          "name" : "count" --列名

        },

        {

          "type" : "longSum", --对long型数据的sum操作，当然还有doubleSum

          "name" : "value\_sum", --列名

          "fieldName" : "value" --需要进行Sum的属性名

        }

      \]

    },

    "ioConfig" : {

      "type" : "realtime" --实时加载

    },

    "tuningConfig" : {

      "type" : "realtime",

      "maxRowsInMemory" : "100000",  --内存中最大保存行号

      "intermediatePersistPeriod" : "PT10M", --内存提交到磁盘的时间间隔

      "windowPeriod" : "PT10M" --时间窗口缓冲时间

    }

  },

  "properties" : {

    "task.partitions" : "1",

    "task.replicants" : "1",

    "topicPattern" : "sunz"

  }

}
```

},

"properties" : {

```
"zookeeper.connect" : "localhost",  --Druid使用的Zookeeper集群所在主机

"druid.discovery.curator.path" : "/druid/discovery",

"druid.selectors.indexing.serviceName" : "druid/overlord",

"commit.periodMillis" : "15000",

"consumer.numThreads" : "2",

"kafka.zookeeper.connect" : "devhadoop241", --kafka所使用的Zookeeper所在主机

"kafka.group.id" : "tranquility-kafka" --Topic名称
```

}

}

启动方式比较简单，只需要执行

bin/tranquility kafka -configFile conf/kafka.json

启动tranquility之后，就可以通过kafka加载数据了，如何建立Kafka生产者这个就不再赘述。



3.关于分段、聚合、内存提交及时间窗口四个时间相关设置的详解

在

上面的配置中，有四个属性的配置是至关重要的，分别为segmentGranularity--数据分段/分卷时间，queryGranularity数据碰撞时间、intermediatePersistPeriod内存持续时间/提交硬盘持久化时间间隔，windowPeriod窗口期/过期数据延迟加载周期。







segmentGranularity







Druid的数据是分卷储存的，通过这个属性，我们可以设置具体的分卷周期，比如：按分钟、小时、天、周、月、年进行分卷，具体分卷方式可以根据具体业务情况和数据规模进行设置。







queryGranularity







数据碰撞周期，这个周期所设置的是在何种时间颗粒度上对数据进行聚合操作，默认设置为none，如果设置为none则根据数据源中所设置时间戳字段进行聚合，如果时间戳一样则自动聚合，当然也可以设置为分钟和小时等，当然这个时间的设置一定要小于数据分卷周期







intermediatePersistPeriod







加载的数据在内存中驻守时间长度，加载的数据由于聚合需要，会先放置在内存中，一段时间之后统一提交到磁盘进行持久化操作，这个属性是影响性能的重要属性，设置的周期过大，内存消耗严重，导致假死，设置的过小，导致每次聚合都将操作磁盘，效率低下







windowPeriod







时间窗口期的作用是对数据延迟加载的设置，比如我们的数据分卷方式是按照小时进行分卷的，那么在只能realtime模式下，只能处理当前时段的数据，也就是意味着比如现在是12：04，那么只能处理12:00--13:00之间产生的数据，而由于11:00--12:00访问量过大，导致上一时段的数据还没有处理完，这部分数据将被抛弃，为了防止此类的数据丢失，我们设置时间窗口，对处理数据的时段设置进行缓冲，比如上一配置中，设置的是10分钟的时间窗口，也就意味着，在12:10之前，依然可以处理上一时段的数据。







四属性的设置策略







从上面的配置的作用可以看出，以上四属性的配置，要遵循 segmentGranularity &gt; queryGranularity &gt;= intermediatePersistPeriod &gt;= windowPeriod的方式来进行配置才能保证处理性能。当然也需要衡量查询需求、数量吞吐量和硬件资源。





