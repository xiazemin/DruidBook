Dimension列是不同的，因为它支持filter和group-by的操作，因此每个dimension需要以下三个数据结构：

1、将值（始终被视为String）映射到整数ID的字典。

2、一个列值的list，根据1的字典做编码。

3、对每个列中独立的值，一个bitmap（位图索引，也被称为反向索引）来表示哪些行包含这个值。

为什么要这三个数据结构？字典让2和3中的数据可以被压缩表示。3中的bitmap允许快速过滤操作（具体来说，它很适合快速应用AND和OR操作符）。最后，group by和topN查询需要2中的值的list。

为了得到对这三个数据结构的具体感知，我们考虑上面的例子数据中的page列。表示此维度的三个数据结构如下表所示。

原始数据

page

justin Biebe

justin Biebe

Ke$ha

Ke$ha

字典对值进行编码

{

"Justin Bieber": 0,

"Ke$ha": 1

}

2: 列的数据

\[0,

0,

1,

1\]

3: 位图 – 每个列的独一无二的值

value="Justin Bieber": \[1,1,0,0\]

value="Ke$ha":         \[0,0,1,1\]

