# Atlas功能

Atlas中间件是由奇虎360公司基于MySQL官方中间件Mysql-Proxy二次开发进行实现，它能够对数据库进行读写分离、分库分表配置，配合MHA架构进行高可用环境搭建有较好的效果。

GITHUB地址：[点我跳转](https://github.com/Qihoo360/Atlas)

在此文中，会着重介绍MHA配合Atlas进行读写分离的配置，关于分库分表我们有更好的选择，不在此处进行举例。

如果你的数据库数据量级不是特别大，MHA高可用+Atlas读写分离应该能够应付绝大多数场景，并且它的配置比MHA+MyCat简单许多。

我们先来观察单纯的MHA架构有什么缺点：

- MHA是单活架构，最低1主2从构成，读写都在主库上进行，从库只有在主库宕机后才会发生作用，硬件利用率较小
- 单纯的使用MHA架构会使主库的压力比较大，承载高频率读写操作的耗时时间很长

所以我们需要使用Atlas中间件，来进行基于MHA高可用的读写分离构建，使硬件利用率提升，主库压力减轻。

以下是架构草图：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021429377.png" alt="image-20211202142925247" style="zoom:100%;" />



# Atlas搭建

## 地址介绍

我们基于上一章节MHA架构中的地址规划继续进行使用：

```
Master:        192.168.0.120   -- 将Atlas中间件安在这台服务器上
Slave1:        192.168.0.120
Slave1:        192.168.0.130
Manager:       192.168.0.200
binlog_server: 192.168.0.210
```

为了节省虚拟机资源，所以可以将Atlas安装到MHA的Manager服务器上。







## 软件安装

在软件安装前，需要注意以下3点：

- Atlas只能运行在64位系统上
- Centos5.x安装e15版本，Centos6.x及以上安装e16版本
- MySQL版本应大于5.1，官方推荐使用5.6

去Github上下载软件：

```
-- 1.官方Github上下载
https://github.com/Qihoo360/Atlas/wiki/Atlas%E7%9A%84%E5%AE%89%E8%A3%85

-- 2.或者直接使用下面的命令，我将安装包上传到了腾讯云COS，不用担心版本，官方一年没更了
$ wget https://code-1302522496.cos.ap-nanjing.myqcloud.com/Atlas-2.2.1.el6.x86_64.rpm

$ rpm -ivh ./Atlas-2.2.1.el6.x86_64.rpm 
```

另外我们这台服务器还没有安装MySQL，所以需要手动安装一下，参考之前的MHA架构一章。

注意Atlas服务器安装完MySQL后应当停止mysqld.service服务，因为我们只需要使用到MySQL的client端。

```
$ systemctl stop mysqld.service
$ systemctl disable mysqld.service
```





## 相关配置

在安装成功后，进入以下目录：

```
$ cd /usr/local/mysql-proxy/conf
```

这个目录中默认有个test.cnf，里面详细记载了Atlas的配置，先不用管它，直接重命名备份：

```
$ mv ./test.cnf ./test.cnf.bak
```

接下来我们要自己写一个配置文件，但在此之前要将Master节点上配置的MHA管理用户以及主从复制用户使用Atlas中提供的功能进行加密并记录加密后得到的值，因为在Atlas配置项中，用户的密码需要密文填入，总体来说这里可以将所有Master上允许远程登录的业务用户都操作一次：

```
$ /usr/local/mysql-proxy/bin/encrypt mha  -- 我们mha管理用户的密码是mha，所以这里直接填写密码
O2jBXONX098=

$ /usr/local/mysql-proxy/bin/encrypt 123  -- 我们主从复制用户的密码是123，所以这里直接填写密码
3yb5jEku5h4=
```

开始书写Atlas配置文件了：

```
$ vim ./test.cnf

[mysql-proxy]
# Atlasd的工作IP和端口
proxy-address = 0.0.0.0:33060
# Atlasd的管理IP和端口
admin-address = 0.0.0.0:2345
# 默认字符集
charset=utf8

# 允许登录管理接口的用户名
admin-username = user
# 管理接口的密码
admin-password = pwd

# 主库的IP和端口号，可设置多个，使用逗号分隔
proxy-backend-addresses = 192.168.0.120:3306
# 从库的IP和端口号，可设置多个，使用逗号分隔
proxy-read-only-backend-addresses = 192.168.0.130:3306,192.168.0.140:3306
# 允许登录工作接口的用户名与其对应的加密后的密码
pwds = repl:3yb5jEku5h4=,mha:O2jBXONX098=

# 是否已守护线程运行
daemon = true
# 开启监控进程和工作进程
keepalive = true
# 工作线程数
event-threads = 8
# 日志记录级别
log-level = message
# 日志存放路径
log-path = /usr/local/mysql-proxy/log
# SQL日志开关
sql-log=ON
```

其他配置项可参照test.cnf.back文件进行查找。





## 相关命令

配置完成后我们就可以使用Atlas的命令了，启动命令如下：

```
$ /usr/local/mysql-proxy/bin/mysql-proxyd test start
```

查看Atlas是否正常运行：

```
$ netstat -lntup | grep mysql-proxy
$ /usr/local/mysql-proxy/bin/mysql-proxy test status
```

重启命令如下：

```
$ /usr/local/mysql-proxy/bin/mysql-proxyd test restart
```

停止命令如下：

```
$ /usr/local/mysql-proxy/bin/mysql-proxyd test stop
```





# Atlas测试

## 两种模式

Atlas有2种模式，一种是对外服务的33060接口，一种是对内服务的2345接口。

- 所有业务有关的操作，都在33060接口上进行，它将自动进行读写分离
- 所有监控有关的操作，都在2345接口上进行，它将自动使用Atlas监控整个服务

当我们添加了Atlas中间件后，后续所有请求都朝Atlas中间件发送而不是再朝真实的MySQL服务器发送。



## 工作接口

使用工作模式进入测试读写分离是否正常运行。

测试读操作，只要没有被BEGIN...COMMIT所包裹的语句，Atlas中间件都将视为读操作：

```
$ mysql -umha -pmha -h192.168.0.200 -P33060 -e " SELECT @@server_id;" 

+-------------+
| @@server_id |
+-------------+
|         140 |
+-------------+

-- 注意！以后所有application都将链接Atlas的33060端口进行业务操作
```

测试写操作，即使是一个查询语句，但被BEGIN...COMMIT包裹，Atlas中间件也会认为它是写操作：

```
$ mysql -umha -pmha -h192.168.0.200 -P33060 -e "BEGIN; SELECT @@server_id; ROLLBACK;"

+-------------+
| @@server_id |
+-------------+
|         120 |
+-------------+

-- 注意！以后所有application都将链接Atlas的33060端口进行业务操作
```





## 管理接口

使用管理接口查看服务相关信息。

```
$ mysql -uuser -ppwd -h192.168.0.200 -P2345

-- 注意！这里针对管理接口我们设置的用户名是user、密码是pwd，参见配置文件
```

打印帮助信息，来查看你目前能够在管理接口下使用的所有语句：

```
> SELECT * FROM help;

+----------------------------+---------------------------------------------------------+
| command                    | description                                             |
+----------------------------+---------------------------------------------------------+
| SELECT * FROM help         | shows this help                                         |
| SELECT * FROM backends     | lists the backends and their state                      |
| SET OFFLINE $backend_id    | offline backend server, $backend_id is backend_ndx's id |
| SET ONLINE $backend_id     | online backend server, ...                              |
| ADD MASTER $backend        | example: "add master 127.0.0.1:3306", ...               |
| ADD SLAVE $backend         | example: "add slave 127.0.0.1:3306", ...                |
| REMOVE BACKEND $backend_id | example: "remove backend 1", ...                        |
| SELECT * FROM clients      | lists the clients                                       |
| ADD CLIENT $client         | example: "add client 192.168.1.2", ...                  |
| REMOVE CLIENT $client      | example: "remove client 192.168.1.2", ...               |
| SELECT * FROM pwds         | lists the pwds                                          |
| ADD PWD $pwd               | example: "add pwd user:raw_password", ...               |
| ADD ENPWD $pwd             | example: "add enpwd user:encrypted_password", ...       |
| REMOVE PWD $pwd            | example: "remove pwd user", ...                         |
| SAVE CONFIG                | save the backends to config file                        |
| SELECT VERSION             | display the version of Atlas                            |
+----------------------------+---------------------------------------------------------+
```

如，查询后端的所有节点信息：

```
> SELECT * FROM backends;

+-------------+--------------------+-------+------+
| backend_ndx | address            | state | type |
+-------------+--------------------+-------+------+
|           1 | 192.168.0.120:3306 | up    | rw   |
|           2 | 192.168.0.130:3306 | up    | ro   |
|           3 | 192.168.0.140:3306 | up    | ro   |
+-------------+--------------------+-------+------+
```

对于节点你可以自由的进行规划，如下所示：

```
-- 动态删除某一个节点
> REMOVE BACKEND 3;

-- 动态添加某一个节点
> ADD SLAVE 192.168.0.140:3306;
```

不需要重启而而想保存在管理接口中应用的配置，你需要输入以下语句：

```
> SAVE CONFIG;
```





## 添加用户

使用Atlas中间件后，如果要添加一个允许远程登录的用户，我们可以向下面这样进行操作。

1）在主库中创建用户：

```
$ mysql -uroot -p -e"GRANT SELECT, UPDATE, INSERT ON *.* TO ken@'%' IDENTIFIED BY '123456';"
```

2）登录atlas管理端口，添加用户：

```
$ mysql -uuser -ppwd -h192.168.0.200 -P2345

> ADD pwd ken:123456;

> SELECT * FROM pwds;
+----------+--------------+
| username | password     |
+----------+--------------+
| repl     | 3yb5jEku5h4= |
| mha      | O2jBXONX098= |
| ken      | /iZxz+0GRoA= |
+----------+--------------+

> SAVE CONFIG;
```



# 宕机处理

当MHA中主库宕机故障转移产生了新库时，我们需要先执行MHA的修复流程。

MHA修复完毕后，需要在Atlas中执行以下操作：

```
-- 1.登录管理接口
$ mysql -uuser -ppwd -h192.168.0.200 -P2345

-- 2.找到新主库索引编号
> SELECT * FROM backends;

-- 3.删除旧主库
>  REMOVE BACKEND stop_master_index;

-- 4.添加新主库
> ADD MASTER 192.168.0.XXX:3306;

-- 5.将旧主库设置为从库
> ADD SLAVE 192.168.0.XXX:3306;
```

