# 功能概述

mysqldump简称MBP，是一款MySQL自带的逻辑备份与数据恢复工具，不需要额外的安装下载。

mysqldump备份出的文件内容均为SQL语句，因此可读性非常高，此外mysqldump对备份文件的压缩率也非常高，这能极大的节省磁盘空间。

尽管mysqldump十分优秀，但是它仍然有一些难以掩饰的缺点。

比如，mysqldump十分依赖存储引擎，它的备份过程是先从磁盘中把数据读取出来然后依赖存储引擎将其转变为SQL语句，比较消耗资源且对于数据量级很大的备份操作来说效率较低。

并且mysqldump并不支持真正意义上的热备份，而是只支持快照备份，因此对数据变更频率较高的库进行备份时，mysqldump可能会存在部分数据丢失的情况，需要配合binlog才能完全恢复数据。

此外，mysqldump并不支持增量备份，仅支持全备份。



# 备份相关

## 基础命令

以下两种命令格式，可用于链接本地或远程的MySQL服务器，是备份的基础命令。

```
-- 本地备份的基础链接命令
$ mysqldump -uroot -p -S /tmp/mysql.sock

-- 远程备份的基础链接命令
$ mysqldump -uroot -p -h地址 -P端口
```



## 全库全表

参数-A用于进行所有库下所有表的全备操作，如下操作演示：

```
$ mysqldump -uroot -p -A > /tmp/full.sql
```

如果出现以下警告信息，一定要谨慎：

```
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events. 

警告：默认情况下，来自具有 GTID 的服务器的部分转储将包括所有事务的 GTID，即使是那些更改了数据库中被抑制部分的事务。 如果您不想恢复 GTID，请传递 --set-gtid-purged=OFF。 要进行完整的转储，请传递 --all-databases --triggers --routines --events。
```

它会然你加一些参数，此时一定要注意--set-gtid-purged=OFF这个参数，对于普通备份来讲可以加上该参数，如果是构建主从环境时则千万不要加，因为它会在备份的SQL文件中额外的加入一些信息，有了这些信息主从就搭建不了。



## 指定某库

参数-B用于进行指定某些库下所有表的全备操作，如下操作演示：

```
$ mysqldump -uroot -p -B 库名1 库名2 > /tmp/full.sql
```

这种备份操作一般来说会比较少做，因为在生产环境中我们会将所有的MySQL内置库也一同会进行备份。





## 指定某表

备份单个表或者多个表时，可使用以下的示例进行操作：

```
-- 备份库1下的表1和表2
$ mysqldump -uroot -p 库名1 表名1 表名2 > /tmp/full.sql

-- 备份库1下的所有表，由于没有加参数-B，所以生成的SQL语句中没有use 库1的操作，需要手动进行use
$ mysqldump -uroot -p 库名1 > /tmp/full.sql
```



## 推荐参数

-R代表存储过程，-E代表事件，--triggers代表触发器，一般做备份操作的时候都把这三个参数加上即可。

```
$ mysqldump -uroot -p -A -R -E --triggers > /tmp/full.sql
```

--master-data=2这个参数配置项如果加入，则会以注释的形式，保存备份开始时间点的binlog的状态信息，是一个必加参数。

```
$ mysqldump -uroot -p -A -R -E --triggers --master-data=2 > /tmp/full.sql

# 功能1：
# 在备份时，会自动记录，二进制日志文件名和位置号，如下所示：
# CHANGE MASTER TO MASTER_LOG_FILE='mysql_bin.000002', MASTER_LOG_POS=154;
# 它告诉你的信息是，目前所备份截止的数据是在mysql_bin.000002中的154事件处
# 在数据恢复时，我们恢复了这个全备文件，还有一些全备周期后的数据需要从binlog中进行截取恢复，那么就打开binlog文件mysql_bin.000002
# 截取154号事件及其后面的事件进行恢复即可。

# 功能2：
# 自动对备份时的表进行锁定操作，如果配合--single-transaction参数，则只对非InnoDB引擎的表进行锁表
# 而--single-transaction参数是进行快照备份的前提参数，所谓快照备份是指在备份时新加入的数据并不会记录，这也是mysqldump的一个特点，不支持真正的热备份
# 所以需要像上面说的那样，mysqldump需要配合binlog来进行数据恢复，才能保持恢复后数据的完整性
```

--single-transaction也是一个必加参数，用于支持InnoDB引擎开启快照备份：

```
$ mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction > /tmp/full.sql
```

