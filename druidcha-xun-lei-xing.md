Druid在不同场景下，有很多的查询类型。对于各种类型的查询类型的配置可以json属性文件设置。Druid查询类型，概括一下为3大类：

聚合查询 - 时间序列查询（Timeseries）、排名查询（TopN）、分组查询（GroupBy）

元数据查询 - 时间范围\(Time Boundary\) 、段元数据\(Segment Metadata\)、数据源\(Datasource\)

Search查询 - Search以聚合查询为主，与其它查询类型比较相对简单，使用上相对比较少，暂不介绍。

如何对查询进行选择呢？

在可能的情况下，我们建议使用的时间序列和TopN查询代替分组查询，分组查询是Druid最灵活的的查询，但是性能最差。时间序列查询是明显快于GROUPBY查询，因为聚合不需要分组尺寸。对于分组和排序在一个单一的维度，TopN查询更优于GROUPBY。

Druid Json查询属性

Druid json查询比较重要的几个属性是：queryType、dataSource、granularity、filter、aggregator等。

查询类型（queryType）：对应聚合查询下的3种类型值：timeseries、topN、groupBy

数据源（dataSource）：数据源，类似数据库中表的概念，对应数据导入时Json配置属性dataSource值。

聚合粒度（granularity）： 粒度决定如何得到数据块在跨时间维度，在配置查询聚合粒度里有三种配置方法：

简单聚合粒度 - 支持字符串值有：all、none、second、minute、five\_minute，ten\_minute，fifteen\_minute、thirty\_minute、hour、day、week、month、quarter、year，all - 将所有块变成一块， none - 不使用块数据（它实际上是使用最小索引的粒度，none意味着为毫秒级的粒度）；按时间序列化查询时不建议使用none，因为所有的毫秒不存在，系统也将尝试生成0值，这往往是很多。

时间段聚合粒度 - Druid指定一精确的持续时间（毫秒）和时间戳返回UTC（世界标准时间）。

常用时间段聚合粒度 - 与时间段聚合粒度差不多，但是常用时间指平时我们常用时间段，如年、月、周、小时等。下面对3种聚合粒度配置举例说明。

简单聚合粒度

查询粒度比数据采集时配置的粒度小，则不合理，也无意义，因较小粒度（相比）者无索引数据；如查询粒度小于采集时配置的查询粒度时，则Druid的查询结果与采集数据配置的查询粒度结果一样。

时间段聚合粒度

指定一个精确时间持续时长（毫秒表示\)，返回UTC时间；支持可选项属性origin，不指定时默认开始时间（1970-01-01T00:00:00Z）



/\*\*持续时间段2小时，从1970-01-01T00:00:00Z开始\*/  

{"type": "duration", "duration": 7200000}

 

/\*\*持续时间1小时，从origin开始\*/  

{"type": "duration", "duration": 3600000, "origin": "2012-01-01T00:30:00Z"}

过滤（Filters）

等价于sql 查询的where。也是支持and，or，in，not等。



"filter": { "type": "selector", "dimension": &lt;dimension\_string&gt;, "value": &lt;dimension\_value\_string&gt; }

聚合（Aggregations）

聚合类型如下：Count aggregator、Sum aggregators、Min / Max aggregators、Approximate Aggregations、Miscellaneous Aggregations



/\*\*Druid进行Count查询的数据量并不一定等于数据采集时导入的数据量，因为Druid在采集数据并导入时已经对数据进行了聚合\*/  

{ "type" : "count", "name" : &lt;output\_name&gt; }  

 

/\*\*longSumaggregator：计算值为有符号位64位整数\*/

{ "type" : "longSum", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }  

 

/\*\*doubleSum aggregator：与longSum类似，计算值为64位浮点型\*/

{ "type" : "doubleSum", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }  

 

/\*\* doubleMin aggregator \*/

{ "type" : "doubleMin", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }

 

/\*\*doubleMax aggregator\*/

{ "type" : "doubleMax", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }

 

/\*\*longMin aggregator\*/

{ "type" : "longMin", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }  

 

/\*\* longMax aggregator\*/

{ "type" : "longMax", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }

类似聚合（Approximate Aggregations）

基数聚合（Cardinality aggregator）

计算Druid多种维度基数，Cardinality aggregator使用HyperLogLog评估基数，这种聚合比带有索引的

hyperUnique聚合慢；一般我们强力推荐使用hyperUniqueaggregator而不是Cardinality aggregator，格式如下：



{  

  "type": "cardinality",  

  "name": "&lt;output\_name&gt;",  

  "fieldNames": \[ &lt;dimension1&gt;, &lt;dimension2&gt;, ... \],  

  "byRow": &lt;false \| true&gt; \# \(optional, defaults to false\)  

}

维度值聚合-当设置属性byRow为false（默认值）时，通过合并所有给定的维度列来计算值集合。单维度等价于：



SELECT COUNT\(DISTINCT\(dimension\)\) FROM &lt;datasource&gt;

对于多维度，等价如下:



SELECT COUNT\(DISTINCT\(value\)\) FROM \(  

  SELECT dim\_1 as value FROM &lt;datasource&gt;  

  UNION  

  SELECT dim\_2 as value FROM &lt;datasource&gt;  

  UNION  

  SELECT dim\_3 as value FROM &lt;datasource&gt;

行聚合-当设置属性byRow为true时，根所不同维度的值合并来计算行值，等价如下：



SELECT COUNT\(\*\) FROM \( SELECT DIM1, DIM2, DIM3 FROM &lt;datasource&gt; GROUP BY DIM1, DIM2, DIM3 \)

HyperUnique aggregator

“hyperunique”在创建索引时聚合的维度值使用HyperLogLog计算估计，更多资料请参考官网：



{ "type" : "hyperUnique", "name" : &lt;output\_name&gt;, "fieldName" : &lt;metric\_name&gt; }

后聚合\(post-aggregators\)

后聚合是对Druid进行聚合后的值进行聚全，如果查询中包括一个后聚合，那么确保所有聚合满足后聚合要求；后聚合有以下几种类型：



Arithmetic post-aggregators

Field accessor post-aggregator

Constant post-aggregator

JavaScript post-aggregator

HyperUnique Cardinality post-aggregator

Arithmetic post-aggregators

算术后聚合应用已提供的函数从左到右获取字段，这些字段可聚合或后聚合；支持+, -, \*, /, and quotient。



算术后聚合语法如下：



postAggregation : {  

  "type"  : "arithmetic",  

  "name"  : &lt;output\_name&gt;,  

  "fn"    : &lt;arithmetic\_function&gt;,  

  "fields": \[&lt;post\_aggregator&gt;, &lt;post\_aggregator&gt;, ...\],  

  "ordering" : &lt;null \(default\), or "numericFirst"&gt;  

}

时间序列查询\(Timeseries\)

这些类型的查询以时间序列查询对象和返回一个JSON数组对象，每个对象表示时间序列查询的值，时间序列查询请求的Json的7个主要属性如下：



