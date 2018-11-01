服务启动之后，我们就可以将数据load到druid中进行查询了。在druid0.9.1.1的安装包中，自带了2015-09-12的wikiticker数据。我们可以用此数据来作为我们druid的学习实例。

```
 首先我们看一下wikipedia的数据， 除了时间之外，包含的维度\(dimensions\)有：
```

channel

cityName

comment

countryIsoCode

countryName

isAnonymous

isMinor

isNew

isRobot

isUnpatrolled

metroCode

namespace

page

regionIsoCode

regionName

user

度量\(measures\) 我们可以设置如下：

count

added

deleted

delta

user\_unique

确定了度量，维度之后，接下来我们就可以导入数据了。首先，我们需要向druid提交一个注入数据的任务，并将目录指向我们需要加载的数据文件wikiticker-2015-09-12-sampled.json

Druid是通过post请求的方式提交任务的， 上面我们也讲过，overload node 用于数据的加载，所以需要在overload节点上执行post请求， 目前单机环境，无需考虑这个。

在druid根目录下执行

curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/wikiticker-index.json localhost:8090/druid/indexer/v1/task

其中wikiticker-index.json 文件指明了数据文件的位置，类型，数据的schema\(如度量，维度，时间，在druid中的数据源名称等\)等信息

 当控制台打印如下信息后，说明任务提交成功

{"task":"index\_hadoop\_wikipedia\_2013-10-09T21:30:32.802Z"}

     可以在overload控制台 http://localhost:8090/console.html来查看任务的运行情况， 当状态为“SUCCESS”时， 说明任务执行成功。

     当数据注入成功后，historical node会加载这些已经注入到集群的数据，方便查询，这大概需要花费1-2分钟的时间。 你可以在coordinator 控制台http://localhost:8081/\#/来查看数据的加载进度

当名为wikiticker的datasource 有个蓝色的小圈，并显示fully available时，说明数据已经可以了。可以执行查询操作了。

