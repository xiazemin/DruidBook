Mondrian是一个开源项目。一个用[**Java**](http://lib.csdn.net/base/java)写成的OLAP引擎。它用MDX语言实现查询，从关系[**数据库**](http://lib.csdn.net/base/mysql)\(RDBMS\)中读取数据。然后经过[**java**](http://lib.csdn.net/base/java)API以多维的方式对结果进行展示。

Mondrian的使用方式同JDBC驱动类似。可以非常方便的与现有的Web项目集成

## 1.1 Mondrian的体系结构\(Architecture\)

Mondrian OLAP 系统由四个层组成; 从最终用户到数据中心, 顺序为:   
1.1.1 表现层\(the presentation layer\)  
1.1.2 维度层\(the dimensional layer\)  
1.1.3 集合层\(the star layer\)  
1.1.4 存储层\(the storage layer\)

![](/assets/mondrian.png)