--max-allowed-packet是一个可选性参数，一般来说我们也会将它加上，在远程备份时，我们需要将远程Server端的数据备份到本地，这个时候就会涉及到数据包大小的问题，使用该参数指定本地可收到的远程Server端传输的最大数据量。

```
$ mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction --max-allowed-packet=512M > /tmp/full.sql
```

-F代表在备份开始时，刷新一个或多个二进制日志文件，究竟是几个取决于备份库的数量，不推荐添加该参数，可能会一次性刷出很多二进制日志文件。

--set-gtid-purged=auto参数是一个可有可无的选项参数，默认值是auto其实就是on开启的意思，当开启时，会在全备文件中生成一个GTID号，由于GTID号具有幂等性，在做主从配置时，从库导入主库数据时发现该GTID号存在就不会将数据进行导入，如果该参数被设置为off，则全备文件中就不会生成GTID号，从库导入主库数据时即使该数据已经被导入了一次由于没有GTID号进行限制便会进行二次导入，这样主从搭建就会出现问题。

所以说该参数直接忘记它就好，因为它默认就是开启的，这正是我们所需要的状态。

# 故障演练

## 情况介绍

我的远端服务器上有1个db1的数据库出现了问题崩掉了，我需要对它进行恢复。

备份策略：每周2晚上23：00开始使用mysqldump进行全备，包括但不限于db1的所有库表信息，包括系统库。

故障时间：在周3凌晨的3：00时，数据库发生异常。

当前时间：现在是周3上午8：00，在这期间肯定有新数据的变更记录。

二进制日志：使用了二进制日志，并且记录模式是GTID+ROW

解决思路：使用mysqldump将全备进行恢复，至于周2晚23：00至周3上午8：00中的数据，可使用binlog进行恢复，恢复思路是找到周3凌晨3：00误操作的事件不对其进行截取，截取所有正常操作的事件。



## 准备数据

登录远程服务器，进入线上数据库开始准备数据：

```
$ ssh 192.168.0.120
$ mysql -uroot -p
```

模拟周2晚23：00之前的数据：

```
CREATE DATABASE db1 CHARSET utf8mb4;

USE db1;

CREATE TABLE userInfo(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT "主键",
    name CHAR(64) NOT NULL COMMENT "姓名",
    age TINYINT NOT NULL COMMENT "年龄"
) ENGINE innodb CHARSET utf8mb4 COLLATE utf8mb4_general_ci;

BEGIN;
    INSERT INTO 
        userInfo(name, age)
    VALUES
        ("Jack", 18),
        ("Mary", 18),
        ("Tom", 18);
COMMIT;

EXIT;
```

模拟周2晚23：00时的全备份：

```
$ mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction --max-allowed-packet=512M >  /tmp/full.sql
```

模拟其他时间段的数据以及误操作：

```
-- 模拟23:00全备完成后至发生误操作之前新插入的数据
USE db1;

BEGIN;
    INSERT INTO
        userInfo(name, age)
    VALUES
        ("Anni", 21);
COMMIT;

-- 模拟3:00发生的误操作
BEGIN;
    DELETE FROM userInfo WHERE id > 1;
COMMIT;

-- 模拟3:00到8:00之后的数据
BEGIN;
    INSERT INTO
        userInfo(name, age)
    VALUES
        ("Booby", 19);
COMMIT;
```

最终剩余的数据如下：

```
SELECT * FROM db1.userInfo;

+----+-------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | Jack  |  18 |
|  5 | Booby |  19 |
+----+-------+-----+
```



## 数据截取

首先第一步，我们现在手里有全备文件，打开全备文件查看最后的全备时正在使用的二进制日志文件以及截止的事件点，全备文件中已经备份到了GTID3的地方：

```
SET @@GLOBAL.GTID_PURGED='4e4c5a8d-4e05-11ec-a21b-000c290fdc76:1-3';
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql_bin.000001', MASTER_LOG_POS=978;
```

然后是第二步，我们需要查看目前二进制日志文件已经记录到的文件编号与事件点，注意这里binlog已经记录到了GTID6的地方：

```
> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+------------------------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+------------------+----------+--------------+------------------+------------------------------------------+
| mysql_bin.000001 |     1806 |              |                  | 4e4c5a8d-4e05-11ec-a21b-000c290fdc76:1-6 |
+------------------+----------+--------------+------------------+------------------------------------------+
```

第三步，分析二进制日志文件、找出误操作的事件点，如果有多个二进制文件则逐个进行分析：

