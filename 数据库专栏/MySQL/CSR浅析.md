# 名词介绍

事务如何保证ACID？这个就要从InnoDB存储引擎在存储数据的底层逻辑上开始说起了，本章节将介绍以下3个重要的知识点：

- redo_log事务重做日志
- undo_log事务回滚日志
- CSR自动故障恢复机制

前两者能保证事务的A（原子）C（一致）I（隔离）性，而后者则能保证事务的D（持久）性和C（一致）性。

在开始之前我们需要了解以下的一些名词：

- CSR：MySQL自动故障恢复机制的简称，而InnoDB存储引擎能够支持该机制
- 磁盘数据页文件：它是存储记录以及索引的文件，即数据库目录下的tableName.ibd文件
- data buffer pool：译为数据缓冲池，从磁盘中加载的记录以及索引信息均存放至此，它位于内存中
- redo_log：事务重做日志，CSR中用于前滚操作所必须依赖的日志，同时事务提交COMMIT操作也需要依赖该日志，它位于data目录中，有2个文件，名称为ib_logfile0和ib_logfile1，默认50M大小，轮询使用
- undo_log：事务回滚日志，CSR中用于回滚操作所必须依赖的日志，同时事务回滚ROLLBACK操作也需要依赖该日志，它在MySQL5.7版本里仍位于共享表空间ibdata文件中，8.0版本后被独立出来
- redo log buffer：redo log的内存缓冲区，它位于内存中
- undo log buffer：undo log的内存缓冲区，它位于内存中
- LSN号：日志序列号，每次MySQL数据库启动时，都会比较磁盘数据页文件（tableName.ibd）的LSN号和redo log的LSN号，两个LSN号必须一致数据库才能正常启动，它位于tableName.ibd、data buffer pool、redo_log、redo log buffer中
- WAL：持久化存储方案的一种，全称为Write Ahead Log，译为日志优先写，MySQL以该方案实现持久化，本质是日志优先于数据写入磁盘
- 脏页：脏页是位于内存中的，由于WAL机制的存在，当数据页在内存中发生修改但没写入到磁盘之前我们把data buffer pool中的数据页称之为脏页，其页中的数据页被称为脏数据
- CKPT：检查点，全称为Checkpoint，实际上就是将脏页刷写到磁盘的动作
- TXID：事务号，全称为Transaction ID，InnoDB引擎会为每一个事务生成一个事务号并伴随着整个事务的生命周期



# 事务原理

## BEGIN

下面我们开启一个事务，并进行COMMIT操作：

```
BEGIN;

    UPDATE
        userInfo
    SET
       age = 21
    WHERE
        name = "Jack";
        
-- 注意！未进行COMMIT或者ROLLBACK操作
```

MySQL内部处理机制如下：

1. 在使用BEGIN开启事务时，会先给userInfo.ibd文件中分配一个TXID号和LSN号，假设为tx01与lsn01
2. 在UPDATE执行时，MySQL会找到需更新数据的数据页，并将其内容加载到data buffer pool中，由DBWR（double write）线程记录变更数据页的内容，并且记录好TXID和更新LSN号，此时将产生脏页与脏数据
3. 使用LOGBWR（log double write）线程，将更新前的数据页变化内容与TXID号以及LSN号记录到undo log buffer中
4. 使用LOGBWR（log double write）线程，将更新后的数据页变化内容与TXID号以及LSN号记录到redo log buffer中
5. 现在，基于WAL原则为了应对用户进行ROLLBACK操作，LOGBWR（log double write）线程会将undo log buffer中的内容全部写入到undo_log即ibdata1（MySQL5.7版本undo_log仍然存在于共享表空间中）文件中

图示如下：

![image-20211113234550877](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211113234550877.png)



## COMMIT

现在我们基于上面BEGIN后发生的情况执行COMMIT操作，内部会做以下的3件事：

6. 执行COMMIT操作，基于WAL原则，LOGBWR线程会先将redo log buffer中记录的日志信息写入redo_log文件中，在日志信息完全写入redo_log即ib_logfile文件后，会对该日志打上COMMIT的标志
7. 触发CKPT，将内存数据页更新到磁盘中，并且更新磁盘数据页文件userInfo.ibd中原有的LSN号，让其与redo_log文件中的LSN号保持一致
8. 清空内存undo log buffer、data buffer pool、redo log buffer以及磁盘上undo_log中的数据

图示如下：

![image-20211114001946823](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211114001946823.png)

可以看到整个事务提交的过程是先写日志，再落盘写数据。

其实redo log buffer将数据刷新到redo_log文件中的策略除开自己手动执行COMMIT外还有另一种情况。

在多任务时，其他线程COMMIT操作也可能会导致整个redo log buffer的刷新，刷新的redo_log文件中会对本线程提交的事务打上NOCOMMIT的标记。

其实这种现象取决于data buffer pool中存储的数据量占据data buffer pool总量的多少来决定，一般来说如果占据到70%左右就会触发该现象，我们可以对其进行手动设置，但一般来说保持默认值即可。

在CSR机制中我们会详细介绍这一个过程。



## ROLLBACK

