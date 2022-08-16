# 物理存储结构

## 文件释义

在MySQL5.7版本的InnoDB引擎中，以下5个文件至关重要。

1）数据存放根目录下有3个文件

- ibdata文件：共享表空间文件，存放的信息很多，诸如表的元信息、UNDO回滚日志（8.0版本以下）等都存放在这里面
- ib_logfile文件：存放REDO重做日志、以及事务相关日志
- ibtmp文件：临时表的存放位置，诸如JOIN UNION等操作产生的临时数据都会存放在这里面，用完就删除

如下所示：

```
$ ls /db/mysql57/3306/data/
ibdata1
ibtmp1
ib_logfile1
```

2）每一张表有2个文件：

- .frm文件：存放表字段信息
- .ibd文件：存放表记录以及索引信息

如下所示：

```
$ ls /db/mysql57/3306/data/db1/ | grep "user"

userInfo.ibd
userInfo.frm
```

3）其他类型的一些文件：

- .TRG：文件将触发器映射到相应的表
- .TRN：文件包含触发器定义

# 逻辑存储结构

## 表空间

MySQL5.5版本之后的InnoDB存储引擎开始使用表空间存储数据，即文件ibdata。

它最初规定将所有的数据都存放在此文件中，这样的存储方式被称为共享表空间存储，但是这样的管理方式比较混乱。

所以经过版本变迁，共享表空间文件ibdata中存储的数据也越来越少，如下所示：

- 5.5版本出现表空间管理模式、此时为共享表空间管理模式，所有数据均存放至此、包括数据行、索引、临时表、统计信息等
- 5.6版本以后，共享表空间只用来存储元信息、UNDO回滚日志、临时表，至此变更为独立表空间管理模式
- 5.7版本以后，表空间中的临时表被独立出来
- 8.0版本以后，表空间中的UNDO回滚日志被独立出来

[MySQL5.6表空间存储数据一览](https://dev.mysql.com/doc/refman/5.6/en/innodb-architecture.html)

[MySQL5.7表空间存储数据一览](https://dev.mysql.com/doc/refman/5.7/en/innodb-architecture.html)

[MySQL8.0表空间存储数据一览](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)

## 段区页的概念

表空间是由段区页组成，基本概念如下：

![](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/SouthEast.jpeg)

图的解释：

```
段(tablespace):
    一个段就是一张表，构成如下：
        leaf node segment			 -- 数据段、B+树的叶子节点
        non leaf node segment        -- 索引段、B+树的非索引节点
        rollback segment			 -- 回滚段

区(extent):
    一个段由多个区构成，一个区由64个连续的页构成，即16*64=1024kb
    尽管页的大小可以改变，但是不论页的大小怎么变化，区的大小总是为1M

页(page):
    一个页也被称为一个block，一个页的大小默认为16kb
    InnoDB 1.0.x开始引入压缩页，即每个页的大小可以通过参数key_block_size设置为2kb、4kb、8kb。
    如果这样做，那么每个区对应页的数量就应该变为为512、256、128个，因为区总是要保证1M的大小
    InnoDB 1.2.x开始新增了参数innodb_page_size，通过该参数可以将默认页的大小设置为4kb、8kb。但是页中的数据并不是压缩的。
    如果这样做，那么每个区中页的数量同样也会变化为256或者128个，因为区总是要保证1M的大小

行(row):
    数据是按行进行存放的。每个页存放的记录也是硬性定义的，最多允许存放16KB/2-200行记录，即7992行记录
```

我们说过存储引擎类似于文件系统，文件系统的作用在于组织数据如何写入到磁盘与规定如何从磁盘上读取数据。

如果没有文件系统，产生1kb的数据时就立马写入磁盘，这无疑是愚蠢的设计，因为I/O次数过多，时间就会大幅度增加。

InnoDB存储引擎也是一样的，当用户进行操作时它并不会直接将用户的数据写入到磁盘，而是先将写入的数据按照16k的大小依次写入到page中，同理读取数据也是一样，page也是最小的读取单元。

# 相关设置项

## 共享表空间

查看默认的共享表空间文件ibdata1的大小：

```
> SHOW VARIABLES LIKE "innodb_data_file_path";
> SELECT @@innodb_data_file_path;  -- 也可使用这个命令
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:12M:autoextend |
+-----------------------+------------------------+
1 row in set (0.00 sec)
```

其中12M是最大阈值，autoextend的意思是到达12M后自动进行扩容，通过以下命令可查看到扩容到多少M。

```
> SHOW VARIABLES LIKE "%extend%";
> SELECT @@innodb_autoextend_increment; -- 也可使用这个命令
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| innodb_autoextend_increment | 64    |
+-----------------------------+-------+
1 row in set (0.00 sec)
```

默认的共享表空间只有1个ibdata1文件，我们可以为其设置多个，如下面这条语句的意思就会初始出2个表空间文件，且第2个表空间文件达到阈值后会自动进行扩容。

通常该配置会放在服务初始化之前应用：

```
# 初始化2个ibdata文件用做共享表空间
innodb_data_file_path=ibdata1:512M;ibdata2:512M:autoextend

# 当第2个文件达到512m后，自动为其扩容64M
innodb_autoextend_increment=64
```

## 独立表空间

在MySQL5.6版本之后，所有数据均采用独立表空间的存储方式。

存储特点如下：

- 一个表一个.ibd文件、存储数据行和索引信息
- 一个表一个.frm文件、存储表基本的字段、约束等信息
- ibtmp1中会存放一些临时表的信息
- ib_logfileN中存储REDO重做日志
- 共享表空间ibdataN中还会存放该表元信息，如统计信息等
- ibdataN中除了存储所有表的元信息外还会存放UNDO回滚日志（8.0版本之前）

结论：一张InnoDB表的所有数据 = .frm+.ibd+ibdataN中该表的元信息

查看独立表空间管理模式是否开启，1开启，0关闭：

```
> SELECT @@innodb_file_per_table;
+-------------------------+
| @@innodb_file_per_table |
+-------------------------+
|                       1 |
+-------------------------+
```

全局级别的关闭独立表空间管理模式，采用共享表空间管理模式，仅对当前mysqld.service服务运行时生效，重启后则失效（不推荐设置）：

```
> SET GLOBAL innodb_file_per_table = 0;
```

# ibd数据迁移

数据库A下有一张数据表t1，其中有1000万行的数据，现在我想将它迁徙到数据库B下，该怎么做？

1）拿到t1的建表语句，在数据库B下进行执行，此时在数据库B中的ibdata1会生成t1表的相关元数据信息、还会生成t1.frm表结构文件以及生成t1.ibd的记录与索引文件。

2）在数据库B中执行以下命令，将t1.ibd文件进行删除，切记不可手动rm删除该文件，因为共享表空间中还存在一些数据，手动rm是删不掉的：

```
> ALTER TABLE t1 DICARD TABLESPACE;
```

3）将数据库A下的t1.ibd拷贝到数据库B下，修改权限并在数据库B中执行以下命令：

```
> ALTER TABLE t1 IMPORT TABLESPACE;
```
