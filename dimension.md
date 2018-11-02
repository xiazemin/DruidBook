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

  注意bitmap不同于之前两个数据结构：前两个数据结构随着数据量的大小线性增长（最坏的情况），bitmap部分的大小是数据大小和列基数的乘积。我们可以对数据进行压缩，因为我们知道列数据的每一行，只会有一个bitmap具有非零项。这意味着高基数列具有高度稀疏的，并且因此高度可压缩的bitmap。Druid采取特别适合于bitmap的压缩算法（例如roaring bitmap 压缩算法）来对它进行压缩。 

       以一个查询为例，select sum\(Characters Added\) from table where timestamp between 2011-01-01T00 and 2011-01-02T00 and Page=Justin Bieber and Gender=Male，首先根据时间段定位到这个Segment，然后查出Justin Bieber的字典编码0，Reach的字典编码1，得到Justin Bieber的bitmap为1100，Male的bitmap为1111，做AND操作得到1100，得到index为0,1，将这两个位置的Characters Added进行相加得到4712。



