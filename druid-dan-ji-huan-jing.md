下载并安装druid

curl -O [http://static.druid.io/artifacts/releases/druid-0.9.1.1-bin.tar.gz](http://static.druid.io/artifacts/releases/druid-0.9.1.1-bin.tar.gz)

tar -xzf druid-0.9.1.1-bin.tar.gz

cd druid-0.9.1.1

文件夹中有如下几个目录：

LICENSE      许可证

bin/            可执行脚本

conf/\*      在集群环境下的配置文件

conf-quickstart/\*        quickstart的配置文件

extensions/\*       druid所有的扩展文件

hadoop-dependencies/\* druid的hadoop扩展文件

lib/\*       druid 依赖的核心软件包

quickstart/\*     quickstart的数据文件

ZK安装

     druid的分布式协同需要依赖zookeeper，所以我们需要安装zk

curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz -o zookeeper-3.4.6.tar.gz

tar -xzf zookeeper-3.4.6.tar.gz

cd zookeeper-3.4.6

cp conf/zoo\_sample.cfg conf/zoo.cfg



./bin/zkServer.sh start

启动druid服务

     启动zk后，我们就可以启动druid的服务了。 首先进入到druid0.9.1.1的根目录，执行

bin/init

     druid会自动创建一个var目录， 内含俩个目录，一个是druid， 用于存放本地环境下hadoop的临时文件，索引日志，segments文件及缓存 和 任务的临时文件。 另一个是tmp用于存放其他临时文件。

      接下来就可以在控制台启动druid服务了。 在单机情况下，我们可以在一台机器上启动所有的druid服务进程，分终端进行。 在分布式生产集群的环境下， druid的服务进程同样也可以在一起启动。

ava \`cat conf-quickstart/druid/historical/jvm.config \| xargs\` -cp "conf-quickstart/druid/\_common:conf-quickstart/druid/historical:lib/\*" io.druid.cli.Main server historical

java \`cat conf-quickstart/druid/broker/jvm.config \| xargs\` -cp "conf-quickstart/druid/\_common:conf-quickstart/druid/broker:lib/\*" io.druid.cli.Main server broker

java \`cat conf-quickstart/druid/coordinator/jvm.config \| xargs\` -cp "conf-quickstart/druid/\_common:conf-quickstart/druid/coordinator:lib/\*" io.druid.cli.Main server coordinator

java \`cat conf-quickstart/druid/overlord/jvm.config \| xargs\` -cp "conf-quickstart/druid/\_common:conf-quickstart/druid/overlord:lib/\*" io.druid.cli.Main server overlord

java \`cat conf-quickstart/druid/middleManager/jvm.config \| xargs\` -cp "conf-quickstart/druid/\_common:conf-quickstart/druid/middleManager:lib/\*" io.druid.cli.Main server middleManager

     druid服务进程启动后，可以在控制台看到相应的日志信息。

