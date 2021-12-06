# 功能概述

xtrabackup简称XBK，是一款第三方的物理备份与恢复工具。

相比于mysqldump来说，xtrabackup的备份与管理更加的简单，并且支持真正意义上的热备份以及增量备份，它的备份方式类似于直接对物理文件进行拷贝操作，运维人员不需要管理逻辑结构，所以备份与恢复的性能较高。

xtrabackup并不是完美无缺的，首先它需要第三方的下载与安装，其次是它不能够对备份文件进行压缩操作，所以会占用更多的磁盘空间，此外xtrabackup由于是物理备份，故备份文件的可读性也比较差。



# 软件安装

下面我们在Centos服务器上安装该软件：

```
-- 1.下载依赖包并安装
$ yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL libev

-- 2.下载软件并安装，这里选择7版本即可
$ wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.12/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.12-1.el7.x86_64.rpm
$ yum install -y ./percona-xtrabackup-24-2.4.12-1.el7.x86_64.rpm
```

xtrabackup有2个版本，分别是innobackupex与xtrabackup，innobackup更加强大，在执行上述命令时会一并对其进行安装，所以我们接下来只需要使用以下命令查看innobackupex是否安装成功即可：

```
$ innobackupex --help
```



# 特点解析

## 备份特点

xtrabackup的备份是基于物理结构的备份，他会先去备份出ibdata文件、tablename.frm文件、tablename.ibd文件以及undo_log日志文件。

由于MySQL5.7中的redo_log信息存放在共享表空间ibdata文件里，所以在MySQL5.7环境下使用xtrabackup备份时并不会像MySQL8.0中一样还要去备份redo_log文件。

在备份过程中，有以下几点注意事项：

- 对于非InnoDB的表来说，会进行短暂锁表的操作，在锁表期间拷贝上述物理文件
- 对于InnoDB的表来说，会立即触发CKPT，将内存数据页刷写至磁盘中，并且会记录一个LSN号，然后在拷贝上述物理文件
- 对于InnoDB的表来说，在备份期间产生的新数据也会一并进行保存，实际上就是对redo_log文件实时的进行刷新，并记录redo_log文件中最新的LSN号码，这也就是常说的热备份



## 恢复特点

xtrabackup在恢复数据时，会模拟CSR的两次滚动过程（先前滚、再回滚）以便追平LSN号，具体CSR滚动过程的详情请参见之前关于CSR的文章。



# 全备与恢复

## 备份相关

全备的基础命令格式如下：

```
$ innobackupex --user=root --password=密码 --no-timestamp --socket=/tmp/mysql.sock /tmp/full

-- --no-timestamp：该选项可以表示不要创建一个时间戳目录来存储备份，指定到自己想要的备份文件夹
```

全备完成后，将产生以下一些重要文件：

```
xtrabackup_binlog_info
xtrabackup_checkpoints
xtrabackup_info
xtrabackup_logfile
```

其中有2个文件尤为重要：

```
1. xtrabackup_binlog_info
-- 使用cat命令查看该文件后，会得到一些信息，如下所示：
mysql_bin.000001	1806	4e4c5a8d-4e05-11ec-a21b-000c290fdc76:1-6
-- 它的意思是目前所备份截止的数据是在mysql_bin.000001中的1806事件处，也是GTID6的事件处
-- 在数据恢复时，我们恢复了这个全备文件，还有一些全备周期后的数据需要从binlog中进行截取恢复，那么就打开binlog文件mysql_bin.000001，截取GTID6事件及其后面的事件进行恢复即可。

2. xtrabackup_checkpoints
-- 使用cat命令查看该文件后，会得到一些信息，如下所示：
backup_type = full-backuped    -- 代表这是全量备份
from_lsn = 0                   -- 上次所到达的LSN号(对于全备就是从0开始，对于增量有别的显示方法)
to_lsn = 4409199               -- 备份开始时间(CKPT)点数据页的LSN 
last_lsn = 4409208             -- 备份结束后，redo日志最终的LSN
compact = 0
recover_binlog_info = 0
-- 注意！在MySQL5.7的版本中，进行全量备份时 last_lsn减9 = to_lsn 时是正确的，这是因为MySQL5.7中InnoDB存储引擎内部会占用9个LSN号
-- 所以每次全备完成之后，应当检查这两个号码是否一致
-- 而对于增量备份来说，应当检查增量备份文件中的from_lsn与全备文件中的to_lsn的号码是否一致
--（1）备份时刻，立即将已经commit过的，内存中的数据页刷新到磁盘(CKPT)并开始备份数据，数据文件的LSN会停留在to_lsn位置。
--（2）备份时刻有可能会有其他的数据写入，已备走的数据文件就不会再发生变化了。
--（3）在备份过程中，备份软件会一直监控着redo与undo，如果一旦有变化会将日志也一并备走，并记录LSN到last_lsn。
-- 从to_lsn到last_lsn就是指备份过程中产生的数据变化，在MySQL5.6版本中这两个号码如果一致则代表进行备份过程时没有任何新的数据变化
-- 而在MySQL5.7版本中last_lsn-9应该等于to_lsn，这也代表进行备份过程时没有任何新的数据变化
```



