## 分布式系统知识点
  
### 数据编码演化：SOA->MSA vs REST vs RPC vs 异步消息代理 vs Actor
  - MSA: MicroService Architecture
  - REST:

### RPC(试图请求远程网络时像调用本地函数一样)
  - Thrift(Facebook)编码格式，自带RPC
  - ProtoBuf(Google)编码格式，gRPC
  - Avro(Hadoop子项目)编码格式，自带RPC
  - RPC 的问题：
    - 本地函数的调用是可预测的; 而网络请求是不可预测的。
    - 本地函数要么返回结果/抛出异常/永远不返回; 网络请求可能会超时，无法知道请求是否成功。
    - 重试网络请求，可能会发生：请求已经完成，只是响应丢失的情况。
    - 网络请求依赖于网络延迟。
    - 本地函数的调用可以传递引用，网络请求则需要传递整个序列化的对象。
    - SOA或REST 的客户端/服务端可以用不用语言实现，RPC框架必须将数据从一种语言转换成另一种。
 
### 消息代理(单向数据流)
  - 与RPC相比的优点：
    - 如果接收方不可用，可以充当缓冲区，提高可靠性。
    - 可以自动将消息重新发送到crash的进程，防止消息丢失。
    - 避免了发送方需要知道接收方的IP，端口的情况。
    - 支持一发多。
    - 在逻辑上将发送方与接收方分离。
  - RabbitMQ
  - ActiveMQ
  - [Kafka(Linkedin)](http://notes.stephenholiday.com/Kafka.pdf)

### Actor模型(单进程中的并发模型)
  - Akka
  - Orleans
  - Erlang OTP
  
### 分布式系统(在多台机器上分布数据)
  - 场景：
    - 扩展：单机负载承受有限
    - 容错：单机故障时系统可以继续工作
    - 延迟：客户遍布，全球布署
  - 复制(Replication) vs 分区(Patitioning)
    - 复制：在多个节点上保存相同数据的复本。
    - 分区：将大块数据扩分成多个小块子集分区。不同的分区分配给不同的节点。

### 复制(数据规模较小)
  - 复制类型：数据同步到其他节点的方式
    - 主从复制: 主写从读
    - 多主复制: 多主节点，一个主节点同时扮演其他主节点的从。使用场景：
      - 多数据中心: 提高可用性和性能。
      - 离线客户端: 比如手机断网后，添加的数据仍然希望在下次联网后同步。
      - 协作编辑: 多个用户同时编辑数据。
    - 无主复制: 任何节点都接受写请求，写请求同时发给所有节点。
  - 同步复制：主节点写完后等从节点确认后才完成。(可以配置为一个从节点是同步，其他从节点是异步)
    - 优点：如果主节点发生故障，从节点可以继续被访问，且服务一致。
    - 缺点：如果从节点无法完成确认(比如当机)，则主节点会被阻塞。
  - 异步复制：无需等待从节点确认后就立即返回。
    - 优点：保证主节点的高可用性(可以一直响应写请求)
    - 缺点：如果主节点当机，则尚未复制到从节点的数据会丢失。
  - 添加新的从节点(先复制快照，再逐步追赶)
  - 节点失效(比如升级内核，或者当机)
    - 从节点失效：追赶式恢复
    - 主节点失效：切换节点
      - 确认节点失效：比如心跳超时(需合理设置超时时间，太长则恢复时花费越久，太短则可能是假失效)
      - 选举新主节点：共识选举
      - 新节点生效：新请求发向新节点，原节点上线后降为从节点(需丢弃当机前未完成复制的写请求)
  - 同步日志
    - ~~基于每条语句~~: 某些环境相关的函数，比如Now()会得到不同结果。 故不采用。
    - ~~基于预写日志~~: 直接复制存储结构的底层数据，比如B-Tree日志。依赖于底层实现，故不采用。
    - ~~基于触发器的复制~~: 数据改变时执行触发器，开销大，一般不采用。
    - 逻辑日志复制(基于行): 逻辑日志与存储逻辑分离。 
  - 同步滞后问题
      - 读取自己写入的数据: 用户提交个人数据后，想立即查看自己的提交，如果从节点尚未同步，则看不到数据。**这时可以直接从主节点读取**。 
      - 单调读: 用户从不同从节点读取数据，刷新页面后得到的数据可能不同。**对于同一个用户应该只使用同一个从节点**。
      - 坚持读取顺序: 对于有顺序要求的数据写入，读取时仍然要按照这个顺序，否则会发生错误。
  - 多主复制时的写冲突
    - 当两个用户同时编辑了wiki的标题，user1改成A->B，user2改成A->C，无法确定使用哪一个，可能有以下解决方式:
      - 基于一个值: 比如写入ID，一个随机数，一个UUID，或者时间戳，数值大的获胜，但是这样容易丢失数据。
      - 合并二值: 比如写入 B/C。
      - 保留冲突: 比如git merge时的字符串
  - 无主复制的节点失效情况
    - 当一个节点失效时，只要一半以上成功写入，可以忽略失败的节点。
    - 当读取时，并行请求多个节点，以版本号确定哪个节点的值更新而使用。
    - 失效节点上线后的数据修复: 读取时检测差异后再修复/后台进程不断反熵修复。
      
### 分区(单机无法存储整个数据集): 将数据分布在多个节点上
  - 不同数据库对分区的称呼:
    - MongoDB | Elasticsearch: Shard
    - HBase: Region
    - Bigtable: tablet
    - Cassandra | Riak: vnode
    - Couchbase: vBucket
  - 分区的目的: 将查询的负载**均衡的分布**在所有节点上
  - 基于主键的范围分区: 比如A-C, D-F, 时间范围等。Bigtable采用这种分区，缺点是有些时候查询会导致热点。
  - 基于主键的Hash分区: 通过Hash函数来划分主键的范围，以避免顺序区域过热。
    - 但是这样仍然无法避免某些热点，比如某些社交媒体上的大v的热点事件，大v的用户id是不变的。
    - 可以在id后加一个随机数，来分散请求，但是这样的后果是需要在应用层合并查询的结果。
  - 分区与二级索引
  - 分区再平衡: 当节点迁移时，需要重新分区使负载均衡。
    - 动态平衡
    - 手动平衡
  - 请求路由: 客户端请求哪个节点?
    - 客户端可以自由选择请求某个节点所在分区，如果不能处理，则转发给下一分区。(Cassandra使用gossip协议同步状态，不依赖ZK)
    - 有一个路由层负责转发。(Linkedin的Espresso进行集群管理，实现路由层)
    - 客户端感知分区与节点的分配关系(订阅消息感知变化，如ZooKeeper)
    
### 事务
  - ACID:
    - Atomicity(原子性): 在出错时中止事务，并回滚已写入的数据。
    - Consistency(前后一致性): 由应用层维护，保持数据有特定的预期，比如账户的贷款余额与贷款余额相等。
    - Isolation(独立隔离性): 并发时访问同一条记录时相互隔离。
    - Durability(持久性): 保证提交的数据写入了磁盘。
  - 单对象写入: 当写入20kb的json数据到数据库中时
    - 如果发送了10kb后断网，数据库是否存储10kb?
    - 如果数据库写入数据时断电，数据库是否存储已写入数据？
    - 如果另一个客户端读取正在写入的的文档，是否看到部分更新的内容？
  - 多对象事务: 多数分布式存储系统不支持多对象事务，因为跨分区时难以实现，甚至带来负面影响。
  - 弱隔离级别: 两个事务并发访问同一条记录时，强隔离会严重影响性能
    - Read committed: 只读取或只覆盖写入已成功提交的数据(给未完成的事务加锁)
      - No dirty reads: 防止读取部分写入但还未提交的数据(加锁会延迟读取，所以可以提供两个版本的数据)
      - No dirty write: 推迟第二个写请求，防止第一个请求被覆盖
  - 快照隔离: 从某个时间点的快照中读取(不用加锁)，而用写锁防止脏写。
  - 防止更新丢失: 两个事务并发写时的冲突(原子写，加锁)
  - 可串行性: 严格按照顺序执行
  - 2PL: 二阶段锁，多个事务可以同时读取一个对象，但是一旦有写操作，必须独占访问

### 分布式的挑战
  - 不可靠的网络(网络，乱序，硬件，超时，阻塞)
  - 不可靠的时钟
  - 节点暂停(垃圾回收，当机)
  
### 一致性
  - 可线性化(Linearizability): 让分布式系统看起来只有一个副本，比如后查询的结果一定是基于先查询的结果。
  - 如何做到线性化？
    - 顺序保证
    - 全序广播
  - [CAP 理论:](http://en.wikipedia.org/wiki/CAP_theorem): 不可能同时做到以下3者
    - 一致性(Consistency): 多个副本间数据的一致
    - 可用性(Availablity): 服务保持可用状态
    - 分区容错性(PartitionTolerance): 在某个分区出现故障时，仍然可以提供服务 
  - 2PC: 2 Phase Commit，阻塞式，需要有一个协调者来协调所有参与者
    1. Requst for Commit(投票阶段) 
    2. Execute Commit(完成阶段)
  - 3PC: 3 Phase Commit，非阻塞式，依赖于一个故障检测器，判断节点是否当机
    1. CanCommit(询问)
    2. PreCommit(预提交)
    3. DoCommit(执行提交)
  - 实践中的分布式事务: **由于难以保证，且有性能问题，所以云服务商基本不支持分布式事务**
  
### 共识: 让节点就某件事情达成一致
  - 使用场景:
    - 主节点选举
    - 原子事务提交: 事务在某些节点成功，在某些节点失败，为了让所有节点的结果一致，要么全部成功，要么全部失败。
  - ZooKeeper: 分布式应用的协调服务，功能包括:
    - 分配任务给节点
    - 发现服务: 找到执行某个服务需要连接的IP地址
    - 成员服务: 检测哪个节点处于活跃状态
  - [Paxos 理论](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf)
    - [算法过程](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)
    - 实现：Chubby(分布式锁服务 by Google)，用于松耦合的分布式系统，[Google-Chubby-PDF](http://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf)
  - ZAB协议: ZooKeeper原子消息广播协议, [与Paxos的比较](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Zab+vs.+Paxos)
    - 实现：ZooKeeper(分布式协调服务 by Yahoo)
  - [Raft 一致性算法](https://raftconsensus.github.io/)
  - Gossip 协议
  - [分布式系统的谬论](http://the-paper-trail.org/blog/distributed-systems-theory-for-the-distributed-systems-engineer/)
  - [拜占廷容错](http://pmg.csail.mit.edu/papers/osdi99.pdf)
  - [拜占廷将军问题](http://bnrg.cs.berkeley.edu/~adj/cs16x/hand-outs/Original_Byzantine.pdf)
  - [分布式一致性的可能性](http://macs.citadel.edu/rudolphg/csci604/ImpossibilityofConsensus.pdf)
  - [Paxos 问题](http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf)
  
### 分布式存储/数据库
  - PostgreSQL(9.0+)
  - MySQL
  - Oracle Data Guard
  - SQLServer AlwaysOn Availability Groups
  - Dynamo(Amazon): 分布式键值数据库，无主复制，非开源。
  - Cassandra: 分布式键值数据库，无主复制，去中心化，非关系型数据库
  - GFS(Google): 分布式文件系统
  - HDFS(GFS的开源实现): 分布式文件系统
  - Hadoop(Google)：分布式存储系统(包括多个组件，比如HDFS，MapReduce，HBASE)
  - BigTable(Google)，分布式表格系统，GFS作为其文件存储系统，使用MapReduce处理海量数据，使用Chubby作为协同服务
  - HBase(BigTable的开源实现)，Hadoop HDFS作为其文件存储系统，使用MapReduce处理海量数据，使用Zookeeper作为协同服务
  - MapReduce(Google): 分布式计算模型
  - CRUSH: Controlled, Scalable, Decentralized Placement of Replicated Data
  
### 日志
  - Log http://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying
  
### 监控系统
  - Dapper(Google)
  
### 编程模型
  - 分布式编程模型 http://web.cs.ucdavis.edu/~pandey/Research/Papers/icdcs01.pdf
  - PSync, http://www.di.ens.fr/~cezarad/popl16.pdf
  - Programming Models for Distributed Computing, http://heather.miller.am/teaching/cs7680/
  - Logic and Lattices for Distributed Programming, http://db.cs.berkeley.edu/papers/UCB-lattice-tr.pdf
  
### 验证分布式系统
  - Jepsen https://github.com/jepsen-io/jepsen
  - Verdi http://verdi.uwplse.org/
  
### 技术博客
  - [How we implemented consistent hashing efficiently](https://blog.ably.io/how-to-implement-consistent-hashing-efficiently-fe038d59fff2)
  - [Notes on Distributed Systems for Young Bloods](http://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/)
  - [High Scalability](http://highscalability.com/)
  - [All Things Distributed](http://www.allthingsdistributed.com/)
  - Distributed Systems: Take Responsibility for Failover http://ivolo.me/distributed-systems-take-responsibility-for-failover/
  - [Files are hard](http://danluu.com/file-consistency/)
  - [On Designing and Deploying Internet-Scale Services]( http://static.usenix.org/event/lisa07/tech/full_papers/hamilton/hamilton_html/)

### 技术框架
  - 负载均衡
    - Nginx：高性能、高并发的web服务器；功能包括负载均衡、反向代理、静态内容缓存、访问控制；工作在应用层
    - LVS： Linux virtual server，基于集群技术和Linux操作系统实现一个高性能、高可用的服务器；工作在网络层
  - WebServer
    - Apache
    - Tomcat
  - 容器
    - Docker
    - Kubernetes (k8s)
  - 缓存
    - Redis
    - MemCache
  - 协调中心
    - ZooKeeper
  - 消息队列
    - Kafka
    - RabbitMQ
  - 实时数据平台
    - akka
    - Storm
  - 离线数据平台
    - Hadoop
    - Spark
  - DB Proxy
    - cobar
  - 搜索
    - ElasticSearch
    - solr
  - 日志
    - rsyslog
    - elk
### 参考
  - 《Design Data-Insentive Application》
  - https://github.com/theanalyst/awesome-distributed-systems
  - https://github.com/aphyr/distsys-class
  - https://www.cnblogs.com/xybaby/p/7787034.html
