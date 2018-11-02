Druid和Impala、Shark 的比较基本上可以归结为需要设计什么样的系统



Druid被设计用于：



一直在线的服务

获取实时数据

处理slice-n-dice式的即时查询

查询速度不同：



Druid是列存储方式，数据经过压缩加入到索引结构中，压缩增加了RAM中的数据存储能力，能够使RAM适应更多的数据快速存取。索引结构意味着，当添加过滤器来查询，Druid少做一些处理，将会查询的更快。

Impala/Shark可以认为是HDFS之上的后台程序缓存层。 但是他们没有超越缓存功能，真正的提高查询速度。

数据的获取不同：



Druid可以获取实时数据。

Impala/Shark是基于HDFS或者其他后备存储，限制了数据获取的速度。

查询的形式不同：



Druid支持时间序列和groupby样式的查询，但不支持join。

Impala/Shark支持SQL样式的查询。