## 恢复相关

在恢复全备之前，要先对文件进行整理，进行CSR日志滚动，追平LSN号：

```
$ innobackupex --apply-log /tmp/full

-- --apply-log参数：一般情况下，在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务，因此，此时数据文件仍处于不一致状态。
-- 而--apply-log的作用是通过回滚未提交的事务及同步已经提交的事务至数据文件使数据文件与事务处于一致性状态
-- 也就是说该参数会进行CSR两次滚动机制，用于追平LSN号
```

然后直接将文件进行拷贝即可，别忘了修改权限：

```
$ \cp -rf /tmp/full/* /db/mysql57/3306/data/
$ chown -R mysql:mysql /db/mysql57/3306/data/
```



## 故障演练

首先我们线上库中有一些数据：

```
> SELECT * FROM db1.userInfo;
+----+-------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | Jack  |  18 |
|  2 | Mary  |  18 |
|  3 | Tom   |  18 |
|  4 | Anni  |  21 |
|  5 | Booby |  19 |
+----+-------+-----+
```

所以直接对其进行全备即可：

```
$ innobackupex --user=root --password=123 --no-timestamp --socket=/tmp/mysql.sock /tmp/full
```

备份完成后，检查xtrabackup_binlog_info文件，获取当前全备截止的GTID号：

```
$ cat /tmp/full/xtrabackup_binlog_info
mysql_bin.000001	1806	4e4c5a8d-4e05-11ec-a21b-000c290fdc76:1-6
```

然后继续模拟新的数据：

```
$ mysql -uroot -p
> USE db1;
> BEGIN;
>     INSERT INTO userInfo(name, age) VALUES ("Jason", 21),("Jarry", 19);
> COMMIT;
```

模拟误操作，由于权限管理不当，公司的实习生删除了整个MySQL的数据目录：

```
$ rm -rf /db/mysql57/3306/data/
```

故障发生后，我们要了解故障是怎么发生的，有没有逻辑误操作什么的，这里没有逻辑误操作，只有物理层面的误操作，所以恢复起来会很方便，不用分析binlog日志，直接找到最后一个GTID号即可，但是目前不知道用的是那个binlog，所以就直接查看最后的一个binlog：

```
$ mysqlbinlog --no-defaults --base64-output=decode-rows -v /db/mysql57/3306/logs/mysql_bin.000001 | grep SET

SET @@SESSION.GTID_NEXT= '4e4c5a8d-4e05-11ec-a21b-000c290fdc76:7'/*!*/;
```

下面是恢复思路，首先xtrabackup备份到了GTID6的位置，而binlog记录到了GTID7的位置，所以我们要使用全备文件恢复到GTID6的位置，再截取binlog恢复到GTID7的位置。

截取binlog日志，只要GTID7的记录：

```
$ mysqlbinlog --no-defaults --skip-gtids --include-gtids='4e4c5a8d-4e05-11ec-a21b-000c290fdc76:7' /db/mysql57/3306/logs/mysql_bin.000001 > /tmp/bin.sql
```

还是按照管理，先到测试库中进行恢复演练，确保测试库是干净的，且配置文件和线上库是一致的：

```
-- 测试库下载生产库上的全备文件以及二进制截取文件
$ scp -r root@192.168.0.120:/tmp/full /tmp
$ scp -r root@192.168.0.120:/tmp/bin.sql /tmp
```

