druid是一个为OLAP查询需求而设计的开源大数据系统，druid提供低延时的数据插入，实时的数据查询

druid使用Java开发，基于Jetty提供http rest服务，也提供了Java/Python等语言的工具包

druid是一个集群系统，使用zookeeper做节点管理和事件监控

druid的特点

druid的核心是时间序列，把数据按照时间序列分批存储，十分适合用于对按时间进行统计分析的场景

druid把数据列分为三类：时间戳、维度列、指标列

druid不支持多表连接

druid中的数据一般是使用其他计算框架\(Spark等\)预计算好的低层次统计数据

druid不适合用于处理透视维度复杂多变的查询场景

druid擅长的查询类型比较单一，一些常用的SQL\(groupby 等\)语句在druid里运行速度一般

druid支持低延时的数据插入、更新，但是比hbase、传统数据库要慢很多

druid为什么快

druid在数据插入时按照时间序列将数据分为若干segment，支持低延时地按照时间序列上卷，所以按时间做聚合效率很高

druid数据按列存储，每个维度列都建立索引，所以按列过滤取值效率很高



