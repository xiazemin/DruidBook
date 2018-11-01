Druid 是目前比较流行的高性能的，分布式列存储的OLAP框架\(具体来说是MOLAP\)。它有如下几个特点：

一. 亚秒级查询

```
 druid提供了快速的聚合能力以及亚秒级的OLAP查询能力，多租户的设计，是面向用户分析应用的理想方式。
```

二.实时数据注入

```
 druid支持流数据的注入，并提供了数据的事件驱动，保证在实时和离线环境下事件的实效性和统一性
```

三.可扩展的PB级存储

```
 druid集群可以很方便的扩容到PB的数据量，每秒百万级别的数据注入。即便在加大数据规模的情况下，也能保证时其效性
```

四.多环境部署

```
 druid既可以运行在商业的硬件上，也可以运行在云上。它可以从多种数据系统中注入数据，包括hadoop，spark，kafka，storm和samza等
```

五.丰富的社区

```
 druid拥有丰富的社区，供大家学习。
```

Data

   druid的数据格式和关系型数据库数据较为类似， 如下：

timestamp             publisher          advertiser  gender  country  click  price

2011-01-01T01:01:35Z  bieberfever.com    google.com  Male    USA      0      0.65

2011-01-01T01:03:63Z  bieberfever.com    google.com  Male    USA      0      0.62

2011-01-01T01:04:51Z  bieberfever.com    google.com  Male    USA      1      0.45

2011-01-01T01:00:00Z  ultratrimfast.com  google.com  Female  UK       0      0.87

2011-01-01T02:00:00Z  ultratrimfast.com  google.com  Female  UK       0      0.99

2011-01-01T02:00:00Z  ultratrimfast.com  google.com  Female  UK       1      1.53

熟悉OLAP的同学，对以下这些概念一定不陌生，druid也把数据分为以下三个部分：

Timestamp Column：将时间单独处理，是因为druid所有的操作都是围绕时间轴来进行的。

Dimension Columns：维度字段，是数据的属性， 一般被用来过滤数据。上面的例子，我们有四个维度, publisher, advertiser, gender, country.  他们每一个都可以看是数据立方体的一个轴，都可以用来用来做横切。

Metric Columns: 度量字段，是用来做聚合或者相关计算的。 上边的数据， click和price是俩个度量。度量是可以衡量的数据，一般可以有如下的操作，count ，sum等等

