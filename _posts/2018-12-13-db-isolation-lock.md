---
layout: post
title:  "数据库隔离级别和锁"
date:   2018-12-13 22:45:52
categories: 隔离级别 锁
published: true
comments: true
thread: 20181213224555555
---
数据库隔离级别和锁
---
> [数据库事务、隔离级别和锁](https://www.jianshu.com/p/cb97f76a92fd)

## 一、隔离级别和并发控制
SQL92标准定义了四种隔离级别——Read Uncommitted，Read Committed，Repeatable Read和Serializable。
定义这4种隔离级别时，制定者主要围绕着基于锁的并发控制来说的。但是后来出现了MVCC，之后主流数据库都开始支持MVCC。有的数据库采用比较纯粹的MVCC实现，比如PostgreSQL；有的则是混杂的，比如MySQL InnoDB。这就会造成数据库的实现和标准的描述有很多出入。


## 二、最不严格的隔离级别
最不严格的隔离级别应该是不隔离。
不隔离很容易理解，不同的事务可以对同一数据并发的随便改：A事务改了一半的结果B能看到；B改了一半的结果A也能看到；如果A和B反复修改同一个数据，那么彼此的修改可以覆盖。数据系统在没有做隔离防护时，就一定会是这个样子。这样也就无所谓事务了。
这里数据访问冲突可以分为两种：
- `Dirty Read`，脏读。即一个事务的没提交之前的修改被另外一个事务可以看到。
- `Dirty Write`，脏写。即一个事务的没提交之前的修改可以被另外一个事务的修改覆盖掉。脏写是无法被接受的，因为他会让事务原子性无法实现.

任何支持事务的数据库都有一个基本原则：不论隔离级别是什么，脏写都是不允许的！！
一般数据库都会使用排他锁来标记要修改的数据（update，delete，select … for update)。锁的存在可以保证——写要block写。这个规则永远生效。

事务数据库对于写操作永远需要锁来避免脏写，即使是基于MVCC的数据库。所谓某个隔离级别使用MVCC不需要锁，仅仅是指在读取的时候是否需要锁。


## 三、Read Committed和Repeatable Read
这俩种隔离级别放一起说是因为它们的基本原则是一样的：`读不block读和写，写不block读`

事务A  | 事务B
--|---
get x (得到0)	  |     
 - | set x = 1
-   | commit
get x (得到1)	  |  
commit  |  

这里可以看到事务A对x的两次读取，因为发生在事务B对x修改的前后，得到了不同的结果。事务A可以看到事务B已经提交的修改。


事务A  | 事务B
--|---
get x (得到0)	  |     
 - | set x = 1
 - | commit
get x (得到0)	  |  
commit  |  

Repeatable Read的直观感觉仿佛是给事务做一个整个数据库做了一个快照，所以很多时候这种隔离级别又被称为Snapshot Isolation。


外值得一提的是幻读的问题。在SQL92标准中提到了Repeatable Read中是可以出现幻读的——即一个事务尽管不能读取到后续其他事务对现有数据的修改，但是能够读取到插入的新数据。但是，基于MVCC的实现，Repeatable Read可以完全避免幻读（这岂不是更好）。`无论MySQL还是PostgreSQL在Repeatable Read隔离级别都不会出现幻读。`

## 四、MVCC
MVCC是"Multi-Version Concurrency Control"的缩写。名字看上去很吓唬人，有点不明觉厉，但是可以这样简单理解——对数据库的任何修改的提交都不会直接覆盖之前的数据，而是产生一个新的版本与老版本共存，使得读取时可以完全不加锁。这个版本一般用进行数据操作的事务ID(单调递增）来定义。`MVCC大致可以这么实现：
每个数据记录携带两个额外的数据created_by_txn_id和deleted_by_txn_id。`

- 当一个数据被insert时，created_by_txn_id记录下`插入该数据的事务ID`，deleted_by_txn_id`留空`。
- 当一个数据被delete时，该数据的deleted_by_txn_id记录`执行该删除的事务ID`。
- 当一个数据被update时，原有数据的deleted_by_txn_id记录`执行该更新的事务ID`，并且新增一条新的数据记录，其created_by_txn_id记录下`更新该数据的事务ID`

在另一个事务进行读取时，由隔离级别来控制到底取哪个版本。同时，在读取过程中，完全不加锁（除非用SELECT … FOR UPDATE强行加锁）。这样可以极大降低数据读取时因为冲突被Block的机会。
那么那些多出来的无用数据怎么被最终被清理呢？支持MVCC的数据库一般会有一个背景任务来定时清理那些肯定没用的数据。`只要一个数据记录的deleted_by_txn_id不为空，并且比当前还没结束的事务ID中最小的一个还要小，该数据记录就可以被清理掉`。在PostgreSQL中，这个背景任务叫做“VACUUM”进程；而在MySQL InnoDB中，叫做“purge“。

#### 4.1 MVCC在隔离级别上的体现
在PostgreSQL的实现中，MVCC产生的所有版本的节点都生成存储数据表的B+树的节点。新的节点和老的节点并存，只是上边的标记不同。这个实现的好处是选择读取哪个版本非常方便，可以和B+树的搜索算法合并到一起，还能兼顾SSI检测（下文会提到）。坏处是清理废弃数据相对麻烦

MySQL采用Undo Log的实现。这种实现下，用于存储数据表的B+树节点总是只保留最新的数据，而老版本的数据被放在Undo Log里，并且以指针的形式关联起来，形成一个链表。这样，在查找老的版本时，需要按链表顺序查找，直到找到created_by_txn_id <= 当前事务ID的最新那条记录即可。这种实现的优缺点和PostgreSQL正相反，查询时会在B+树查找后多引入一个链表查询；但是清理废弃数据时会更简单，只要把Undo Log找到一个合适的位置一刀切了即可。

有了MVCC，Read Committed和Repeatable Read就的实现就很直观了：
- 对于Read Committed，每次读取时，总是取最新的，被提交的那个版本的数据记录。
- 对于Repeatable Read，每次读取时，总是取created_by_txn_id小于等于当前事务ID的那些数据记录。在这个范围内，如果某一数据多个版本都存在，则取最新的。


隔离级别|MySQL|PostgreSQL
-|-|-
Read Uncommitted|支持|不支持，等价于Read Committed
Read Committed|支持，基于MVCC实现|支持，基于MVCC实现
Repeatable Read|支持，基于MVCC实现了Snapshot Isolation，可避免幻读|支持，基于MVCC实现了Snapshot Isolation，可避免幻读
Serializable|支持，Repeatable Read + 共享锁|支持，基于MVCC实现了Serialized Snapshot Isolation
默认级别|Repeatable Read|Read Committed
MVCC实现|基于Undo Log|基于B+树直接记录多个版本
