# 1

## mysql的存储引擎架构的特点是什么？

将处理和存储分离，存储可以根据业务类型来选择。

逻辑架构包括客户端、服务器层和引擎层。服务器层的核心功能包括查询解析、分析、优化、缓存、内置函数，以及所有的跨存储引擎功能（如存储过程、触发器和视图等）。存储引擎通过门面API返回服务器层的操作结果，不解析sql（InnoDB会解析外键而例外）。

## mysql存储引擎实现的锁有哪些？

表锁、行锁，共享读锁、排他写锁，意向排他锁。

## mysql如何提交事务？

mysql默认自动提交，手动提交时，除了commit语句，DDL等语句也会导致自动提交，结束事务。

## MyIsam/InnoDB/XtraDB/TokuDB的特点分别是什么？

InnoDB将每个表的数据和索引放在单独的文件中，表是基于聚簇索引建立的，B+树结构对主键查询的性能很高，但要求非主键索引包含主键列。

XtraDB引擎是基于InnoDB的改进版本，兼容可替代。TokuDB引擎使用分形树索引结构，与缓存无关，可以用于大数据处理。

## InnoDB如何解决幻读？

InnoDB支持高并发的MVCC多版本并发控制(与间隙锁一起解决幻读)只发生在RR隔离级别上，在每行记录后面保存两个隐藏列来时间的，分别保存一些额外的存储信息： DATA\_TRX\_ID(6字节，事务id, 自增的系统版本号)，DATA\_ROLL\_PTR(7字节，undo日志和找过去版本的数据)，DB\_ROW\_ID(6字节，包含于聚簇索引)，DELETE BIT(删除标志位对删除事务操作的快照有效)。

创建和过期删除的系统版本号。每开始一个新的事务，系统版本号递增，同时对操作行生成一个快照。

事务内查询只查找版本号不大于当前事务系统版本号的数据行，可以保证事务读取的数据不受后续事务的影响。删除版本未定义或者大于当前事务版本号，则保证在事务开始之前未删除。



## InnoDB发生死锁如何处理？

InnoDB检测到死锁依赖时立即返回错误，防止特别慢的查询。发生死锁时，将持有最少行级排他锁的事务进行回滚。

## B树和B+树的演化过程和区别是什么？

二叉树，平衡二叉树，多叉树，B树，B+树

## B树,B-树,B+树,B\*树的区别

B树(Binary Tree)是二叉树。特点是：非叶子节点最多2个节点，节点都带数据，而且子节点数据值左小大。B-树(Balance Binary Tree)是平衡二叉树。特点是：左右叶子节点高度相差不超过1，左右都是B-树，节点都有数据。B+树，即通常在数据库索引数据结构提到的B树或者B-树。结构与B-树有所不同，数据在叶子节点，非叶子节点相当于不存储数据的索引，叶子节点横向也有横向链指针。B\*树是B+树的变体，非叶子节点也添加横向链指针。

## B+树的优点是什么

内部节点可以集中存储，又可以载入内存，都减少了磁盘io。查询数据的路径长度近似，效率稳定。

大多数mysql引擎都支持的B-Tree索引(InnoDB使用B+Tree)通过比较非叶子节点指向下级节点的上下限指针来确认需要查找的下层节点。叶子节点的指针指向数据，顺序的存储结构适合范围查找。

## MyIsam引擎和InnoDB引擎的区别是什么？

MYISAM是按列值与行号来组织索引的。它的叶子节点中保存的实际上是指向存放数据的物理块的指针。
从MYISAM存储的物理文件我们能看出，MYISAM引擎的索引文件（.MYI）和数据文件(.MYD)是相互独立的。

