为了实现流数据的加载，我们可以通过一个简单http api来向druid推送数据，而tranquility就是一个不错的数据生产组件

下载并安装tranquility

curl -O [http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz](http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz)

tar -xzf tranquility-distribution-0.8.0.tgz

cd tranquility-distribution-0.8.0

druid目录中自带了一个配置文件 conf-quickstart/tranquility/server.json 启动tranquility服务进程， 就可以向druid的 metrics datasource 推送实时数据。

bin/tranquility server -configFile &lt;path\_to\_druid\_distro&gt;/conf-quickstart/tranquility/server.json

这一部分向大家介绍了如何通过tranquility服务来加载流数据， 其实druid还可以支持多种广泛使用的流式框架， 包括Kafka, Storm, Samza, and Spark Streaming等

流数据加载中，维度是可变的，所以在schema定义的时候无需特别指明维度，而是将数据中任何一个字段都当做维度。而该datasource的度量则包含

count

value\_sum \(derived from value in the input\)

value\_min \(derived from value in the input\)

value\_max \(derived from value in the input\)

我们采用了一个脚本，来随机生成度量数据，导入到这个datasource中

bin/generate-example-metrics \| curl -XPOST -H'Content-Type: application/json' --data-binary @- [http://localhost:8200/v1/post/metrics](http://localhost:8200/v1/post/metrics)

执行完成后会返回

{"result":{"received":25,"sent":25}}

这表明http server 从你这里接收到了25条数据，并发送了这25条数据到druid。 在你第一次运行的时候，这个过程需要花一些时间，一段数据加载成功后，就可以查询了。

Query data

接下来就是数据查询了，我们可以采用如下几种方式来查询数据





