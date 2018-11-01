接下来就是数据查询了，我们可以采用如下几种方式来查询数据

Direct Druid queries      直接通过druid查询

```
 druid提供了基于json的富文本查询方式。在提供的示例中，quickstart/wikiticker-top-pages.json  是一个topN的查询实例。
```

curl -L -H'Content-Type: application/json' -XPOST --data-binary @quickstart/wikiticker-top-pages.json [http://localhost:8082/druid/v2/?pretty](http://localhost:8082/druid/v2/?pretty)

Visualizing data 数据可视化

```
      druid是面向用户分析应用的完美方案， 有很多开源的应用支持druid的数据可视化， 如pivot, caravel 和 metabase等
```

SQL and other query libraries 查询组件

有许多查询组件供我们使用，如sql引擎， 还有其他各种语言提供的组件，如python和ruby。 具体如下：



python： druid-io/pydruid



R: druid-io/RDruid



JavaScript: implydata/plywood



7eggs/node-druid-query



Clojure: y42/clj-druid



Ruby: ruby-druid/ruby-druid



redBorder/druid\\_config



SQL: Apache Calcite



implydata/plyql



PHP: pixelfederation/druid-php





