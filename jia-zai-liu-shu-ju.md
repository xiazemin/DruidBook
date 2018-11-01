为了实现流数据的加载，我们可以通过一个简单http api来向druid推送数据，而tranquility就是一个不错的数据生产组件

     下载并安装tranquility

curl -O http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz

tar -xzf tranquility-distribution-0.8.0.tgz

cd tranquility-distribution-0.8.0

      druid目录中自带了一个配置文件 conf-quickstart/tranquility/server.json  启动tranquility服务进程， 就可以向druid的 metrics datasource 推送实时数据。

bin/tranquility server -configFile &lt;path\_to\_druid\_distro&gt;/conf-quickstart/tranquility/server.json

这一部分向大家介绍了如何通过tranquility服务来加载流数据， 其实druid还可以支持多种广泛使用的流式框架， 包括Kafka, Storm, Samza, and Spark Streaming等

