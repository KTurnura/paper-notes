# # 论文笔记

看到有前辈在github上记录自己读论文的一些记录，以此也想记录并分享一些自己对论文的一些看法。从九月份刚入学开始，读了一部分博客推荐的经典论文，后面开始跟MIT 6.824的课程，MIT课程的代码可能会在repository中更新并解读，关于论文的一些想法会在[issue](https://github.com/KTurnura/paper-notes/issues)中记录，有对论文有见解的可以在issue中评论，也可以分享一些您认为某篇论文的扩展和值得读的论文

后续一些关键论文的解读可能会放在自己的 [博客](https://github.com/KTurnura/KTurnura.github.io) 中，如果准备对某篇博客进行解读会在issue中指出

关于readme，打算周期性展示一个月内读过的论文，然后每月更新之后归档上一个月的论文

[toc]

#  1. architecutre

## 1.1 Lambda

## 1.2 Kappa

## 1.3 summingbird ： twitter

## 1.4 Kappa vs Lambda

# 2. Database 

## 2.1 Scalable SQL and Nosql data stores

# 3. File System

## 3.1 GFS

# 4. Data Storage Layer

## 4.1 BigTable



# # 5. CAP theorem && ACID

## 5.1 **Time, Clocks, and the Ordering of Events in a Distributed System**

本文定义了Happened Before 关系， 由于物理时钟有误差，所以本文考虑逻辑时钟，最终目的就是为了的到一种事件全局排序的机制。而由于事件排序是偏序的。所以定义了“Happened Before关系”。 逻辑时钟给事件打上了时间戳，可以通过比较事件的时间戳数值来判断事件的发生次序（Happened Before 关系）。逻辑时钟给事件打时间戳的时候，需要满足一定条件，该文定义了一个时钟条件。 该文详述的证明了该条件，且该条件也需要满足其他条件[[6_总结#5]]

但时钟条件具有单向推导逻辑的限制，我们不能根据两个事件对应的时间戳在数值上的大小（因为"Happened Before 具有偏序性"）来推断出他们之间是否存在"Happened Before"关系。

此时给不同事件指定时间戳，虽然没有意义，但可以按照该时间戳从小到大把所有事件都排成一个序列，那么就得到了分布式系统中所有事件的全局排序。此时所有事件的"Happened before 关系就都被保持住了".即 这种排序和"Happened Before "顺序保持一致

而更近一步，事件的全局排序结合状态机复制（State Machine Replication）的思想，几乎可以为任何分布式系统的设计提供思路

然而，在分布式领域，逻辑时钟仍然存在一些异常行为，有时候行为是发生在系统外部，系统内部并不知晓，此时，我们又需要真实物理事件作为基准。但由于不同机器的物理时钟之间存在偏差，我们需要使物理时钟满足Strong Cock Condition 来保证系统总能对正确的顺序进程打上合理的时间戳。
**而时空本身就是一种偏序**，在相对论中，事件的排序是根据**可能**发送的消息来定义的。然而，我们这里采取了更务实的做法，仅仅考虑那些实际上已经发送过的消息

此时我们需要运行一个时钟同步算法，来保证即使两个进程之间存在偏序关系，仍能为不同进程的不同事件打上合理的时间戳。
在论文中，考虑物理时钟的两种误差，不断对各进程本地的物理时钟进行微调，将误差控制在能够满足Strong Clock Conditionde 的范围内
a) :进程内，保证时钟不回退
b)： 进程外，在不同进程的物理时钟之间交换信息，借助这些信息同步时钟读书，将不同进程的物理时钟误差控制在一定范围内

它还划定了系统的能力边界。它告诉我们，什么样的问题可以在系统内部，遵循一个纯异步的模型（asynchronous model）框架就能解决（比如非拜占庭模型下的共识问题）；而什么样的问题，必须求诸系统的“外部”（也就是物理世界）才能得到解决（比如拜占庭模型下的共识问题、线性一致性问题等）。所有这些，都深深地影响了人们对于分布式系统的思考方式。s in a Distributed System

## 5.2 1982_Byzantine Generals Problem

本文介绍了拜占庭问题
在最开始全通量图的情况下，给出了口头消息和签字消息的 解拜占庭问题的基本条件。

1. 口头消息
   1. ==少于3m+1个将军，无法处理m个叛徒的问题==


后序给出了在不完全连接图的时候在口头消息和签字消息的解法：

然后给出了现实中实现Signature Algorithm 算法的例子

## 5.3 2005_**Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services**

三选二，然后部分解决办法

## 5.4 **2012_ CAP Twelve years later how the rules have changed**

## 5.5 2008_An ACID Alternative : BASE

如果ACID提供了分区数据库的一致性选择，那你可以选择什么样的替代品呢？ 一个答案就是BASE (basically available, soft state, eventually consistent).

BASE 是ACID的反面， ACID是悲观并且强一致性，对于每一个操作。 BASE 是乐观，接收数据库一致性是一个可以变动的状态。 虽然听起来不可能，在现实中，它更容易管理去实现扩展性，这个是ACID不能实现的。

==BASE的可用性可以通过以下方式获得，支持部分出错，但是整体还是工作的。==
即使出错，也不影响该系统的整体可用性

为了实现分布式系统的松一致性，可以将数据库中的表进行解耦，类似于先对一个表进行更新，然后在更新另一个表，相对于一个表更容易保持一致性

假设我们解耦了更新user 和 transaction 表。这两个表的一致性不保证。如果在第一个事务和第二个事务之间发生了错误，那么这两个表就永远不能一致了，如果只要求一个大概统计，那么还是可以接受的。

如果估计不能被接受，则需要引入新的组件来解耦两个表的更新

引入一个持久化的消息队列来解决这个问题。

在交易插入时候，用入队列的方式持久化消息，他和这消息在user被需要时后要进行更新以便达到平衡。

一个独立的消息处理component将从消息队列中，取出每一个消息，并将这些信息写入user table。看来这个例子解决了所有问题，但是有个问题。如果消息里还有user host的联系，那么还是有2PC情况。

消息处理进程 对于2PC问题的一个解决方案， 啥都不做。解耦这个更新到一个分离的后台component， 保持顾客可用行。消息处理的稍低的可用性可以被接收。

虽然有幂等性，但并不是所有的事务都满足幂等性，

本文给出了一个例子，该例子依赖于在队列里找一个消息处理，处理之后再移除。这可以在两个独立的交易，一个是在消息队列，另外一个在user数据库。queue操作只有当数据库操作成功的时候才commit。这个算法支持部分失败，可以保证交易而没有使用2PC

有一个简单的技术可以保证更新的幂等性，如果只考虑顺序。假定你想查看user最后一天的销售和采购，你可以依赖一个一样的schema更新用这个日期用message；还有一个办法是使用时间，也可以使用自增性transaction ID


消息的有序性也相当有必要， 但如果实现消息系统能够确保消息按照接收顺序传递的成本还很高，而且会导致一些自以为解决了的错误。
我们可以放宽消息排序的要求并最终仍然提供一致的数据库视图。 

本文一直所讲的，就是使用一致性来换取可用性，在另一方面，即使用软状态和最终一致性，就可以实现一个比较好的分布式系统。
如果确实需要知道状态是否达到了一只，可以使用EDA（事件驱动架构），获得使状态变得一致生成的事件。

传统服务需要进行横向扩展时，传统事务模型就会出现问题， 解耦操作并依次执行它们可以提高可用性和规模以一致性为代价。 BASE 提供了一个思考这种解耦的模型。

## 5.6 Zookeeper : A simple totally ordered broadcast protocol

Zab协议是为分布式协调服务Zookeeper专门设计的一种支持崩溃恢复的原子广播协议，是Zookeeper保证数据一致性的核心算法。

在Zookeeper中主要依赖Zab协议来实现数据一致性，基于该协议，zk实现了一种主备模型（即Leader和Follower模型）的系统架构来保证集群中各个副本之间数据的一致性。 这里的主备系统架构模型，就是指只有一台客户端（Leader）负责处理外部的写事务请求，然后Leader客户端将数据同步到其他Follower节点。

Zookeeper 客户端会随机的链接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据；

如果是写请求，那么节点就会向 Leader 提交事务，Leader 接收到事务提交，会广播该事务，只要超过半数节点写入成功，该事务就会被提交。

基于ZAB协议，Zookeeper实现一种**主备模式的系统架构来保持集群中主备副本之间数据的一致性**。

ZAB协议包括两种基本模式：消息广播（Message Broadcasting）和崩溃恢复（Leader Activation）。
Zookeeper 消息广播顺序

1. 读请求直接发生在Follower上，如果发生写请求，则Follower 将请求转发给Leader
2. 当Follower 接收到事务请求时，会将该请求转发给Leader，让Leader 来进行处理Follower 在接收到事务后，会向所有Follower节点发送Proposal 提议，并等待各个节点的Ack 反馈，
3. 在广播事务之前Leader服务器会先给这个事务分配一个全局单调递增的唯一ID，也就是事务ID（zxid），每一个事务必须按照zxid的先后顺序进行处理。
4. 各个Follower 对Leader 节点进行Ack反馈，Leader 对接受到的Ack进行统计，如果超过半数的Follower 进行了Ack，则执行该事务，否则向客户端进行事务请求失败的Response
5. 如果Leader节点接收到了超过半数的Ack响应，此时Leader会向所有的Follower发出事务Commit的指令，同时自己也执行一次Commit，并向客户端进行事务请求成功的Response。

Zookeeper的消息广播过程类似 2PC（Two Phase Commit），ZAB **仅需要超过一半以上的Follower返回 Ack 信息就可以执行提交，大大减小了同步阻塞，提高了可用性**。


除消息广播之外，Zookeeper还有崩溃恢复的功能，在上文[[11_Paper_Ref]]中做了更详细的介绍
Leader选举场景

1. Zookeeper启动
2. Leader崩溃

重点介绍zxid ： 由于只有Leader才能进行Proposal，所以这个zxid很容易做到全局唯一且自增。因为Follower没有生成zxid的权限。zxid越大，表示当前节点上提交成功了最新的事务，

==Zab 协议通过 epoch 编号来区分 Leader 变化周期，能够有效避免不同的 Leader 错误的使用了相同的 zxid 编号提出了不一样的 Proposal 的异常情况==

## 5.7 The Design of a Practical System for Fault-Tolerant Virtual Machines

基于通过另一台服务器上的备份虚拟机复制主虚拟机的执行的方法，实现了一个企业级的容错系统

区别于常见的容错系统： Primary / backup approach

## 5.8 Raft : 2014 Raft In Search of Undersatandable consensus Algorithm

## 

# 6. Batch processing

## 6.1 Mapreduce




# 7. Blog

## 1. you can't sacrifice partition tolerance

https://codahale.com/you-cant-sacrifice-partition-tolerance/

```
// TODO
```

## 2. Challenges of Data Stream Processing : big data Streams 1:!

https://nexocode.com/blog/posts/data-stream-processing-challenges/

```
// TODO
```

## 3. **条分缕析分布式：到底什么是一致性？**

http://zhangtielei.com/posts/blog-time-clock-ordering.html

区分Consensus 和consistency ， 
Consensus 共识算法： Paxos
Consistency 一致性：线性一致性
所谓的强一致性只与CAP定理有关系，但CAP定理现在并没有办法涵盖分布式领域的主要问题

Consistency ： 它指的是任何一个数据库事务的执行，都应该让整个数据库保持在「一致」的状态。
大多数用于数据库的ACID 特性
比如使用： 2PC和3PC属于 原子提交协议两种不同的具体实现
而原子提交是ACID 中Atomic（原子性的一部分

而新论文将原子提交问题抽象为了新的一致性问题： 称为Uniform consensus： 要求所有节点（包括故障节点都要达成共识）

而通常的consensus 问题通常只关注没有发生故障饿节点达成共识
例如： Paxos 算法和解决拜占庭将军问题的算法

CAP理论： C（Consistency）  A（Available ：可用性） P(Partition-tolerance)只能同时满足三个

但在CAP理论中，C指的线性一致性：系统表现的就像只有一个副本一样

传统对CAP理论的一致性介绍都颇为片面

1. 在分布式系统完成某写操作后的任何读操作，都应该获取到该写操作写入的那个最新的值。 只是线性一致性的特例
2. **保持所有节点在同一个时刻具有相同的、逻辑一致的数据**“ 。 以偏概全

分布式事务处理的并不是同一个数据对象的多个副本的问题，而指的是将针对多个数据对象的各种操作组合起来，提供ACID的特性。将分布式事务看成是强一致性的保证，猜测可能实际上指的就是ACID的原子性。总之，「强一致性」这个词很容易产生误解，所以建议谨慎使用。

## 4. **条分缕析_强弱一致性**

http://zhangtielei.com/posts/blog-distributed-strong-weak-consistency.html

跟大家继续讨论顺序一致性、线性一致性、最终一致性等几个概念。

为了避免产生歧义，我们先明确一下这几个概念的英文表达：

- 顺序一致性的英文是：_sequential consistency_。
- 线性一致性的英文是：_linearizability_。实际上，它就是CAP定理中的C，我们在[上一篇文章](https://mp.weixin.qq.com/s/qnvl_msvw0XL7hFezo2F4w)中已经提到过。
- 最终一致性的英文是：_eventual consistency_。


在进行详细的技术性讨论之前，我们先把本文要讨论的几个重点问题和结论列出如下：

- 线性一致性和顺序一致性，属于分布式系统的一致性模型 (_consistency model_)。这代表了分布式系统的一个非常非常重要的方面。
- 通常人们把线性一致性称为「强一致性」，把最终一致性称为「弱一致性」，但线性一致性和最终一致性其实存在本质的区别。严格来说，它们并不是一个范畴的概念。
- 一致性模型之间的「强弱」比较，是一个相对的概念。比如，线性一致性是比顺序一致性更强的一致性模型。当然，除了线性一致性和顺序一致性，也存在其它一些一致性模型（其中很多都比顺序一致性要弱）。
- 满足线性一致性的系统，也必定满足顺序一致性，但反过来不一定。这是由一致性模型之间的强弱关系决定的。

线性一致性> 顺序一致性> 最终一致性

## 5. **漫谈分布式系统、拜占庭将军问题与区块链**

http://zhangtielei.com/posts/blog-consensus-byzantine-and-blockchain.html