现在我们基于上面BEGIN后发生的情况执行ROLLBACK操作，内部会做以下的2件事：

6. 执行ROLLBACK操作，LOGBWR线程会将undo log buffer中的数据重写回到data buffer pool中，并且会把内存脏页数据恢复到最开始的值，然后对LSN号进行回滚更正
7. 清空内存undo log buffer、redo log buffer以及磁盘上undo_log中的数据

图示如下：

![image-20211113235225099](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211113235225099.png)

# CSR机制

## 日志作用

到目前为止，我们还没有发现undo_log和redo_log的任何作用，这是因为这2个日志只有在mysqld.service服务出现故障时才会应用到。

## 单纯前滚

如果在COMMIT操作过程前发生了宕机，此时内存中的数据会全部丢失，并且redo_log文件还没有来得及为本次操作打上COMMIT的标记，而仅仅记录了数据变化日志与TXID号以及LSN号情况下重启mysqld.service服务时将会产生如下情况：

1. 重启mysqld.service服务，发现redo_log中记录的LSN号和userInfo.ibd文件中记录的LSN号不一致，将触发CSR自动故障恢复机制的第一个阶段，前滚操作开始
2. 通过redo_log文件中的变更记录日志，在内存数据页中恢复更改的数据
3. 发现redo_log文件中的事务标记是COMMIT，CKPT将内存中数据页更新到磁盘，同时更新userInfo.ibd文件中的LSN号，让其追平redo_log文件中记录的LSN号
4. 前滚工作完成，mysqld.service服务正常启动

图示如下：

![image-20211114002921900](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211114002921900.png)



## 前滚后滚

如果在COMMIT操作过程前发生了宕机，此时内存中的数据会全部丢失，但是恰好宕机前一秒由于其他线程的COMMIT操作导致了整个redo log buffer的自动刷新，此时redo_log文件中已经写入了刚刚本线程操作事务的TXID号以及LSN号且标记此次操作是NOCOMMIT状态的情况下重启mysqld.service服务时将会产生如下情况：

1. 重启mysqld.service服务，发现redo_log中记录的LSN号和userInfo.ibd文件中记录的LSN号不一致，将触发CSR自动故障恢复机制的第一个阶段，前滚操作开始
2. 通过redo_log文件中的变更记录日志，在内存数据页中恢复更改的数据
3. 发现redo_log文件中的事务标记是NOCOMMIT，将触发CSR自动故障恢复的第二个阶段，回滚操作开始
4. 通过undo log文件中的信息记录，在内存数据页中对前滚数据进行更改
5. 使用LOGBWR线程，将更新的数据页变化信息与TXID以及LSN号记录到redo log buffer中
6. 此时LOGBWR线程会先将redo log buffer中记录的信息写入到redo_log文件中，在日志信息完全写入log file即ib_logfile文件后，会对该日志打上COMMIT标记
7. 触发CKPT，将内存数据页更新到磁盘文件userInfo.ibd中
8. 回滚工作完成，redo_log中记录的LSN号与userInfo.ibd中记录的LSN号一致，mysqld.service服务正常启动

图示如下：

![image-20211114003917670](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211114003917670.png)

# 相关配置

## INNODB_FLUSH_METHOD

配置参数据INNODB_FLUSH_METHOD，该参数控制的是undo/redo log buffer 和data buffer pool将数据刷写磁盘的时候是否经过os buffer。

- fsync：日志和数据缓冲区向磁盘写入数据时，必须先将数据写入os buffer后再刷新至磁盘中，此为默认设置
- O_DIRECT：数据缓冲区向磁盘写入数据时，不必将数据先写入os buffer中，而是直接写入到磁盘
- O_DSYNC：日志缓冲区向磁盘写入数据时，不必将数据先写入os buffer中，而是直接写入到磁盘



## INNODB_FLUSH_LOG_AT_TRX_COMMIT

配置参数INNODB_FLUSH_LOG_AT_TRX_COMMIT，定义COMMIT时，日志文件从缓冲区到磁盘的刷新方式。

- 设置为1时：日志在每次事务提交时必须先写入os buffer后再刷新至磁盘中。
- 设置为0时：每秒写入一次日志到os buffer并将其刷新到磁盘。未刷新日志的事务可能会在崩溃中丢失。
- 设置为2时：在每次事务提交后写入日志到os buffer中，并每秒刷新一次到磁盘。未刷新日志的事务可能会在崩溃中丢失。



## 推荐设置

推荐将配置项写入至配置文件中，达到永久生效的目的。

最高安全模式：

```
innodb_flush_log_at_trx_commit=1
innodb_flush_method=O_DIRECT
```

最高性能模式:

```
innodb_flush_log_at_trx_commit=0
innodb_flush_method=fsync
```



# 相关阅读

参考文章：

- [详细分析MySQL事务日志(redo log和undo log)](https://www.cnblogs.com/f-ck-need-u/p/9010872.html)
- [InnoDB和事务流程、Crash Recovery、ACID](http://m.mamicode.com/info-detail-3100316.html)
- [InnoDB关键特性之double write](https://www.cnblogs.com/geaozhang/p/7241744.html)