然后测试库先恢复xtrabackup的全备文件：

```
$ innobackupex --apply-log /tmp/full
$ \cp -rf /tmp/full/* /db/mysql57/3306/data/
$ chown -R mysql:mysql /db/mysql57/3306/data/
```

再恢复截取的二进制日志文件：

```
$ systemctl restart mysqld.service
$ mysql -uroot -p
> SET SQL_LOG_BIN = 0;
> SOURCE /tmp/bin.sql;
> SET SQL_LOG_BIN = 1;
```

查看测试库数据是否恢复成功：

```
> SELECT * FROM db1.userInfo;
+----+-------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | Jack  |  18 |
|  2 | Mary  |  18 |
|  3 | Tom   |  18 |
|  4 | Anni  |  21 |
|  5 | Booby |  19 |
|  6 | Jason |  21 |
|  7 | Jarry |  19 |
+----+-------+-----+
```

至此数据恢复成功，将测试库的数据使用xtrabackup进行全备后发送到线上库进行恢复：

```
-- 测试库
$ rm -rf /tmp/full
$ innobackupex --user=root --password=123 --no-timestamp --socket=/tmp/mysql.sock /tmp/full

-- 线上库
$ mv /tmp/full /tmp/old_full
$ scp -r root@192.168.0.130:/tmp/full /tmp
$ innobackupex --apply-log /tmp/full
$ mkdir /db/mysql57/3306/data
$ \cp -rf /tmp/full/* /db/mysql57/3306/data/
$ chown -R mysql:mysql /db/mysql57/3306/data/
$ systemctl restart mysqld.service
$ mysql -uroot -p
$ SELECT * FROM db1.userInfo;
+----+-------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | Jack  |  18 |
|  2 | Mary  |  18 |
|  3 | Tom   |  18 |
|  4 | Anni  |  21 |
|  5 | Booby |  19 |
|  6 | Jason |  21 |
|  7 | Jarry |  19 |
+----+-------+-----+
```



# 增量与恢复

## 备份相关

增量备份必须依赖于全量备份。

每个增量备份在备份后都会生成一个新的文件夹，并且具有全备中所有的相关文件。

增量备份命令如下：

```
$ innobackupex --user=root --password=密码 --no-timestamp --incremental --incremental-basedir=/tmp/full --socket=/tmp/mysql.sock /tmp/incr1

-- --incremental ：添加该参数以表明此次是增量备份
-- --incremental-basedir：此次增量备份所依赖的备份文件
```

在备份完成后，我们需要检查本次增量备份后的from_lsn是否等于被基于备份文件的to_lsn：

```
$ cat /tmp/full/xtrabackup_checkpoints 
backup_type = full-prepared
from_lsn = 0
to_lsn = 4412147                 -- 这里
last_lsn = 4412156
compact = 0
recover_binlog_info = 0

$ cat /tmp/incr1/xtrabackup_checkpoints 
backup_type = incremental        -- 代表这是一个增量备份
from_lsn = 4412147               -- 这里
to_lsn = 4412738
last_lsn = 4412747
compact = 0
recover_binlog_info = 0
```

上面已经增量备份了incr1，还需要增量备份incr2，就需要将incr2的增量备份基于incr1来做：

```
$ innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/tmp/incr1 --socket=/tmp/mysql.sock /tmp/incr2
```



## 恢复相关

进行增量备份恢复时，全备文件必须先进行整理，并且必须添加参数--redo-only：

```
$ innobackupex --apply-log --redo-only /tmp/full

-- --redo-only代表在进行整理文件模拟CSR过程的过程中，只加载redo_log文件中的信息
```

其次，需要将增量备份的文件合并到全量备份文件中并进行整理，先来整理incr1，只要不是最后一个合并的增量备份文件，就需要加参数--redo-only：

```
$ innobackupex --apply-log --redo-only --incremental-dir=/tmp/inc1/ tmp/full 
```

接着我们需要对incr2也进行合并，值得注意的是由于本次是最后一个增量备份的合并操作，所以不必加参数--redo-only：

```
$ innobackupex --apply-log --incremental-dir=/tmp/incr2/ tmp/full 
```

