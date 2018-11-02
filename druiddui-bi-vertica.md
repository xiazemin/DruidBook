怎么比较Druid和Vertica?



Vertica 类似与之前介绍的ParAccel/Redshift\(Druid-vs-Redshift\). 不是实时注入数据； 提供SQL的全部语法支持



另外一个很大不同是: Vertica 不适用index, 尝试利用run-length encoding和其他的压缩技术和产生不同排序的实体化副本投射系统\(最大化利用run-length encoding\)



不太清楚Vertica如何分发和复制数据， 所以很难说两者有什么不同