```
$ mysqlbinlog --no-defaults --base64-output=decode-rows -vvv /db/mysql57/3306/logs/mysql_bin.000001
```

GTID5号处为误操作，截取关键内容如下：

```
# 上面最近的一个GTID号是：
SET @@SESSION.GTID_NEXT= '4e4c5a8d-4e05-11ec-a21b-000c290fdc76:5'/*!*/;

# at 1436
#211126  7:29:34 server id 3306  end_log_pos 1506 CRC32 0x903fb9e8 	Delete_rows: table id 190 flags: STMT_END_F
### DELETE FROM db1.userInfo
### WHERE
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2='Mary' /* STRING(256) meta=60928 nullable=0 is_null=0 */
###   @3=18 /* TINYINT meta=0 nullable=0 is_null=0 */
### DELETE FROM db1.userInfo
### WHERE
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */
###   @2='Tom' /* STRING(256) meta=60928 nullable=0 is_null=0 */
###   @3=18 /* TINYINT meta=0 nullable=0 is_null=0 */
### DELETE FROM db1.userInfo
### WHERE
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */
###   @2='Anni' /* STRING(256) meta=60928 nullable=0 is_null=0 */
###   @3=21 /* TINYINT meta=0 nullable=0 is_null=0 */
# at 1506
```

也就是说，我们在截取线上库的二进制日志时，不截取GTID为5的即可。

现在对二进制日志文件中的数据进行截取：

```
-- 全备文件中截取到GTID3的地方
-- 所以我们接下来就要截取binlog文件里GTID4-6的地方
-- 但是不要5这个误操作
$ mysqlbinlog --no-defaults --skip-gtids --include-gtids='4e4c5a8d-4e05-11ec-a21b-000c290fdc76:4-6' --exclude-gtids='4e4c5a8d-4e05-11ec-a21b-000c290fdc76:5' /db/mysql57/3306/logs/mysql_bin.000001 > /tmp/bin.sql
```



## 恢复数据

现在，我们需要将线上服务器的全备文件以及截取到的binlog文件发送至本地，尝试进行恢复，如果恢复效果良好则再放到线上库中正式恢复。

第一步，下载生产库中的全备文件以及binlog文件生成的备份文件：

```
$ scp root@192.168.0.120:/tmp/full.sql /tmp
$ scp root@192.168.0.120:/tmp/bin.sql /tmp
```

第二步，在本地服务器上进行测试，尝试恢复，此时一定要确保本地服务器的数据库是干净且没有任何数据的：

```
$ mysql -uroot -p
> SET SQL_LOG_BIN = 0;
> SOURCE /tmp/full.sql
> SOURCE /tmp/bin.sql
> SET SQL_LOG_BIN = 1;
```

第三步，恢复完成后查看相关数据：

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

第四步，本地库的数据恢复成功，我们将本地库的所有数据进行导出，上传到远程服务器进行恢复。

```
-- 注意！rm -rf在生产库是禁止操作！
$ rm -rf /tmp/bin.sql
$ rm -rf /tmp/full.sql
$ mysqldump -uroot -p -A -R -E --triggers --master-data=2 --single-transaction --max-allowed-packet=512M > /tmp/full.sql
```

第五步，将本地库的全备份文件上传到线上库：

```
$ scp /tmp/full.sql root@192.168.0.120:/tmp/full.sql
```

第六步，线上库导入本地库上传的全备文件，进行数据恢复：

```
$ mysql -uroot -p
> SET SQL_LOG_BIN = 0;
> SOURCE /tmp/full.sql
> SET SQL_LOG_BIN = 1;
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

由于GTID号的幂等性约束，所以已存在的数据不会进行重复导入，至此数据恢复成功！



# 单表恢复

如果误删除的只有一张表，且大小只有10M，而整个全备文件有500G，该如何进行恢复？

我们知道mysqldump的备份文件是SQL语句，所以只需要查找出被误删除的表与相关记录，并且导入成单独的SQL文件即可。

```
-- 1.获得表结构
$ sed -e'/./{H;$!d;}' -e 'x;/CREATE TABLE `表名`/!d;q' /tmp/full.sql > /tmp/table.sql

-- 2.获得INSERT INTO 语句，用于数据的恢复
$ grep -i 'INSERT INTO `表名`'  /tmp/full.sql >> /tmp/table.sql &

-- 3.登录数据库，进入库中，导入createbale
> USE 库名;
> SOURCE /tmp/table.sql;
```