然后要对合并后的全量备份文件再次进行整理，此时不添加参数--redo-only：

```
$ innobackupex --apply-log /tmp/full
```



## 故障演练

每周日进行全量备份，其他都进行增量备份。

异常是在周三上午发生的，全盘死机了，服务完全崩溃。

现在需要恢复周日全备+周一增量+周二增量，以及根据周二增量记录的备份位置截取到增量备份完成后至周三异常发生前的二进制日志记录。

模拟开始，清除全部数据：

```
> DROP DATABASE db1;
> RESET MASTER;  -- 主从环境禁用该操作，从库必崩
> EXIT;

$ rm -rf /tmp/{bin.sql,full,old_full,incr1,incr2}
```

模拟将要被全备份的记录信息：

```
CREATE DATABASE full CHARSET utf8mb4;
USE full;
BEGIN;
    CREATE TABLE full_table(id INT);
    INSERT INTO full_table VALUES(1),(2),(3);
COMMIT;
```

进行周日的全备份：

```
$ innobackupex --user=root --password=123 --no-timestamp  --socket=/tmp/mysql.sock /tmp/full
```

检测/tmp/full/xtrabackup_checkpoints文件中last_lsn减9是否等于to_lsn：

```
$ cat /tmp/full/xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 4429630
last_lsn = 4429639
compact = 0
recover_binlog_info = 0
```

模拟周一新增的数据变化：

```
CREATE DATABASE incr1 CHARSET utf8mb4;
USE incr1;
BEGIN;
    CREATE TABLE incr1_table(id INT);
    INSERT INTO incr1_table VALUES(1),(2),(3);
COMMIT;
```

进行周一的增量备份：

```
$ innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/tmp/full --socket=/tmp/mysql.sock /tmp/incr1
```

检测/tmp/incr1/xtrabackup_checkpoints文件中from_lsn是否等于/tmp/full/xtrabackup_checkpoints文件中last_lsn的to_lsn：

```
$ cat /tmp/full/xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 4429630                        -- 这里
last_lsn = 4429639
compact = 0
recover_binlog_info = 0

$ cat /tmp/incr1/xtrabackup_checkpoints 
backup_type = incremental
from_lsn = 4429630                      -- 这里
to_lsn = 4435783
last_lsn = 4435792
compact = 0
recover_binlog_info = 0
```

模拟周二的数据变化：

```
CREATE DATABASE incr2 CHARSET utf8mb4;
USE incr2;
BEGIN;
    CREATE TABLE incr2_table(id INT);
    INSERT INTO incr2_table VALUES(1),(2),(3);
COMMIT;
```

进行周二的增量备份：

```
$ innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/tmp/incr1 --socket=/tmp/mysql.sock /tmp/incr2
```

检测/tmp/incr2/xtrabackup_checkpoints文件中from_lsn是否等于/tmp/incr1/xtrabackup_checkpoints文件中last_lsn的to_lsn：

```
$ cat /tmp/incr1/xtrabackup_checkpoints 
backup_type = incremental
from_lsn = 4429630
to_lsn = 4435783                           -- 这里
last_lsn = 4435792
compact = 0
recover_binlog_info = 0

$ cat /tmp/incr2/xtrabackup_checkpoints 
backup_type = incremental
from_lsn = 4435783                         -- 这里
to_lsn = 4441836
last_lsn = 4441845
compact = 0
recover_binlog_info = 0
```

模拟周三的数据变化：

```
CREATE DATABASE incr3 CHARSET utf8mb4;
USE incr3;
BEGIN;
    CREATE TABLE incr3_table(id INT);
    INSERT INTO incr3_table VALUES(1),(2),(3);
COMMIT;
```

模拟异常发生：

```
$ rm -rf /db/mysql57/3306/data/
```

现在我们准备开始恢复，先将所有的增量合并到全备中：

```
-- 1.第一次整理全备文件，并且后面要合并，就加参数 --redo-only
$ innobackupex --apply-log --redo-only /tmp/full

-- 2.不是最后一次合并增量文件，就加参数 --redo-only
$ innobackupex --apply-log --redo-only --incremental-dir=/tmp/incr1 /tmp/full 

-- 3.是最后一次合并增量文件，不加参数 --redo-only
$ innobackupex --apply-log --incremental-dir=/tmp/incr2 /tmp/full

-- 4.最后一次整理全备文件，并且后面不需要合并，不加参数 --redo-only
$ innobackupex --apply-log  /tmp/full
```

