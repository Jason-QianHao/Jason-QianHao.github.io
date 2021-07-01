---
title: ZooKeeper
tags: ZooKeeper
---

*由简书搬迁而来[**原文链接**](https://www.jianshu.com/p/70ef30fb4b26)*

> 目录  
>   1 分布式架构  
>   2 一致性协议  
>   3 Zookeeper概述  
>   4 JavaAPI  
>   5 Zookeeper应用场景  
>   6 系统模型  
>   7 序列化与协议  
>   8 客户端  
>   9 会话  
>   10 服务器启动  
>   11 Leader选举  
>   12 数据与存储  
>   13 ZooKeeper代码  
>
> 参考目录  
>   · 《从Paxos到Zookeeper 分布式一致性原理与实践》   

# 分布式架构

## 对比两种架构

### 集中式

  由一台或多台主计算机组成中心节点，数据集中存储于这个中心节点中，并且整个系统的所有业务单元都集中部署在这个中心节点上，系统的所有功能均由其集中处理

### 分布式

（1）定义：是一个硬件或者软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统

（2）特点

  · 分布性：系统中的多台计算机都会在空间上随意分布，同时，机器的分布情况也会随时变动

  · 对等性：分布式系统中的计算机没有主/从之分

  · 并发性：同一个分布式系统中的多个节点，可能会并发地操作一些共享的资源。

  · 缺乏全局时钟：很难定义两个事件谁先谁后

  · 故障总是会发生的：所有计算机都有可能发生任何形式的故障。所以，除非需求指标允许，在系统设计时不能放过任何异常情况

（3）分布式环境的各种问题

  · 通信异常：在现代计算机体系结构中，单机内存访问的延时在纳秒数量级(10ns左右)，而正常的一次网络通信的延迟在0.1~1ms左右。

  · 网络分区（脑裂）：随着网络延时不断增大，导致系统中只有部分节点之间能够进行正常通信，而另一些节点则不能

  · 三态：每一次的请求与响应，存在着三种情况，成功、失败、超时。

  · 节点故障：服务器节点出现的宕机或“僵死”现象。

## 一致性

  分布式系统事务处理与数据一致性。

### ACID

  事务的四个特征，原子性、一致性、隔离性（隔离级别：未提交读、提交读、可重复读、串行化）、持久性。

### 分布式事务

#### 定义

  指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于分布式系统的不同节点上。

  分布式的操作序列称为子事务，也可以说是嵌套型的事务。

#### CAP理论

<img src="/../assets/Zookeeper/1240-20210701160045650.jpeg" alt="img" style="zoom:50%;" />

（1）定义：一个分布式系统不可能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三个基本需求，最多只能同时满足其中的两项

（2）一致性

  数据在多个副本之间是否能够保持一致的性质。当一个系统在数据一致的状态下执行更新操作后，应该保证系统的数据仍然处于一致的状态。

（3）可用性

  系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。

（4）分区容错性

  分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

#### BASE理论

  是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写。

（1）核心思想：即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。

（2）基本可用：允许损失部分可用性。如响应时间上的损失，从0.5秒变为1~2秒；功能上的损失，网站由于并发太高引导用户到一个降级页面。

（3）软状态：允许系统在不同节点的数据副本之间进行数据同步的过程存在延时。

（4）最终一致性：系统中的数据副本，在经过一段时间的同步后，最终能够到达一个一致的状态。

# 一致性协议 

## 概念

（1）协调者（Coordinator）：统一调度所有分布式节点的执行逻辑。协调者最终决定这些参与者是否要把事务真正的进行提交。

（2）参与者（Participant）：被调度的分布式节点

## 2PC

### 定义

  Two-Phase Commit的缩写，二段提交协议，是使分布式系统架构下的所有节点在进行事务处理过程中能够保持原子性和一致性而设计的一种算法。

### 一致性的方式

  通过统一决定事务的提交或回滚，从而保证分布式数据的一致性

### 流程

（1）阶段一：提交事务请求（“投票阶段”，是否继续执行事务提交操作）

  · 事务询问：协调者向所有的参与者发送事务内容，询问是否可以执行事务提交操作，并开始等待各参与者的响应。

  · 执行事务：各参与者节点执行事务操作，并将Undo和Redo信息记入事务日志中

  · 各参与者向协调者反馈事务询问的响应：如果参与者成功执行了事务操作，那么就反馈给协调者Yes响应，表示事务可以执行；如果参与者没有成功执行事务，那就反馈No响应，表示事务不可执行

（2）阶段二：执行事务提交

  **· 情况一：执行事务提交。如果所有的参与者获得的反馈都是Yes响应，则执行事务提交**

​    · 发送提交请求：协调者向所有参与者节点发出Commit请求

​    · 事务提交：参与者接收到Commit请求后，会正式执行事务提交操作，并在完成提交之后释放在整个事务执行期间占用的事务资源。

​    · 反馈事务提交结果：参与者在完成事务提交之后，向协调者发送Ack消息

​    · 完成事务：协调者收到所有参与者反馈的Ack消息后，完成事务

  · **情况二：中断事务。如果任何一个参与者反馈了No响应，或等待超时无法接受所有参与者的反馈，则中断事务**

​    · 发送回滚请求：协调者向所有参与者节点发出Rollback请求

​    · 事务回滚：参与者收到Rollback请求，利用Undo信息执行事务回滚操作，并完成后释放所占用的资源

​    · 反馈事务回滚结果：参与者在完成事务回滚后，向协调者发送Ack消息

​    · 中断事务：协调者接收到所有参与者反馈的Ack消息后，完成事务中断。

### 优缺点

（1）优点：原理简单，实现方便

（2）缺点：

​    · 同步阻塞：各个参与者在等待其他参与者的响应中，无法进行其他操作

​    · 单点问题：一旦协调者出现问题，那么整个二阶段提交流程将无法运转

​    · 数据不一致：阶段二若协调者发出Commit请求后，部分节点网络异常，导致只有部分参与者收到了Commit请求，会出现数据不一致情况。

​    · 太过保守：二阶段提交协议没有设计较为完善的容错机制，任意一个节点的失败都会导致整个事务的失败。

## 3PC

  将2PC的提交事务请求一分为二，形成由CanCommit、PreCommit和doCommit三个阶段组成的事务处理协议。

### 流程

（1）阶段一：CanCommit

​    · 事务询问：协调者向所有的参与者发送一个包含事务内的canCommit请求，询问是否可以执行事务提交操作，并开始等待各参与者的响应。

​    · 各参与者向协调者反馈事务询问的响应：若参与者认为可以顺利执行事务，反馈Yes响应，并进入预备状态，否则返回No响应

（2）阶段二：PreCommit

  **· 情况一：执行事务预提交：所有参与者的反馈都是Yes，执行预提交
**

​    · 发送预提交请求：协调者向所有参与者节点发出preCommit请求，进入Prepared阶段

​    · 事务预提交：参与者收到preCommit请求后，会执行事务操作，并将Undo和Redo信息记录到事务日志中

​    · 各参与者向协调者反馈事务执行的响应：如果参与者成功执行了事务操作，那么就会反馈给协调者Ack响应，同时等待最终的指令：提交（commit）或中止（absort）。

  **· 情况二：中断事务：任何一个参与者向协调者反馈了No响应，或者在等待超时之后无法接受到所有参与者反馈，就中断事务**

​    · 发送中断请求：协调者向所有参与者节点发出abort请求

​    · 中断事务：参与者中断事务

（3）阶段三：doCommit

  **· 情况一：执行提交：收到所有参与者的Ack响应，则从“预提交”进入“提交”状态
**

​    · 发送提交请求：发送doCommit请求

​    · 事务提交： 参与者收到doCommit请求后，执行事务提交操作，完成后释放所有占用资源

​    · 反馈事务提交结果：完成事务提交后，向协调者发送Ack消息

​    · 完成事务：协调者接收到所有参与者的Ack消息后，完成事务

  **· 情况二：中断事务：任何一个参与者向协调者反馈了No响应，或者在等待超时之后无法接受到所有参与者反馈，就中断事务**

​    · 发送中断请求：协调者向所有的参与者节点发送abort请求

​    · 事务回滚：参与者接收到abort请求后，利用Undo信息执行事务回滚操作，并完成后释放所有占用资源。

​    · 反馈事务回滚结果：完成回滚之后，发送Ack消息

​    · 中断事务：协调者接收到所有参与者反馈的Ack消息后，中断事务

### 优缺点

（1）优点：降低了参与者的阻塞范围，并且能够在出现单点故障后继续达成一致

（2）缺点：参与者收到preCommit消息后，出现网络分区，依然会提交事务，导致数据不一致。

## Paxos

参考：[分布式系列文章——Paxos算法原理与推导 - lzslbd - 博客园](https://www.cnblogs.com/linbingdong/p/6253479.html)

### 概念

（1）是一种基于消息传递且具有高度容错特性的一致性算法，是目前公认的解决分布式一致性问题最有效的算法之一。

（2）解决问题：如何在一个可能发生诸如机器宕机或网络异常的等异常的分布式系统中，快速且正确地在集群内部对某个数据的值（**提案proposal**）达成一致，并且保证不论发生以上任何异常，都不会破坏整个系统一致性。

### 角色

（1）Proposer 提出提案，如果某个提案被Accptor选定（chosen），那么该提案里的value就被选定了

（2）Acceptor 接受提案，只要Acceptor接受了某个提案，Acceptor就认为该提案里的value被选定了

（3）Learner Acceptor告诉Learner哪个value被选定，Learner就认为那个value被选定

（4）Proposal 提案 = 编号 + value（最终要达成一致的value）

### 约束和规定

（1）规定：一个提案被选定需要被半数以上的Acceptor接受

（2）Paxos的目标：保证最终有一个value会被选定，当value被选定后，进程最终也能获取到被选定的value。

（3）约束（推导过程）

  · P1：一个Acceptor必须接受它收到的第一个提案

  · P1a：一个Acceptor只要尚**未响应过**任何**编号大于N**的**Prepare请求**，那么他就可以**接受**这个**编号为N的提案**。

  · P2：如果某个value为v的提案被选定了，那么每个编号更高的被选定提案的value必须也是v

  · P2a：如果某个value为v的提案被选定了，那么每个编号更高的被Acceptor接受的提案的value必须也是v

  · P2b：如果某个value为v的提案被选定了，那么之后任何Proposer提出的编号更高的提案的value必须也是v

  · P2c：对于任意的N和V，如果提案[N, V]被提出，那么存在一个半数以上的Acceptor组成的集合S，满足以下两个条件中的任意一个：
    · S中每个Acceptor都没有接受过编号小于N的提案。
    · S中Acceptor接受过的最大编号的提案的value为V

### Proposer生成提案

（1）**Prepare请求**

  Proposer选择一个**新的提案编号N**，然后向**某个Acceptor集合**（半数以上）发送请求，要求该集合中的每个Acceptor做出如下响应（response）：

​    · 向Proposer承诺保证**不再接受**任何编号**小于N的提案**。

​    · 如果Acceptor已经接受过提案，那么就向Proposer响应**已经接受过**的编号小于N的**最大编号的提案**。

  我们将该请求称为**编号为N**的**Prepare请求**。

（2）**Accept请求**

  如果Proposer收到了**半数以上**的Acceptor的**响应**，那么它就可以生成编号为N，Value为V的**提案[N,V]**。这里的V是所有的响应中**编号最大的提案的Value**。如果所有的响应中**都没有提案**，那 么此时V就可以由Proposer**自己选择**。

  生成提案后，Proposer将该**提案**发送给**半数以上**的Acceptor集合，并期望这些Acceptor能接受该提案。我们称该请求为**Accept请求**。（注意：此时接受Accept请求的Acceptor集合**不一定**是之前响应Prepare请求的Acceptor集合）

### Acceptor接受提案

（1）响应：未响应过编号大于N的提案时，就接受提案

（2）拒绝：已经响应过N编号

### Paxos算法

**（1）阶段一：**

  · Proposer选择一个**提案编号N**，然后向**半数以上**的Acceptor发送编号为N的**Prepare请求**。

  · 如果一个Acceptor收到一个编号为N的Prepare请求，且N**大于**该Acceptor已经**响应过的**所有**Prepare请求**的编号，那么它就会将它已经**接受过的编号最大的提案（如果有的话）**作为响应反馈给Proposer，同时该Acceptor承诺**不再接受**任何**编号小于N的提案**。

**（2）阶段二：**

  · 如果Proposer收到**半数以上**Acceptor对其发出的编号为N的Prepare请求的**响应**，那么它就会发送一个针对**[N,V]提案**的**Accept请求**给**半数以上**的Acceptor。注意：V就是收到的**响应**中**编号最大的提案的value**，如果响应中**不包含任何提案**，那么V就由Proposer**自己决定**。

  · 如果Acceptor收到一个针对编号为N的提案的Accept请求，只要该Acceptor**没有**对编号**大于N**的**Prepare请求**做出过**响应**，它就**接受该提案**。

<img src="/../assets/Zookeeper/1240-20210701160045691.jpeg" alt="img" style="zoom:50%;" />

### 其他问题

（1）Learner学习被选定的value

<img src="/../assets/Zookeeper/1240-20210701160045800.png" alt="img" style="zoom:50%;" />

（2）如何保证Paxos算法的活性

<img src="/../assets/Zookeeper/1240-20210701160045906.png" alt="img" style="zoom:50%;" />



# Zookeeper概述

## 概念

### 定义

  是一个开放源代码的分布式协调服务，是Google Chubby的开源实现，也是一个典型的分布式数据一致性的解决方案，可以基于它实现数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、master选举、分布式锁和分布式队列等功能

### 特性

（1）顺序一致性：从同一个客户端发起的事务请求，最终功能将会严格按照其发起顺序被应用到Zookeeper中去

（2）原子性：所有事务请求的处理结果在整个集群中所有机器上的应用情况时一致的

（3）单一视图：物理客户端连接的是哪个Zookeeper服务器，其看到的服务端数据模型都是一致的

（4）可靠性：保留客户端的事务响应

（5）实时性：Zookeeper保证事务被应用成功，则客户端可以读取到事务变更后的最新数据状态（仅仅保证在一定时间内，客户端最终一定会读取到最新状态）。

### 设计目标

（1）简单的数据模型

（2）可以构建集群：只要集群中存在超过一半的机器能够正常工作，那么整个集群就能正常对外服务。

（3）顺序访问：每个事务操作都会被分配一个全局唯一的递增编号，反应先后顺序。

（4）高性能，数据存储在内存中

### 集群角色

  没有沿用传统的Master/Slave概念，引入了下列三个角色。

（1）Leader：选举产生一台机器，为客户端提供读和写服务

（2）Follower：只能提供读服务，可以参与Leader选举

（3）Observer：只能提供读服务，但不参与选举 [Apache ZooKeeper - 集群中 Observer 的作用以及 与 Follow 的区别_小工匠-CSDN博客](https://blog.csdn.net/yangshangwei/article/details/111658771)

### 会话（Session）  

  以客户端和服务器之间的一个TCP长连接为生命周期的开始。sessionTimeout为客户端最大断开连接时间

### 数据节点（ZNode）

（1）持久节点

（2）持久顺序节点

（3）临时节点（随着会话结束而删除）

（4）临时顺序节点

  顺序节点有由父节点维护的自增整型数字，追加在节点后面。

### 版本

  每一个Znode都含有一个Stat的数据结构，记录了三个数据版本：

​    · version 当前ZNode版本

​    · cversion 当前ZNode子节点的版本

​    · aversion 当前ZNode的ACL版本

### Watcher

  事件监听器，对客户端的触发事件进行回调。

  该机制是Zookeeper实现分布式协调服务的重要特性

### ACL

  Access Control Lists 策略，用来进行权限控制。共有五种权限：

​    · CREATE 创建子节点的权限

​    · READ 获取节点数据和子节点列表的权限

​    · WRITE 更新节点数据的权限

​    · DELETE 删除子节点的权限

​    · ADMIN 设置节点ACL的权限

## ZAB协议

### 概述

（1）Zookeeper Aotomic Broadcast Zookeeper原子消息广播协议，是一种特别为Zookeeper设计的奔溃可恢复的原子消息广播算法

（2）核心：定义了对于那些会改变Zookeeper服务器数据状态的事务请求的处理方式

​    · 所有事务请求必须由一个唯一的服务器来协调处理，即Leader服务器

​    · 生成一个事务Proposal广播给所有的Follower服务器

​    · 等待半数以上的Follower服务器正确反馈后，Leader发送commit命令，所有Follower提交Porposal

（3）ZAB协议具有两种基本的模式：崩溃恢复和消息广播

### 崩溃恢复

#### 定义

  当整个服务器框架在启动过程中，或是当Leader服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB协议进入恢复模式并**选举产生新的Leader服务器，**并进行数据同步。

#### 基本特性

  针对可能会出现的两个数据不一致性的隐患，需要保证以下特性：

​    · ZAB协议需要确保那些已经在Leader服务器上提交的事务最终被所有服务器都提交

​    · ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务

#### Leader选举算法

​    针对需要确保的基本特性，设计的选举算法可以：然Leader选举算法能够保证新选举出来的Leader服务器拥有集群中所有机器最高编号（即ZXID最大）的事务Proposal。那么就可以保证这个新选举出来的Leader一定具有所有已经提交的提案。

  选举中的选票就是利用Proposal中的epoch值进行比较。

#### 数据同步

（1）定义

  完成选举后，Leader服务器首先需要确认事务日志中的所有Proposal是否都已经被集群中过半的机器提交了，即是否完成数据同步。

（2）过程

  · Leader服务器会为每个Follower服务器都准备一个队列。

  · 将没有被各Follower同步的事务放在队列中逐个发送，每个事务后紧跟一个commit消息

  · 将同步成功的Follower加入真正可用的Follower列表

### 消息广播

（1）定义

  当集群中已经有过半的Follower服务器完成了和Leader同步后，则整个服务器框架进入消息广播模式。

  Leader服务器在接收到客户端的事务请求后，会生成对应的事务提案并发起一轮广播协议；而如果集群中的其他机器收到客户端的事务请求，那么这些非Leader服务器会首先将这个事务请求转发给Leader服务器。

（2）使用原子广播协议

  · 类似于二段提交协议，但移除了中断逻辑。所有的Follower服务器要么正常反馈要么直接抛弃Leader服务器。

  · 在收到过半的Follower服务器的反馈后直接就可以开始提交事务，不需要等待集群中所有的Follower服务器响应

  · 整个消息广播协议是基于具有FIFO特性的TCP协议来进行网络通信的，因此能够很容易地保证消息广播过程中消息接收与发送的顺序性。

（3）过程

  · Leader服务器为每一个Follower服务器准备一个单独的队列，将需要广播的事务依次放入队列

  · Follower接收到Proposal后，会以事务日志的形式写入到本地磁盘中去，并且在成功写入后反馈给Leader服务器一个ACk响应

  · Leader收到半数以上的响应，发送Commit消息，各自提交事务

### 事务编号ZXID

（1）是一个64位的数字

（2）低32位可用看作是一个简单的单调递增的计数器，对客户端每一个事务请求加1

（3）高32位代表了Leader周期epoch编号

（4）每当选举产生一个新的Leader时，从该Leader的本地日志中解析出最大编号ZXID，对其epoch值加1作为新的epoch，低32位置0，生成新的ZXID。

### 系统模型

#### 进程分类

（1）进程正常工作，则成为进程处于UP状态

（2）进程处于崩溃状态，称为DOWN状态

（3）一个集群中过半UP状态的进程组成的一个子集称为Quorum

#### 进程运行状态

（1）LOOKING：Leader选举阶段

（2）FOLLOWING：Follower服务器和Leader保持同步状态

（3）LEADING：Leader服务器作为主进程领导状态

### 与Paxos的区别

（1）ZAB协议主要用于构建一个高可用的分布式数据主备系统

（2）Paxos是用于构建一个分布式的一致性状态机系统

# JavaAPI

（1）增删查改操作中，更新操作时基于CAS乐观锁 + version版本号实现的

（2）两个开源客户端ZkClient和Curator，对Zookeeper原生API接口进一步包装

# Zookeeper应用场景

## 发布/订阅

（1）推拉相结合方式

​    客户端向服务器注册关注节点，数据发送变更后，服务器发送Watcher通知，客户端主动获取最新配置信息

（2）可以将配置信息放在Zookeeper上，配置发生变更，利用Watcher通知，客户端通过订阅来获取最新配置信息。

## 负载均衡

（1）定义

​    Load Balance是一种相当常见的计算机网络技术，用来对多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源进行分配负载，以达到优化资源使用、最大化吞吐率、最小化响应时间和避免过载的目的。

（2）分类

  · 硬件负载均衡：F5设备

  · 软件负载均衡：Zookeeper利用节点存储IP等方法可实现负载均衡，还有Nginx反向代理

## 命名服务

（1）Java语言中的JNDI（Java Naming and Directory Interface）Java命名与目录接口，是一种典型的命名服务

（2）分布式全局唯一ID

  · UUID：32位字符和4个短线的字符串，缺点是字符串过长，占空间

  · Zookeeper顺序节点：在API返回值中返回节点的完整名字，后面的数字为自动递增的

<img src="/../assets/Zookeeper/1240-20210701160045719-5126445.jpeg" alt="img" style="zoom:50%;" />

## 分布式协调/通知

### 通常做法

​    不同的客户端都对Zookeeper上同一个节点进行Watcher注册，监听数据节点变化，所有订阅的客户端都会收到相应的通知并作出相应处理

### 通用的分布式系统机器间通信

（1）心跳检测

  · 通过主机之间是否可以相互PING通来判断，或是通过TCP连接固有的心跳检测来实现

  · 基于Zookeeper：不同节点在一个节点下创建子节点进行判断

（2）工作进度汇报

  每个任务客户端在指定节点下面创建临时子节点，可以进行：

​    · 通过判断临时节点是否存在来确定任务机器是否存活

​    · 各个任务机器会实时地将自己的任务执行进度写到这个临时节点，以便中心系统能够实时地获取到任务的执行进度。

（3）系统调度

  管理人员在控制台上进行操作，实际是修改节点数据，而Zookeeper进一步把这些数据变更以事件通知的形式发送给了对应的订阅客户端。

## 集群管理

（1）包括集群监控和集群控制两大块。

​    集群监控：侧重对集群运行时状态的收集

​    集群控制：侧重对集群进行操作与控制

（2）基于Zookeeper下列两大特性实现管理：

  · 对数据节点监听，并结合Watcher事件通知

  · 临时节点，在客户端与服务器之间会话失效后被自动清除

## Master选举

  这里指的是Client集群中的选举，不是Zookeeper集群

（1）背景

​    在海量数据处理中，非常耗费I/O和CPU，只让client集群中的一台去处理数据，结果共享。选举的Master就会负责进行一系列的海量数据处理。

（2）方式

​    Client集群每天定时在Zookeeper上创建一个临时节点，只有一个客户端能够成功创建它就成为Master，如果挂了，就重新选举。

<img src="/../assets/Zookeeper/1240-20210701160045977.jpeg" alt="img" style="zoom:50%;" />

## 分布式锁

### 排他锁

（1）获取锁

​    创建**临时子节点**成功，即获得锁

（2）释放锁

​    · Client宕机，Zookeeper的临时节点被删除

​    · 正常执行完事务，Client主动删除

​    锁被释放后，所有正在等待获取锁的事务都能够被通知到。

### 共享锁

（1）获取锁

​    根据读/写请求，创建不同的**临时顺序节点**。

（2）读写顺序

​    · 对于读：若没有比自己序号小的子节点，或是所有比自己序号小的子节点都是读请求，那么表明自己已经成功获取到了共享锁，否则等待。

​    · 对于写：若自己不是序号最小的子节点，那么就需要等待

（3）羊群效应

​    · Client收到大量和自己并不相关的事件通知

​    · 改进锁：每个锁只关注序号比自己小的那个节点（缩小锁的范围）

## 分布式队列

  大致分为两大类，一种是常规的先入先出队列，另一种则是要等到队列元素集聚之后才统一安排执行的Barrier模型

（1）FIFO

​    类似全写的共享锁，创建临时顺序节点，监听小于自己的节点。

（2）Barrier

​    · 创建临时子节点

​    · 调用getChildren()接口获取节点下的所有子节点，知道子节点个数满足才执行

# 系统模型

## 数据模型

（1）树状结构

<img src="/../assets/Zookeeper/1240-20210701160045662.jpeg" alt="img" style="zoom:50%;" />

（2）ZNode

​    除了分为四类节点外，还有一个Stat对象保存节点信息，Stat对象属性如下：

<img src="/../assets/Zookeeper/1240-20210701160045784.jpeg" alt="img" style="zoom:50%;" />

## 版本

  在数据节点中引入三个版本号。

（1）作用：保证分布式数据原子性操作

（2）version属性正是用来实现乐观锁机制中的“写入校验”

（3）具体实现：在Zookeeper服务器中的PrepRequestProcessor处理器中，处理每一个数据更新请求（setDataRequest）时，比较当前请求的版本version和当前服务器上该数据的最新版本currentVersion，不匹配抛异常。

## Watcher

### 概述

<img src="/../assets/Zookeeper/1240-20210701160045719.jpeg" alt="img" style="zoom:50%;" />

（1）Watcher机制主要包括**客户端线程**、**客户端WatchManager**和**Zookeeper服务器**。

（2）客户端在向Zookeeper服务器注册Watcher的同时，会将Watcher对象存储在客户端的WatcherManager中。当Zookeeper服务器触发Watcher事件后，会向客户端发送通知，客户端线程从WatcherManager中取出对应的Watcher对象来执行回调逻辑。

### Watcher接口

  接口主要包含以下属性

（1）KeeperState枚举类

（2）EventType枚举类

（3）process（WatchedEvent event）回调方法

<img src="/../assets/Zookeeper/1240-20210701160045778.jpeg" alt="img" style="zoom:50%;" />

### WatchedEvent

  在回调方法process（WatchedEvent event）中，传入参数WatchedEvent

（1）包含通知状态（keeperState）、事件类型（eventType）和节点路径（path）三个基本属性。用来封装事件传递给Watcher。

（2）服务器生成WathedEvent后，调用**getWrapper**方法将自己包装成一个可序列化的、可以用于网络传输的**WatcherEvent**事件

（3）客户端接收到WatcherEvent事件后，先还原成一个WatchedEvent事件，并传递给process方法处理

### 工作机制

  总体分为三个过程：客户端注册Watcher、服务端处理Watcher和客户端回调Watcher。

<img src="/../assets/Zookeeper/1240-20210701160045769.jpeg" alt="img" style="zoom:50%;" />

#### 客户端注册Watcher

（1）注册方式

​    · new Zookeeper（.....Watcher watcher）；

​    · getData（...）;

​    · getChildren（...）;

​    · exist（...）；

（2）流程

<img src="/../assets/Zookeeper/1240-20210701160045790.jpeg" alt="img" style="zoom:50%;" />

- **为什么只传部分参数**

  这里网上没有查到相关资料，我觉得是为了减轻网络传输。packet里面有整个wacher的信息，包括不同事件会有什么响应，这些不需要发送到服务端，这个喝后面把watcher注册到本地的ZKWatcherManager应该也有关系。

​    在运行过程中，始终保持着数据节点路径和Wather对象的一一映射。

<img src="/../assets/Zookeeper/1240-20210701160045803.jpeg" alt="img" style="zoom:50%;" />

#### 服务端处理Watcher

注意服务端WatcherManager和ClientWatcherManager的区别，参考：[Watcher机制](https://xn--zookeeper-8r4ol888a/)

（1）分为Servercnxn存储和Watcher触发两部分

（2）流程

<img src="/../assets/Zookeeper/1240-20210701160045839.jpeg" alt="img" style="zoom:50%;" />

<img src="/../assets/Zookeeper/1240-20210701160045813.jpeg" alt="img" style="zoom:50%;" />

- **为什么Watcher一次性有效**

  参考：[漫谈zookeeper watcher单次触发特性](https://nightfield.com.cn/index.php/archives/194/)

  <img src="/../assets/Zookeeper/image-20210701163144724.png" alt="image-20210701163144724" style="zoom:50%;" />

  所以，在`getData()`、`exists()` 和 `getChildren()` 的操作中，添加了一个 `Watcher` 参数，用于重新注册watcher。

#### 客户端回调Watcher

（1）分为**SendThread接收事件通知**和**EventThread处理事件通知**两部分。

（2）流程

<img src="/../assets/Zookeeper/1240-20210701160045853.jpeg" alt="img" style="zoom:50%;" />

# 序列化与协议

## 概述

  对于一个网络通信，首先需要解决的就是对数据的序列化和反序列化处理。ZooKeeper使用Jute组件进行序列化。

## 使用Jute的步骤

（1）实体类需要实现Record接口的serialize和deserialize方法

（2）构建一个序列化器BinaryOutputArchive

（3）序列化，调用实体类的serialize方法

（4）反序列化。调用实体类的deserialize方法

## 序列化器

（1）Record接口

![img](/../assets/Zookeeper/1240-20210701160045832.jpeg)

（2）底层真正的序列化器

  · OutputArchive接口 Jute底层序列化器，基于OutputStream实现，有3种常见实现

  · InputArchive接口 Jute底层反序列化器，基于InputStream实现

## 通信协议

  基于TCP/IP协议，Zookeeper实现了自己的通信协议来完成客户端与服务器、服务器与客户端之间的网络通信。

  除非是“会话创建”请求，其他所有client请求都会带上请求头。

# 客户端

## ZooKeeper客户端的核心组件

（1）ZooKeeper实例：客户端入口

（2）ClientWatchManager：客户端Watcher管理器

（3）HostProvider：客户端地址列表管理器

（4）ClientCnxn：客户端核心线程，分为两个：SendThread是I/O线程，主要负责ZooKeeper客户端和服务器之间的网络I/O通信；EventThread是事件线程，主要负责对服务端事件进行处理。

<img src="/../assets/Zookeeper/1240-20210701160045858.jpeg" alt="img" style="zoom:50%;" />

## 会话创建

  分为三个阶段：初始化阶段、会话阶段、响应处理阶段。

<img src="/../assets/Zookeeper/1240-20210701160045880.jpeg" alt="img" style="zoom:50%;" />

### 初始化阶段

（1）初始化Zookeeper对象

​    实例化一个ZooKeeper对象，并创建一个Watcher管理器：ClientWatcherManager

（2）设置会话默认Watcher

​    构造方法中传入的Watcher对象，被认作默认Watcher保存在ClientWatcherManager中

（3）构造ZooKeeper服务器地址列表管理器：HostProvider

​    构造方法中的服务器地址，会放在服务器地址列表管理器HostProvider中

（4）创建并初始化客户端网络连接器：ClientCnxn

​    创建一个网络连接器ClientCnxn，同时初始化客户端两个核心队列**outgoingQueue和pendingQueue分别作为客户端的请求发送队列和服务端响应的等待队列**

​    底层是ClientCnxnSocket处理器。

（5）初始化SendThread和EventThread

​    将ClientCnxnSocket分配给SendThread作为底层网络I/O处理器；初始化EventThread的**待处理事件队列waitingEvents**。

### 会话创建阶段

（6）启动SendThread和EventThread

（7）获取一个服务器地址

​    SendThread从HostProvider中随机获取出一个地址，委托给ClientCnxnSocket去创建TCP连接

（8）创建TCP连接

（9）构造ConnectRquset请求

​    ZooKeeper客户端进一步将请求包装成网络I/O层的Packer对象，放入请求发送队列outgoingQueue中去

（10）发送请求

​    ClientCnxnSocket从outgoingQueue中取出一个Packet对象，将其序列化为ByteBuffer后向服务器发送

### 响应处理阶段

（11）接收服务端响应

​    ClientCnxnSocket接收到服务端的响应后，判断当前的客户端状态是否是“已初始化”，若没有初始化则交由readConnectResult方法处理

（12）处理Response

​    反序列化得到ConnectResponse对象从，从中获取ZooKeeper服务器分配的sessionId

（13）连接成功

​    通知SendThread线程，设置客户端会话参数包括readTimeout和connectTimeout等，并更新客户端状态

​    通知地址管理器HostProvider当期成功连接的服务器地址

（14）生成事件：SyncConnected-None

​    SendThread生成这个事件，并传递给EventThread

（15）查询Watcher

​    EventThread从ClientWatcherManager管理器中查询出对应的Watcher，放入waitingEvents队列中

（16）处理事件

​    EventThread不断从watingEvents中取出Watcher对象调用对象的process接口方法

## 服务器地址列表

（1）用户传入的服务器地址列表，通常是一个使用英文状态逗号分隔的多个IP地址和端口的字符串

（2）地址使用ConnectStringParser解析器对象封装起来。

（3）解析器解析ChrootPath：命名空间，实则的相对路径

（4）HostProvider地址列表管理器

​      解析器将IP和端口封装成InetSocketAddress对象，以ArrayList形式保存到ConnectStringParser的相应属性中，进一步封装到HostProvider的默认实现类StaticHostProvider中。

<img src="/../assets/Zookeeper/1240-20210701160045864.jpeg" alt="img" style="zoom:50%;" />

​    StaticHostProvdier类中解析InetSocketAddress后，将地址打散重排形成一个环形循环队列。next()方法每次按照顺序返回一个已经解析过的InetSocketAddress。

## ClientCnxn

### Packet

<img src="/../assets/Zookeeper/1240-20210701160045873.jpeg" alt="img" style="zoom:50%;" />

（1）Packet是ClientCnxn内部定义的一个对协议层的封装

（2）Packet中包含了最基本的请求头、响应头、请求体、响应体、节点路径和注册的Watcher等信息

（3）Packet的createBB()方法负责序列化，最终生成可用于底层网络传输的ByteBuffer对象。

​    只会将requestHeader、request和readOnly三个属性进行序列化。

### 底层Socket通信

  · 默认实现是ClientCnxnSocketNIO。

  · 从outgoingQueue中取出的Packet对象，在请求发送完毕后，会立即将该Packet保存到pendingQueue队列中。

  · SendThread维护了会话生命周期，在一定周期频率内向服务端发送一个PING包来实现心跳检测。

# 会话

## 会话状态

（1）CONNECTING：开始创建ZooKeeper客户端

（2）CONNECTED：成功连接上服务器

（3）RECONNECTING：

（4）RECONNECTED：

（5）CLOSE：出现会话超时、权限检测失败或是客户端主动退出程序等情况

## 服务端会话工作原理

### Session

（1）4个基本属性

​    · sessionID：唯一标识一个会话

​    · Timeout：会话超时时间

​    · TickTime：下次会话超时时间点。13位的Long整型

​    · isClosing：标记一个会话是否关闭

（2）sessionID生成步骤

​    · 获取当前时间的毫秒表示。调用System.currentTimeMillis（）

​    · 左移24位

​    · 无符号右移8位

​    · 添加机器标识SID，左移56位。SID就是在myid文件中的值

​    · 将运算后的时间按位或SID

  sessionID的高8位确定了所在机器，后56位使用当前时间的毫秒表示进行随机

### SessionTracker

  是ZooKeeper服务端的会话管理器，辅助会话的创建、管理和清理等工作。

#### 分桶策略

  将类似的会话放在同一区块中进行管理，以便于ZooKeeper对会话进行不同区块的隔离处理以及同一区块的统一处理。

  分配区块的原则是“下次超时时间点”。

<img src="/../assets/Zookeeper/1240-20210701160045876.jpeg" alt="img" style="zoom:50%;" />

#### 会话激活

  服务端不断受到来自客户端的这个心跳检测，并需要重新激活对应的客户端会话。

（1）步骤

​    · 检验该会话是否已经被关闭

​    · 计算会话新的超时时间

​    · 定位该会话当前的区块

​    · 迁移会话

<img src="/../assets/Zookeeper/1240-20210701160045881.jpeg" alt="img" style="zoom:50%;" />

<img src="/../assets/Zookeeper/1240-20210701160045920.jpeg" alt="img" style="zoom:50%;" />

#### 会话超时检查

  创建了一个单独的线程专门进行会话超时检查。

  逐个对会话桶中剩下的会话进行清理。

# 服务器启动

<img src="/../assets/Zookeeper/1240-20210701160045923.jpeg" alt="img" style="zoom:50%;" />

## 单机版

  主要分为五大步骤：配置文件解析、初始化数据管理器、初始化网络I/O管理器、数据恢复和对外服务。可以概括为预启动和初始化两大类。

<img src="/../assets/Zookeeper/1240-20210701160047286.jpeg" alt="img" style="zoom:50%;" />

### 预启动

（1）统一由QuorumPeerMain作为启动类

​    是由zkServer.cmd和zkServer.sh两个脚本中配置的启动类

（2）解析配置文件zoo.cfg

（3）创建并启动历史文件清理器DatadirCleanupManager

​    包括对事务日志和快照数据文件进行定时清理

（4）判断当前是集群模式还是单机模式的启动

​    根据zoo.cfg解析的集群服务器地址列表来判断，如果是单机模式交给ZooKeeperServerMain进行启动处理

（5）再次进行配置文件zoo.cfg的解析

（6）创建服务器实例**ZooKeeperServer**

### 初始化

（1）创建服务器统计器ServerStats

​    包含了最基本的运行时信息。

（2）创建ZooKeeper数据管理器FIleTxnSnapLog

​    是上层服务器和底层数据存储之间的对接层，包括事务日志文件和快照数据文件的接口。

（3）设置服务器tickTime和会话超时时间限制

（4）创建ServerCnxnFactory

​    可以通过配置属性来决定使用自己实现的NIO还是Netty框架作为网络连接工厂。

（5）初始化ServerCnxnFactory

​    初始化一个线程作为ServerCnxnFactory的主线程，再初始化NIO服务器

（6）启动ServerCnxnFactory主线程

​    服务器端口号2181，虽然此时已经开放连接端口，但ZooKeeper此时还无法正常处理客户端请求的。

（7）恢复本地数据

​    从本地快照数据文件和事务日志文件中进行数据恢复

（8）创建并启动会话管理器

​    创建SessionTracker。初始化SessionTracker后，立即开始会话管理器的会话超时检查。

（9）初始化ZooKeeper的请求处理链

​    责任链模式，会有多个请求处理器一次处理一个客户端请求，串联处理。

​    单机版主要包括：PrepRequestProcessor-->SyncRequestProcessor-->FinalRequestProcessor

（10）注册JMX服务

​    将服务器运行时的一些信息以JMX（Java 管理扩展）的方式暴露给外部。

（11）注册ZooKeeper服务器实例

​    将已经完成初始化的ZookeeperServer实例注册到ServerCnxnFactory即可。

## 集群版

<img src="/../assets/Zookeeper/1240-20210701160045898.jpeg" alt="img" style="zoom:50%;" />

### 预启动

（1）统一由QuroumPeerMain作为启动类

（2）解析配置文件zoo.cfg

（3）创建并启动历史文件清理器DatadirCleanupManager

（4）判断当前是集群模式还是单机模式

​      在zoo.cfg中配置了多个服务器地址，因此此处选择集群模式启动ZooKeeper

### 初始化

（1）创建ServerCnxnFactory

（2）初始化ServerCnxnFactory

（3）创建ZooKeeper数据管理器FileTxnSnapLog

（4）创建QuorumPeer实例

​    Quorum是Zookeeper服务器实例（ZooKeeperServer）的托管者。QuorumPeer代表了ZooKeeper集群中的一台机器，在运行期间QuorumPeer会不断检测当前服务器实例的运行状态，同时根据情况发起Leader选举。

（5）创建内存数据库ZKDatabase

​    负责管理ZooKeeper的所有会哈记录以及DataTree和事务日志的存储。

（6）初始化QuorumPeer

​    将FileTxnSnapLog、ServerCnxnFactory和ZkDatabase等核心组件注册到QuorumPeer中去，同时配置服务器地址列表、leader选举算法和会话超时时间限制等参数

（7）恢复本地数据

（8）启动ServerCnxnFactory主线程

### Leader选举

（1）初始化Leader选举

​    根据SID、lastLoggedZxid（最新ZXID）和当前的服务器epoch（currentEpoch）来生成一个初始化投票。

​    ZooKeeper中默认有三种算法：LeaderElection、AuthFastLeaderElection和FastLeaderElection，可以在zoo.cfg总配置。**从3.4.0版本开始，ZooKeeper只支持FastLeadElection算法了**。

​    创建Leader选举锁需要的网络I/O层QuorumCnxManager，同时启动端口监听，等待其他服务器创建连接。

（2）注册JMX服务

（3）检测当前服务器状态

​    服务状态在LOOKING、LEADING和FOLLOWING/OBSERVING之间切换。

​    启动阶段，QuorumPeer是LOOKING，开始进行Leader选举

（4）leader选举

### Leader和Follower的启动

（1）交互期

<img src="/../assets/Zookeeper/1240-20210701160045933.jpeg" alt="img" style="zoom:50%;" />

（2）启动

  · 创建并启动会话管理器

  · 初始化ZooKeeper的请求处理链

  · 注册JMX服务

# Leader选举

## 投票格式

  投票包括所推举的服务器的myid和ZXID，以（myid，ZXID）表示。

## 服务器启动时期的Leader选举（无Leader）

（1）初始化投票，第一轮投票投给自己

（2）接收来自各个服务器的投票

​      会判断投票的有效性，检测是否是本轮投票，是否来自LOOKING状态的服务器

（3）处理投票

​      · 优先比较ZXID，较大的作为Leader

​      · ZXID相同则比较myid，较大的作为Leader

​    判断完后，**变更投票**，再一次向集群发送投票。

（4）统计投票

​    判断是否已经有过半的机器接收到相同的投票信息。达到时，Leader服务器就被选出了。

（5）改变服务器状态

​    Leader确定后，服务器更新自己状态，Follower变为FOLLOWING，Leader变为LEADING。

## 服务器运行期间的Leader选举（Leader下线）

（1）**变更服务器状态**

​    所有非observer服务器变为LOOKING，进入Leader选举流程。

（2）每个Server会发出一个投票

​    第一次投票，投给自己

（3）接收来自各个服务器的投票

（4）处理投票

（5）统计投票

（6）改变服务器状态

# 数据与存储

.  ZooKeeper中数据存储分为内存数据存储和磁盘数据存储两部分。

## 内存数据

  ZooKeeper会定时将所有的节点路径、节点数据及其ACL信息等存储到磁盘上。

###  DataTree

<img src="/../assets/Zookeeper/1240-20210701160045950.jpeg" alt="img" style="zoom:50%;" />

（1）DataTree是内存属性核心，是一个“树”结构，代表了内存中的一份完整的数据

（2）DataNode是数据存储的最小单元

（3）nodes是一个ConcurrentHashMap结构，节点路径为key，DataNode为值

（4）对于所有的临时节点，为了方便计时清理，有一个单独的ConcurrentHashMap<Long，Hashset<String>>

### ZKDatabase

（1）是ZooKeeper的内存数据库

（2）负责管理所有会话、DataTree存储和事务日志

（3）定时向磁盘dump快照数据

（4）服务器启动时，通过磁盘的事务日志和快照数据文件恢复内存数据库。

## 事务日志

### 文件存储

（1）ZooKeeper的zoo.cfg文件中配置事务日志存储目录dataDir

（2）运行过程中在dataDir中生成包含版本号的子目录

（3）日志文件的后缀名，是写入该日志文件的第一条事务记录的ZXID

### 日志写入

  FileTxnLog负责维护事务日志对外的接口，包括事务日志的写入和读取等。写入日志大致过程：

（1）确定是否有事务日志可写

​    ZooKeeper首先判断FileTxnLog组件是否已经关联上一个可写的事务日志文件

​    如果没有关联，则用第一个事务操作的ZXID作为后缀创建一个事务日志文件，同时构建事务日志头信息。同时该文件的文件流放入streamsToFlush集合（将数据强制刷入磁盘上）中。

（2）确定事务日志文件是否需要扩容（预分配）

​    当前日志文件大小不足4KB，在现有文件大小的基础上增加64MB，使用‘\0’填充。这样可以避免频繁的磁盘Seek

（3）事务序列化

（4）生成Checksum

（5）写入事务日志文件流

​    写入到BufferedOutputStream中。

（6）事务日志刷入磁盘

​    从streamsToFlush中提取文件流

### 日志截断

  非Leader的ZXID比Leader大，Leader会发送TRUNC命令给该机器，机器删除包含或大于此ZXID的事务日志文件

## Snapshot数据快照

### 作用

  记录ZooKeeper服务器上某一个时刻的全量内存数据内容，并将其写入到指定的磁盘文件中。

### 文件存储

  同事务日志原理

### 数据快照

  FileSnap负责维护快照数据对外的接口，包括快照数据的写入和读取等。

  ZooKeeper在进行若干次事务日志记录之后，将内存数据库的全量数据Dump到本地文件中，这个过程就是数据快照。可以配置snapCount参数控制每次数据快照之间的事务操作数。

  数据快照的过程如下：

（1）确定是否需要进行数据快照

​    理论上进行snapCount次事务后开始快照，但实际上采取“过半随机”策略：logCount > ( snapCount / 2 + randRoll )

（2）切换事务日志文件

​    当前事务日志已经“写满”（已经写了snapCount个事务日志），需要重新创建一个新的事务日志。

（3）创建数据快照异步线程

​    需要单独的异步线程来进行数据快照

（4）获取全量数据和会话信息

​    从ZKDatabase中获取到DataTree和会话信息。

（5）生成快照数据文件名

​    ZooKeeper根据当前已提交的最大ZXID来生成数据快照文件名

（6）数据序列化

## 初始化

（1）作用

​    将存储在磁盘上的数据文件加载到ZooKeeper服务器内存中，主要包括了从快照文件中加载快照数据和根据事务日志进行数据订正两个过程

（2）FileTxnSnapLog是ZooKeeper事务日志和快照数据访问层，分为FileTxnLog和FileSnap初始化

## 数据同步

### 初始化

  Leader服务器从ZooKeeper的内存数据库中提取出事务请求对应的提议缓存队列proposal，同时完成对以下三个ZXID值的初始化。

（1）peerLastZxid：该**Learner**服务器最后处理的ZXID

（2）minCommittedLog：Leader服务器提议缓存队列committedLog中的最小ZXID

（3）maxCommittedLog：Leader服务器提议缓存队列committedLog中的最大ZXID

### 集群数据同步方式

（1）直接差异化同步 DIFF

​    peerLastZxid介于minCommittedLog和maxCommittedLog之间。

（2）先回滚再差异化同步 TRUNC + DIFF

​    Leader服务器在已经将事务记录到了本地事务日志中，但是没有成功发起Proposal流程的时候就挂了。

（3）仅回滚同步 TRUNC

​    peerLastZxid大于maxCommittedLog.

（4）全量同步（SNAP同步）

​    · peerLastZxid小于maxCommittedLog

​    · Leader服务器上没有提议缓存队列，peerLastZxid不等于Leader服务器数据恢复后得到的最大ZXID。

# ZooKeeper代码

## 依赖
```xml
 <dependency>
   <groupId>org.apache.zookeeper</groupId>
   <artifactId>zookeeper</artifactId>
   <version>3.4.6</version>
 </dependency>
```
## 客户端代码
```java
 public class ZookeeperDemo {
   /**
   \* 集群连接地址
   */
   private static final String CONNECT_ADDR =   "192.168.110.138:2181,192.168.110.147:2181,192.168.110.148:2181";
   /**
   \* session超时时间
   */
   private static final int SESSION_OUTTIME = 2000;
   /**
   \* 信号量,阻塞程序执行,用户等待zookeeper连接成功,发送成功信号，
   */
   private static final CountDownLatch countDownLatch = new CountDownLatch(1);
   public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
    ZooKeeper zk = new ZooKeeper(CONNECT_ADDR, SESSION_OUTTIME, new   Watcher() {
    public void process(WatchedEvent event) {
    // 获取时间的状态
    KeeperState keeperState = event.getState();
    EventType tventType = event.getType();
    // 如果是建立连接
    if (KeeperState.SyncConnected == keeperState) {
      if (EventType.None == tventType) {
        // 如果建立连接成功,则发送信号量,让后阻塞程序向下执行
        countDownLatch.countDown();
        System.out.println("zk 建立连接");
      }
    }
    }
   });
   // 进行阻塞
   countDownLatch.await();
   //创建父节点
   // String result = zk.create("/testRott", "12245465".getBytes(),   Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
   // System.out.println("result:" + result);
   //创建子节点
   String result = zk.create("/testRott/children", "children 12245465".getBytes(),   Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
   System.out.println("result:"+result);
   zk.close();
  }
 }
```
## Watcher的使用

```java
 public class ZkClientWatcher implements Watcher {
   // 集群连接地址
   private static final String CONNECT_ADDRES = "192.168.110.159:2181,192.168.110.160:2181,192.168.110.162:2181";
   // 会话超时时间
   private static final int SESSIONTIME = 2000;
   // 信号量,让zk在连接之前等待,连接成功后才能往下走.
   private static final CountDownLatch countDownLatch = new CountDownLatch(1);
   private static String LOG_MAIN = "【main】 ";
   private ZooKeeper zk;
   public void createConnection(String connectAddres, int sessionTimeOut) {
    try {
      zk = new ZooKeeper(connectAddres, sessionTimeOut, this);
      System.out.println(LOG_MAIN + "zk 开始启动连接服务器....");
      countDownLatch.await();
    } catch (Exception e) {
      e.printStackTrace();
    }
   }
   public boolean createPath(String path, String data) {
    try {
      this.exists(path, true);
      this.zk.create(path, data.getBytes(), Ids.OPEN_ACL_UNSAFE,       CreateMode.PERSISTENT);
      System.out.println(LOG_MAIN + "节点创建成功, Path:" + path + ",data:" + data);
    } catch (Exception e) {
      e.printStackTrace();
      return false;
    }
    return true;
   }
   /**
   \* 判断指定节点是否存在
   *
   \* @param path
   \*      节点路径
   */
   public Stat exists(String path, boolean needWatch) {
    try {
      return this.zk.exists(path, needWatch);
    } catch (Exception e) {
      e.printStackTrace();
      return null;
    }
   }
   public boolean updateNode(String path,String data) throws KeeperException,   InterruptedException {
    exists(path, true);
    this.zk.setData(path, data.getBytes(), -1);
    return false;
   }
   public void process(WatchedEvent watchedEvent) {
    // 获取事件状态
    KeeperState keeperState = watchedEvent.getState();
    // 获取事件类型
    EventType eventType = watchedEvent.getType();
    // zk 路径
    String path = watchedEvent.getPath();
    System.out.println("进入到 process() keeperState:" + keeperState + ", eventType:" + eventType + ", path:" + path);
    // 判断是否建立连接
    if (KeeperState.SyncConnected == keeperState) {
      if (EventType.None == eventType) {
        // 如果建立建立成功,让后程序往下走
        System.out.println(LOG_MAIN + "zk 建立连接成功!");
        countDownLatch.countDown();
      } else if (EventType.NodeCreated == eventType) {
        System.out.println(LOG_MAIN + "事件通知,新增node节点" + path);
      } else if (EventType.NodeDataChanged == eventType) {
        System.out.println(LOG_MAIN + "事件通知,当前node节点" + path + "被修改....");
      }
      else if (EventType.NodeDeleted == eventType) {
      System.out.println(LOG_MAIN + "事件通知,当前node节点" + path + "被删除....");
    }
   }
   System.out.println("--------------------------------------------------------");
 }
   public static void main(String[] args) throws KeeperException, InterruptedException {
    ZkClientWatcher zkClientWatcher = new ZkClientWatcher();
    zkClientWatcher.createConnection(CONNECT_ADDRES, SESSIONTIME);
    // boolean createResult = zkClientWatcher.createPath("/p15", "pa-644064");
    zkClientWatcher.updateNode("/pa2","7894561");
   }
 }
```