聚簇索引中的每个叶子节点包含主键值、事务ID、回滚指针(rollback pointer用于事务和MVCC）和余下的列(如col2)。

 INNODB的二级索引与主键索引有很大的不同。InnoDB的二级索引的叶子包含主键值，而不是行指针(row pointers)，这减小了移动数据或者数据页面分裂时维护二级索引的开销，因为InnoDB不需要更新索引的行指针。

## MYSQL常用优化（sql优化，表结构优化等）

SQL优化、表机构优化、索引优化、缓存参数优化

## 利用mysql进行微博数据库的设计，假设注册用户10000，该用哪种方式呢 ？

## sql优化的方式

慢查询日志和执行计划。

## MySQL集群的主从复制怎么做的，具体有哪些线程做哪些事情，使用了哪些日志。

复制线程把二进制日志异步复制到从机，成为中继日志。从机的线程重放中继日志，写入数据库。

# 性能检查

## 如何检查sql性能？

`long_query_time=0` 的设置可以捕获所有慢查询，写入慢查询日志。服务器写磁盘的权限不够时，可以通过 `tcpdump` 抓取mysql的tcp网络包保存并使用pt-query-digest工具分析。这样定位到慢查询语句。

*show profile* 命令用于进行sql分析，通过命令行设置 `set profiling=1` 可以在会话级别启用这个有点耗费资源的功能。还可以通过 `_information_schema.profiling` 表来查询并排序输出。

`show status`命令返回了全局和会话的计数器，反映了某些操作的数量（比如创建的临时表和磁盘临时表的数量，explain不区分临时表位置）。 `show global status` 则是从服务器启动时就开始的全局计数。

间歇性问题（如系统停顿或者慢查询）的解决不要反复试错，太低效。可能的原因有：curl从外服务器获取数据耗时，缓存击穿，dns查询超时，锁竞争，查询缓存没有被禁用，并发过高导致查询计划的优化耗时。

查询慢不一定是sql问题，整体查询都变慢，就可能是服务器问题。通过三种方法定位：

`show global status` 每秒执行一次，可以通过某些计数器（比如Threads\_running, Threads\_connnected, Questions和Queries）的“尖刺/凹陷”来发现。使用了连接池的服务器的连接线程数变化不大。

`show processlist` 通过反复捕获输出来观察线程状态，还可以直接查询information_schema.processlist表。

慢查询日志的分析，除了工具，还可言使用简单的命令行

`awk '/^# Time:/{print $3, $4, c;c=0}/^# User/{c++}' slow-query.log`
统计每秒的查询数量，可以从“尖刺/凹陷”推测服务器中断的情况。

## 如何优化索引

适合扩展的冗余索引、重复索引和未使用的索引都删除。

InnoDB只有在访问行的时候才会加锁，索引通过减少磁盘io对行数据的回表访问减少了锁的数量。但即使使用了索引，查询计划分析后也可能锁住结果之外的数据。

对少量值项的列加索引，在使用前置的全IN查询条件的时候可以让扩展的多列索引生效。IN查询的选项不要过多，也不要组合使用。

不要使用多个范围条件，否则新增一个对范围条件列合并计算的新的索引列。连续的多个等值IN查询优于范围查询的是可以使用后续索引，二者在执行计划上都表现为范围查询，在一定程度上可以互换。

优化排序，增加索引可以加快同时使用order by和limit的查询。如果limit偏移量过大的翻页功能要扫描数据然后丢弃，那么可以从业务上限制翻页数量，只提供靠前的搜索结果。还可以先按条件查询出主键，然后延迟关联到需要的行。

```optimize table```和```alter table [table] engine=[engine_now]```都可以减少索引和数据的碎片。

## 查询性能优化

查询性能低下最基本的原因是访问的数据太多。那么就需要确认是否访问了过多的行或列，确认服务器层是否在分析过多的行。

mysql衡量查询开销的最简单的3个指标是：响应时间（包括服务时间和排队时间，可以通过计算顺序和随机io数量估算具体硬件条件下的时间）、扫描的行数和返回的行数（比例多为1-10）。explain的type列的访问类型从慢到快依次为：全表扫描(ALL)、索引扫描(index)、范围扫描(range)、唯一索引查询(ref)、常数引用(const)等。如果没有合适的访问类型，就增加一个合适的索引。

mysql使用where条件的方式从好到坏依次为：存储引擎层在索引中使用where条件过滤不匹配的记录。服务器层使用索引覆盖扫描来返回记录，过滤后返回结果，无需回表。服务器层从数据表中返回记录，然后过滤（Extra列为Using where）。

重构查询的方式包括：权衡复杂查询和简单查询，分段切分查询（降低事务过大的影响），分解关联查询（利于queryCache的缓存，减少锁竞争，利于分库分表，不需要缓存中间结果，可构成哈希关联）。

mysql执行查询的过程包括：客户端发送查询给服务器。服务器首先检查查询缓存，命中则返回。否则在服务器层进行sql解析、预处理，再由优化器生成对应的执行计划。根据执行计划调用存储引擎的api来执行查询。返回结果给客户端。

mysql语法解析器把sql语句解析成“解析树”，验证查询语句的语法规则。预处理器则检查解析树是否与实际的数据表对应，并验证服务器权限。

count(*)查询的通配符会被忽略，查询优化可以从业务上改为近似值，mysql使用索引覆盖扫描，增加汇总表或者外部缓存。

关联查询的优化，需要确保on或者using子句的列上有索引，而且只需要在第二个表上建立索引。需要确保group by和order by的表达式只涉及一个表中的列，这样mysql才可能使用索引优化。升级mydql的时候可能把普通关联变成笛卡积，不同类型的关联可能结果不同。

union查询总是被通过创建并填充临时表的方式来执行，因此需要把where, limit, order by等子句冗余添加到子查询中，使优化器充分利用这些条件优化。如果不需要消除重复行，就使用union all. 否则被自动添加distinct来检查唯一性。

子查询的优化，尽量用关联查询代替，但在使用mysql5.6或者MariaDB的情况下可以忽略。

group by和distinct查询都可以使用索引优化，否则mysql使用临时表或者文件排序来分组。关联查询采用查找表的标识列分组的效率会比其他列更高。select后面的非分组列直接依赖于分组列，并且在每个组内的值是唯一的，或者业务上不在乎这个值。如果没有通过order by子句显式指定排序列，当查询使用group by子句的时候，结果机会自动按照分组的字段进行排序。如果不关心结果集的顺序，就可以使用```order by null```禁止文件排序。也可以在group by子句中直接使用desc/asc关键字将分组结果集按照需要的方向排序。

limit分页的优化，在索引覆盖扫描后延迟关联查询。也可以转换为已知位置的范围查询。如果范围查询进一步对自增主键降序排序，那么就可以优化大偏移量的limit查询。其他优化包括预先计算的汇总表。

# 复制

## mysql复制过程

复制是主从架构，基于行或者语句的复制方式都是通过在主库上记录二进制日志，在备库重放来实现异步的数据复制。

复制过程：
1. 主库在事务提交前把更新事件记录到二进制日志，然后通知引擎提交事务。
1. 备库的IO线程与主库建立一个普通的客户端连接，然后在主库上启动一个特殊的二进制转储线程，用于读取二进制日志的事件，通过连接发送到备库。该线程读完已有的事件就通过睡眠方法等待被新事件的信号量通知唤醒。备库IO线程则将接收到的事件记录到中继日志中。
1. 备库sql线程从中继日志中读取事件并执行，赶上io线程的记录时，中继日志通常已经在系统缓存中，因此开销低。

配置主从复制的步骤：创建复制帐号，配置主库和从库的log_bin和server_id等（备库的log_slave_updates=1配置允许将重放事件也记录到二进制日志中），在备库上变更主库位置并启动备库而开始复制。主库的二进制文件名和偏移量可以通过```show master status```来查看。

主库上的二进制日志最重要的配置项```sync_binlog=1```使用同步刷盘策略，保证服务器崩溃时不丢失事件。log_bin和relay_log的明确指定可以避免日志文件默认通过机器名命名在换机器时的混乱。
skip_slave_start可以组织备库在崩溃后自动启动复制，这样可以先修复可能的问题。

可建议使用的复制拓扑包括：一主多备，主动-被动模式的主主复制（在主动端暂停复制，在被动端修改表结构后，角色反转并启动原主动端的复制线程，可以避免锁表操作），增加备库的主主复制。其他如主动-主动模式的主主复制、主库-分发主库-备库、环形复制、树形复制都是不建议使用的。

## 分库分表中间件的原理

Mycat, Cobar在xml配置文件中定义路由函数。

sharding-jdbc则是改写sql，合并结果实现的。
