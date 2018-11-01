kafka

kafka 消息缓存作为一个 firehorse（druid 的数据源，输入源）， druid集成的很好。官网上给出的建议是如果在集群部署的话需要自己定制 kafka的comsumer api作为druid的输入源，否则在一致性方面可能会有问题，但是单点 druid和测试用直接用druid集成的 kafka配置就可以了，\[2\]给出了详细的运行步骤。在 druid要运行realtime 和historical必须要让它们知道你的数据源，数据的格式等信息，这些信息由 .spec为后缀的文件来指定，也是我们定制自己的数据源的重要文件，详细的各个字段在文档 \[3\]中dataSchema 都有提到。以 druid自带的examples/indexing/wikipedia.spec 文件来解释几个重要的字段。

dataSource指定数据源的名字，可以自定义，后面在做查询的时候你的 json文件需要提供一个dataSource数据源的字段，就是你所写的这个值。

将 zookeeper.connect修改为你自己的zookeeper的主机号和端口号， feed等。feed 这个字段非常重要，它就是指定去 kafka中哪个topic 中拉取数据的 topic名，必须跟flume 输出的topic， kafka作为缓存的topic 是同一个。另外一些参数是性能调优方面的，即箭头所指的持久化周期大小和 realtime服务的窗口大小， 没列举出来的可调优的字段，其他配置 dimensionsSpec和metricsSpec 等属性的都可以在文档 中找到。

1\) flume -&gt; kafka: kafkaSink程序导出jar包拷贝至flume/lib下；修改agent配置文件；创建topic

2\) kafka -&gt; druid: 修改${DRUID\_HOME}/examples/indexing/wikipedia.spec文件，feed字段修改位kafka topic名，另外还有一些zookeeper地址之类的具体信息

3\) 启动zookeeper服务

4\) 启动kafka server服务

5\) 启动flume agent服务

6\) 启动druid realtime（配置文件要指向firehorse为kafka的文件）

7\) 用flume avro-client发送数据

8\) 用curl命令发送查询请求

在Druid目录下执行如下指令：

bin/generate-example-metrics

在kafka目录下执行：

./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic metrics

现在kafka-console-producer就开始等待数据的输入了，复制刚生成的示例数据并粘贴到kafka-console-producer控制台终端，回车确认。当然也可以复制更多数据到终端，或者CTRL-D退出。

现在就可以进行数据查询了，当然也可以参考下文去加载自定义数据集。

数据查询

数据发送完成后就可以进行数据查询了，使用方法详见 supported query methods.

加载自定义数据

目前为止，我们已经按照Druid发布版本中的数据提取规范，将数据从kafka加载到了Druid。每一个数据提取规范都是为了特定的数据集设计的，也可以通过自定义提取规范来加载自定义数据。

自定义数据提取规范，可以按需修改conf-quickstart/tranquility/kafka.json配置文件

dataSchema，使用的数据集名称

timestampSpec，哪个是时间字段

dimensionsSpec，哪些能作为维度字段

metricsSpec，哪些能作为度量进行计算

使用网页浏览为例并将输入发送到pageviews的topic里，示例数据如下：



{"time": "2000-01-01T00:00:00Z", "url": "/foo/bar", "user": "alice", "latencyMs": 32}

首先创建topic



./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic pageviews

修改conf-quickstart/tranquility/kafka.json配置文件，修改后：



{

  "dataSources" : {

    "metrics-kafka" : {

      "spec" : {

        "dataSchema" : {

          "dataSource" : "pageviews-kafka",

          "parser" : {

            "type" : "string",

            "parseSpec" : {

              "timestampSpec" : {

                "column" : "time",

                "format" : "auto"

              },

              "dimensionsSpec" : {

                "dimensions" : \["url", "user"\],

                "dimensionExclusions" : \[

                  "timestamp",

                  "value"

                \]

              },

              "format" : "json"

            }

          },

          "granularitySpec" : {

            "type" : "uniform",

            "segmentGranularity" : "hour",

            "queryGranularity" : "none"

          },

          "metricsSpec" : \[

            {

              "name": "views",

             "type": "count"

            },

           {

              "name": "latencyMs", 

              "type": "doubleSum",

              "fieldName": "latencyMs"

            }

          \]

        },

        "ioConfig" : {

          "type" : "realtime"

        },

        "tuningConfig" : {

          "type" : "realtime",

          "maxRowsInMemory" : "100000",

          "intermediatePersistPeriod" : "PT10M",

          "windowPeriod" : "PT10M"

        }

      },

      "properties" : {

        "task.partitions" : "1",

        "task.replicants" : "1",

        "topicPattern" : "pageviews"

      }

    }

  },

  "properties" : {

    "zookeeper.connect" : "localhost",

    "druid.discovery.curator.path" : "/druid/discovery",

    "druid.selectors.indexing.serviceName" : "druid/overlord",

    "commit.periodMillis" : "15000",

    "consumer.numThreads" : "2",

    "kafka.zookeeper.connect" : "localhost",

    "kafka.group.id" : "tranquility-kafka"

  }

}

下面启动Druid的kafka提取服务：



bin/tranquility kafka -configFile ../druid-0.9.2/conf-quickstart/tranquility/kafka.json

如果Tranquility或者kafka已经启动，可以停止并重新启动。

最后将数据发送到kafka的topic，以下面这些数据为例：



{"time": "2000-01-01T00:00:00Z", "url": "/foo/bar", "user": "alice", "latencyMs": 32}

{"time": "2000-01-01T00:00:00Z", "url": "/", "user": "bob", "latencyMs": 11}

{"time": "2000-01-01T00:00:00Z", "url": "/foo/bar", "user": "bob", "latencyMs": 45}

Druid流处理需要相对当前（准实时）的数据，相而言windowPeriod值控制的是更宽松的时间窗口（也就是流处理会检查数据timestamp的值，而时间窗口只关注数据接收的时间）。所以需要将2000-01-01T00:00:00Z转换为ISO8601格式的当前系统时间，你可以用以下命令转换：



python -c 'import datetime; print\(datetime.datetime.utcnow\(\).strftime\("%Y-%m-%dT%H:%M:%SZ"\)\)'

更新上述JSON中的时间戳，然后将这些消息复制并粘贴到此kafka-console-producer，然后按Enter键：



./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic pageviews

就这样，数据应该已经保存在Druid里了，可以使用任何Druid支持的查询方式查询这些数据了。

http://druid.io/docs/0.9.2/ingestion/stream-ingestion.html

