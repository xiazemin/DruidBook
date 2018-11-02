索引服务也可以产生Segment文件，相对于实时节点，索引节点主要包括以下优点：

1. 除了支持Pull的方式摄取数据，还支持Push的方式

2. 可以通过API的方式定义任务配置

3. 可以更灵活的使用系统资源

4. 可以控制segment副本数量的控制

5. 可以灵活的完成和segment数据文件相关的操作

6. 提供可扩展以及高可用的特性

4.5.1 主从结构的架构

Indexing Service是高可用、分布式、Master/Slave架构服务。主要由三类组件构成：负责运行索引任务\(indexing task\)的Peon，负责控制Peon的MiddleManager，负责任务分发给MiddleManager的Overlord；三者的关系可以解释为：Overlord是MiddleManager的Master，而MiddleManager又是Peon的Master。其中，Overlord和MiddleManager可以分布式部署，但是Peon和MiddleManager默认在同一台机器上，架构图如下：

![](/assets/overlord.png)

4.5.2 统治节点\(Overload\)

Overlord负责接受任务、协调任务的分配、创建任务锁以及收集、返回任务运行状态给调用者。当集群中有多个Overlord时，则通过选举算法产生Leader，其他Follower作为备份。

Overlord可以运行在local（默认）和remote两种模式下，如果运行在local模式下，则Overlord也负责Peon的创建与运行工作，当运行在remote模式下时，Overlord和MiddleManager各司其职，根据上图所示，Overlord接受实时/批量数据流产生的索引任务，将任务信息注册到Zookeeper的/task目录下所有在线的MiddleManager对应的目录中，由MiddleManager去感知产生的新任务，同时每个索引任务的状态又会由Peon定期同步到Zookeeper中/Status目录，供Overlord感知当前所有索引任务的运行状况。

Overlord对外提供可视化界面，通过访问[http://:/console.html，我们可以观察到集群内目前正在运行的所有索引任务、可用的Peon以及近期Peon完成的所有成功或者失败的索引任务。](http://:/console.html，我们可以观察到集群内目前正在运行的所有索引任务、可用的Peon以及近期Peon完成的所有成功或者失败的索引任务。)

4.5.3 MiddleManager

MiddleManager负责接收Overlord分配的索引任务，同时创建新的进程用于启动Peon来执行索引任务，每一个MiddleManager可以运行多个Peon实例。

在运行MiddleManager实例的机器上，我们可以在${ java.io.tmpdir}目录下观察到以XXX\_index\_XXX开头的目录，每一个目录都对应一个Peon实例；同时restore.json文件中保存着当前所有运行着的索引任务信息，一方面用于记录任务状态，另一方面如果MiddleManager崩溃，可以利用该文件重启索引任务。

4.5.4 Peon

Peon是Indexing Service的最小工作单元，也是索引任务的具体执行者，所有当前正在运行的Peon任务都可以通过Overlord提供的web可视化界面进行访问

