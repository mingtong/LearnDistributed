## 分布式系统知识点

### 原理
  - [CAP 理论:](http://en.wikipedia.org/wiki/CAP_theorem)
    - 一致性(Consistency): 多个副本间数据的一致
    - 可用性(Availablity): 服务保持可用状态
    - 分区容错性(PartitionTolerance): 在某个分区出现故障时，仍然可以提供服务 
  - 2PC: 2 Phase Commit
    1. Requst for Commit(投票阶段) 
    2. Execute Commit(完成阶段)
  - 3PC: 3 Phase Commit
    1. CanCommit(询问)
    2. PreCommit(预提交)
    3. DoCommit(执行提交)
  - [Paxos 理论](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf)
    - [算法过程](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)
    - 实现：Chubby(分布式锁服务 by Google)，用于松耦合的分布式系统，[Google-Chubby-PDF](http://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf)
  - ZAB: ZK原子消息广播协议, [与Paxos的比较](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Zab+vs.+Paxos)
    - 实现：ZooKeeper(分布式协调服务 by Yahoo)
  - [Raft 一致性算法](https://raftconsensus.github.io/)
  - [分布式系统的谬](http://the-paper-trail.org/blog/distributed-systems-theory-for-the-distributed-systems-engineer/)
  - [拜占廷容错] http://pmg.csail.mit.edu/papers/osdi99.pdf
  - [拜占廷将军问题] http://bnrg.cs.berkeley.edu/~adj/cs16x/hand-outs/Original_Byzantine.pdf
  - [分布式一致性的可能性] http://macs.citadel.edu/rudolphg/csci604/ImpossibilityofConsensus.pdf
  - [Paxos 问题] http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf
  
### 分布式存储、数据库
  - Dynamo: Amazon. http://bnrg.eecs.berkeley.edu/~randy/Courses/CS294.F07/Dynamo.pdf
  - Bigtable: Google. http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf
  - GFS: Google. http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/archive/gfs-sosp2003.pdf http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.161.6751&rep=rep1&type=pdf
  - Cassandra: A Decentralized Structured Storage System.
  - CRUSH: Controlled, Scalable, Decentralized Placement of Replicated Data. http://www.ssrc.ucsc.edu/Papers/weil-sc06.pdf

### SOA->MSA vs REST vs RPC vs 消息代理
  - MSA: MicroService Architecture
  - REST:
  - RPC:

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
  
### 日志
  - Log http://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying
  
### 监控系统
  - Dapper, Google. http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36356.pdf
  
### 编程模型
  - 分布式编程模型 http://web.cs.ucdavis.edu/~pandey/Research/Papers/icdcs01.pdf
  - PSync, http://www.di.ens.fr/~cezarad/popl16.pdf
  - Programming Models for Distributed Computing, http://heather.miller.am/teaching/cs7680/
  - Logic and Lattices for Distributed Programming, http://db.cs.berkeley.edu/papers/UCB-lattice-tr.pdf
  
### 验证分布式系统
  - Jepsen https://github.com/jepsen-io/jepsen
  - Verdi http://verdi.uwplse.org/
  
### 技术博客
  - How we implemented consistent hashing efficiently https://blog.ably.io/how-to-implement-consistent-hashing-efficiently-fe038d59fff2
  - Notes on Distributed Systems for Young Bloods http://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/
  - High Scalability http://highscalability.com/
  - All Things Distributed http://www.allthingsdistributed.com/
  - Distributed Systems: Take Responsibility for Failover http://ivolo.me/distributed-systems-take-responsibility-for-failover/
  - Files are hard http://danluu.com/file-consistency/
  - On Designing and Deploying Internet-Scale Services http://static.usenix.org/event/lisa07/tech/full_papers/hamilton/hamilton_html/

### 技术框架
  - 负载均衡
    - Nginx：高性能、高并发的web服务器；功能包括负载均衡、反向代理、静态内容缓存、访问控制；工作在应用层
    - LVS： Linux virtual server，基于集群技术和Linux操作系统实现一个高性能、高可用的服务器；工作在网络层
  - WebServer
    - Apache
    - Tomcat
  - Service
    - SOA
    - MicroServices
    - Spring Boot
  - 容器
    - Docker
    - Kubernetes (k8s)
  - 缓存
    - Redis
    - MemCache
  - 协调中心
    - ZooKeeper
  - RPC
    - gRPC
    - Dubbo
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
  - 数据库
    - MySQL
    - MongoDB
    - HBase
  - 搜索
    - ElasticSearch
    - solr
  - 日志
    - rsyslog
    - elk
- 参考
  - https://github.com/theanalyst/awesome-distributed-systems
  - https://github.com/aphyr/distsys-class
  - https://www.cnblogs.com/xybaby/p/7787034.html