然后记录最后一次的二进制日志记录，这里是记录到GTID9的位置：

```
$ cat /tmp/incr2/xtrabackup_binlog_info 
mysql_bin.000001	2020	7dd9670a-4e70-11ec-8147-000c290fdc76:1-9
```

查看当前二进制日志文件已经记录到的位置，由于还是物理错误，不是逻辑错误，所以不用分析binlog文件：

```
$ mysqlbinlog --no-defaults --base64-output=decode-rows -v /db/mysql57/3306/logs/mysql_bin.000001 | grep SET

SET @@SESSION.GTID_NEXT= '7dd9670a-4e70-11ec-8147-000c290fdc76:12'/*!*/;
```

截取binlog，拿到GTID10-12这3条记录：

```
$ mysqlbinlog --no-defaults --skip-gtids --include-gtids='7dd9670a-4e70-11ec-8147-000c290fdc76:10-12' /db/mysql57/3306/logs/mysql_bin.000001 > /tmp/bin.sql
```

还是按照管理，先到测试库中进行恢复演练，确保测试库是干净的，且配置文件和线上库是一致的：

```
-- 测试库下载生产库上经过合并整理的全备文件以及二进制截取文件
$ scp -r root@192.168.0.120:/tmp/full /tmp
$ scp -r root@192.168.0.120:/tmp/bin.sql /tmp
```

然后测试库先恢复xtrabackup的全备文件，上面已经经过了整理了，所以这里不再需要进行整理：

```
$ \cp -rf /tmp/full/* /db/mysql57/3306/data/
$ chown -R mysql:mysql /db/mysql57/3306/data/
```

再恢复截取的二进制日志文件：

```
$ systemctl restart mysqld.service
$ mysql -uroot -p
> SET SQL_LOG_BIN = 0;
> SOURCE /tmp/bin.sql;
> SET SQL_LOG_BIN = 1;
```

查看测试库数据是否恢复成功：

```
> SELECT * FROM full.full_table;
> SELECT * FROM incr1.incr1_table;
> SELECT * FROM incr2.incr2_table;
> SELECT * FROM incr3.incr3_table;
```

至此数据恢复成功，将测试库的数据使用xtrabackup进行全备后发送到线上库进行恢复：

```
-- 测试库
$ rm -rf /tmp/full
$ innobackupex --user=root --password=123 --no-timestamp --socket=/tmp/mysql.sock /tmp/full

-- 线上库
$ mv /tmp/full /tmp/old_full
$ scp -r root@192.168.0.130:/tmp/full /tmp
$ innobackupex --apply-log /tmp/full
$ mkdir /db/mysql57/3306/data
$ \cp -rf /tmp/full/* /db/mysql57/3306/data/
$ chown -R mysql:mysql /db/mysql57/3306/data/
$ systemctl restart mysqld.service
$ mysql -uroot -p
> SELECT * FROM full.full_table;
> SELECT * FROM incr1.incr1_table;
> SELECT * FROM incr2.incr2_table;
> SELECT * FROM incr3.incr3_table;
```



# 单表恢复

如果误删除的只有一张表，且大小只有10M，而整个全备文件有500G，该如何进行恢复？

这得配合之前InnoDB存储结构中提到的一种ibd文件拷贝的物理恢复策略，搭配进行使用：

必须拿到创建表的SQL语句，然后再进行恢复。

```
-- 1.拿到误删除表的建表语句并进行创建
略

-- 2.执行以下命令，删除自带的空的这个表名.ibd文件
> ALTER TABLE 表名 DICARD TABLESPACE;

-- 3.将全备文件中的表ibd文件进行拷贝
> cp /tmp/full/库名/表名.ibd /db/mysql57/3306/data/库名/

-- 4.进行权限的修改
> chown -R mysql:mysql /db/mysql57/3306/data/库名/表名.ibd

-- 5.导入拷贝过来的ibd文件
> ALTER TABLE t1 IMPORT TABLESPACE;
```

