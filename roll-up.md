roll-up （上卷）是olap的基本操作\(除此之外还有下钻，切片等， 基本理论是一样的\)。  在数据统计里，由于数据量太多，一般对细分的数据不是特别干兴趣，或者说没有太大关注的意义。

但是按照维度的汇总或者统计，确实很有用的。druid通过一个roll-up的处理，将原始数据在注入的时候就进行汇总处理。roll-up 是在维度过滤之前的第一层聚合操作，如下：

GROUP BY timestamp, publisher, advertiser, gender, country

:: impressions = COUNT\(1\),  clicks = SUM\(click\),  revenue = SUM\(price\)

聚合后数据就变成了如下的样子

timestamp             publisher          advertiser  gender country impressions clicks revenue

2011-01-01T01:00:00Z  ultratrimfast.com  google.com  Male   USA     1800        25     15.70

2011-01-01T01:00:00Z  bieberfever.com    google.com  Male   USA     2912        42     29.18

2011-01-01T02:00:00Z  ultratrimfast.com  google.com  Male   UK      1953        17     17.31

2011-01-01T02:00:00Z  bieberfever.com    google.com  Male   UK      3194        170    34.01

我们可以看到，roll-up可以压缩我们需要保存的数据量。 druid 通过roll-up 减少了 我们存储在后台的数据量。  但这种缩减是有损失的， 当我们做了roll-up, 我们就无法查询细分的数据了。 或许，我们在可以在rollup的时候，将其粒度控制在我们可以查询到我们需要查看的最细数据为止。druid可以通过 queryGranularity 来控制注入数据的粒度。 最小的queryGranularity 是 millisecond\(毫秒级\)

  


