# MHA介绍

## **功能概述**

MHA（Master High Availability）是由日本人yoshinorim开发的一款成熟且开源的MySQL高可用程序，它实现了MySQL主从环境下MASTER宕机后能够自动进行单次故障转移的功能，其本身由perl语言编写，安装方便使用简单。

GitHub：[点我跳转](https://github.com/yoshinorim/mha4mysql-manager)

使用MHA能够让我们更大程度的解放双手，用更少的指令完成更多的事，MHA主要能够做以下几件事：

- 自动的在MASTER宕机后选举新的SLAVE作为MASTER，保证服务不被中断
- 自动的在MASTER宕机后将所有未被选举为新MASTER的SLAVE重新指向新的MASTER并启动复制
- 自动的在MASTER宕机后向数据库管理人员发送报警邮件
- 自动的进行VIP漂移服务，确保服务运行不会暂停

MHA搭建条件最少是1主2从，且必须是独立的服务器，不能单机多实例进行搭建。



## **架构图示**

MHA架构图如下所示，理想服务器数量是5台，我们也可以使用3台进行搭建，这篇文章会使用最理想的情况即5台服务器进行搭建：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021430195.png" alt="image-20211202143027102" style="zoom:100%;" />



## **相关说明**

MHA实际上就是一个软件集合，它的软件分为2部分：

- Manager软件
- Node软件

上图中每一个色块都是一个Node，包括Manager本身也是一个Node。

Node软件必须安装在所有的MHA节点上，而Manager软件则只需安装在管理节点上。

不同的软件由不同的工具包组成，如下所示：

```
-- Master
masterha_manager            - 用于启动MHA
masterha_check_ssh          - 用于检查MHA的SSH配置情况
masterha_check_repl         - 用于检查MHA的主从复制情况
masterha_master_monitor     - 用于检查Master节点是否宕机
masterha_check_status       - 用于检查当前MHA的运行状态
masterha_master_switch      - 用于自动故障恢复
masterha_conf_host          - 用于添加或者删除Manager中配置的server信息

-- Node
save_binary_logs            - 保存并复制Master的binlog
apply_diff_relay_logs       - 识别差异的中继日志事件并将其差异的event事件应用于其他的Slave中
purge_relay_logs            - 自动清除中继日志，且不会阻塞SQL_T线程
```

对于MASTER、SLAVE1、SLAVE2的作用这里不再描述，主要描述一下VIP以及Manager和binlog_server的作用：

```
-- VIP
  对外服务的虚拟IP，当Master宕机之后故障转移过程中选举了新的Master时虚拟IP也会漂移到新Master上，确保服务不会中断

-- Manager
  用于管理所有的Node，它会自动的监听MySQL主从复制的状态，当主库发生宕机后会开启一系列的故障转移工作

-- binlog_server
  这是一个单独存放拷贝主库binlog的服务器，不会参与任何业务处理，并且不会与主库的binlog产生任何延迟，具体思路是当主库的事务准备提交前，会将binlog记录发送给该服务器，只有当二进制日志存储服务器将该条记录成功存储后，主库上这一事务方可被提交，由此可见，这种技术是在牺牲性能的前提下保证了数据的一致性，一般来说我们都会进行开启，在主机宕机且SSH不可被链接状态下，从库的数据恢复依然可以从二进制日志存储服务器中获取数据
```



## **工作流程**

以下是MHA的工作流程，Manager节点通过masterha_manager脚本启动MHA后会先进行检查工作：

- Manager节点通过masterha_check_ssh脚本检查各节点的互信配置
- Manager节点通过masterha_check_repl脚本检查主从之间的复制情况

检查工作均完成且确认无误后，Manager节点会进行监控工作：

- Manager节点通过masterha_master_monitor对主库不断的进行心跳检测，主库3次无响应后会认为其以宕机

当主库宕机后，会开始故障转移工作，首先会对SLAVE进行选主，有以下3种算法：

- 读取Manager配置文件，判断是否有强制选主的Slave
- 自动判断目前已有从库的日志量，将最接近主库日志量的从库选为新的主库
- 根据配置文件中先后顺序进行选主

当选主完成后，Manager节点会再次通过SSH链接已宕机主库，有以下2种情况发生：

- 已宕机主库的SSH能够链接，表明MySQL服务是由逻辑因素所导致的宕机，此时Manager会通过save_binary_logs脚本，计算各个从库（包括新主库）与已宕机主库之间binlog差异，将已宕机主库的binlog位置找出来并进行截取、分发到各个从库上进行数据对齐，确保数据一致性
- 已宕机主库的SSH不能链接，表明MySQL服务是由物理因素所导致的宕机，此时Manager会通过apply_diff_relay_logs脚本，计算各个从库relay-log的差异，将差异较大的从库与新主库进行relay-log对齐，确保数据一致性

当所有库的数据一致性被确保之后，所有从库都将会与新主库建立主从关系，同时旧的已宕机主库信息将会从Manager项目配置文件中移除，至此整个MHA软件服务结束，Manager不会再对Node进行管理，接下来需要管理员手动对已宕机主库进行排查恢复，并且手动搭建旧主库与新主库之间的主从关系然后重新启动整个MHA服务。



# 前期准备工作

## **地址规划**

MHA架构仅支持单数的Node（不包含Manager和binlog_server），所以我们可以按照最低标准进行地址规划，如下所示：

| 作用   | IP地址        | 服务端口 | 操作系统                 | MySQL版本 | 配置                  |
| ------ | ------------- | -------- | ------------------------ | --------- | --------------------- |
| Master | 192.168.0.120 | 3306     | Centos7.3 基础设施服务器 | 5.7.34    | 2核CPU 2G内存 20G硬盘 |
| SLAVE  | 192.168.0.130 | 3306     | Centos7.3 基础设施服务器 | 5.7.34    | 2核CPU 2G内存 20G硬盘 |
| SLAVE  | 192.168.0.140 | 3306     | Centos7.3 基础设施服务器 | 5.7.34    | 2核CPU 2G内存 20G硬盘 |

此外，Manager和binlog_server也需要2台服务器，地址规划如下所示：

| 作用          | IP地址        | 服务端口 | 操作系统                 | MySQL版本 | 配置                  |
| ------------- | ------------- | -------- | ------------------------ | --------- | --------------------- |
| Manager       | 192.168.0.200 | 3306     | Centos7.3 基础设施服务器 | 无        | 1核CPU 1G内存 10G硬盘 |
| binlog_server | 192.168.0.210 | 3306     | Centos7.3 基础设施服务器 | 5.7.34    | 1核CPU 1G内存 10G硬盘 |

如果你只有3台服务器，可以将Manager安装在Slave2上，不要binlog_server。



## **机器配置**

我们准备5台虚拟机：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021400733.png)

每一台机器都按照下面进行配置：

```
-- 1.修改hostname
$ hostnamectl set-hostname [server_role]

-- 2.修改ip地址和UUID
$ vim /etc/sysconfig/network-scripts/ifcfg-ens33
$ systemctl restart network
```

关闭防火墙这个不用多说了，必做的操作。



## **安装MySQL**

接下来是在MASTER、SLAVE1、SLAVE2、binlog_server上安装MySQL。

1）下载MySQL5.7.34：

```
$ cd ~

$ wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.34-linux-glibc2.12-x86_64.tar.gz
```

2）创建必要目录与解压MySQL：

```
$ mkdir -p /db/mysql57/3306/{data,logs}

$ tar -xvf ./mysql-5.7.34-linux-glibc2.12-x86_64.tar.gz -C /db/mysql57/3306/

$ mv /db/mysql57/3306/mysql-5.7.34-linux-glibc2.12-x86_64/ /db/mysql57/3306/mysql
```

3）添加环境变量：

```
$ vim /etc/profile

export PATH=/db/mysql57/3306/mysql/bin:$PATH

$ source /etc/profile
```

4）创建MySQL管理用户：

```
$ groupadd mysql

$ useradd -g mysql mysql

$ chown -R mysql:mysql /db/mysql57/
```

5）验证是否安装成功：

```
$ mysql -V
```

6）这一步很关键，填写配置文件：

```
$ vim /etc/my.cnf

[mysqld]
user=mysql
# 确保server_id都不相同
server_id=120
port=3306
basedir=/db/mysql57/3306/mysql
datadir=/db/mysql57/3306/data
# 如果加上这2句，在有binlog_server服务器的情况下，MHA高可用将失败，所以
# character-set-server=utf8mb4
# collation-server=utf8mb4_general_ci
default-storage-engine=INNODB
socket=/tmp/mysql.sock
innodb_data_file_path=ibdata1:512M;ibdata2:512M:autoextend
innodb_autoextend_increment=64
autocommit=1
innodb_flush_log_at_trx_commit=1
innodb_flush_method=O_DIRECT
transaction-isolation=Read-Committed
log_error=/db/mysql57/3306/logs/mysqld.log
slow_query_log_file=/db/mysql57/3306/logs/slow.log
slow_query_log=1
long_query_time=0.1
log_queries_not_using_indexes
# 确保以下5项都是打开的
log_bin=/db/mysql57/3306/logs/mysql_bin
binlog_format=row
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1

[mysql]
port=3306
# default-character-set=utf8mb4
socket=/tmp/mysql.sock
prompt="> "

[client]
port=3306
# default-character-set=utf8mb4
```

7）填写完成后进行初始化工作：

```
$ mysqld --initialize-insecure
```

8）开启sys服务：

```
$ cat >/etc/systemd/system/mysqld.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/db/mysql57/3306/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
EOF

$ systemctl start mysqld.service

$ systemctl enable mysqld.service
```

9）尝试登录，再次验证server_id是否都不相同：

```
$ mysql -uroot -e "SELECT @@server_id;"
```



## **主从搭建**

主从搭建还是使用GTID主从而不是event主从，搭建流程如下：

1）在MASTER上执行以下操作，创建复制用户：

```
$ mysql -uroot -S /tmp/mysql.sock -e "grant replication slave on *.* to repl@'%' identified by '123'";

-- 注意，如果是线上环境，请设置repl用户的允许登录地址
-- repl用户的权限是仅用于复制操作
```

2）在SLAVE1和SLAVE2上执行以下操作，这里我们不构建延时从库：

```
> CHANGE MASTER TO
     MASTER_HOST='192.168.0.120',
     MASTER_USER='repl',
     MASTER_PASSWORD='123',
     MASTER_PORT=3306,
     MASTER_CONNECT_RETRY=10,
     MASTER_AUTO_POSITION=1; 
> START SLAVE;
```

3）在SLAVE1和SLAVE2上执行以下操作，检查复制状态，确保IO_T和SQL_T都是Yes：

```
> SHOW SLAVE STATUS\G;

Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```



## **主从验证**

这里我们对主从进行一次验证，为了后续的VIP功能测试，所以创建一个用户。

在MASTER上执行：

```
> mysql -uroot
> CREATE USER "tom"@"%" IDENTIFIED BY "123";
> GRANT all ON *.* TO "tom"@"%";
> FLUSH PRIVILEGES;
```

在SLAVE1和SLAVE2上查看用户是否创建成功：

```
> SELECT user, host FROM mysql.user WHERE user = "tom" AND host = "%";
```

至此，GTID主从搭建已经全部完成了。



# MHA准备工作

## **互信配置**

接下来开始做SSH互信配置，保证每个Node彼此之间能够互相链接，且在链接时不会有任何弹出提示要求输入密码的信息出现。

所有服务器上都执行以下操作：

```
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa >/dev/null 2>&1

-- 输入yes和root密码
$ ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.120
$ ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.130
$ ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.140
$ ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.200
$ ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.210
```

互信配置完成后进行一次检查，所有服务器都执行，确保没有遗漏任何一台服务器：

```
$ ssh root@192.168.0.120 pwd; ssh root@192.168.0.130 pwd; ssh root@192.168.0.140 pwd; ssh root@192.168.0.200 pwd; ssh root@192.168.0.210 pwd
```

## **软链接配置**

MHA在启动后，不会加载环境变量profile，而是直接从/usr/bin下读取需要的应用，所以我们需要在MASTER、SLAVE1、SLAVE2、binlog_server上做2条软链接：

```
$ ln -s /db/mysql57/3306/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
$ ln -s /db/mysql57/3306/mysql/bin/mysql /usr/bin/mysql
```

## **创建用户**

在Master主库上创建MHA的管理用户，这个管理用户会自动的被复制到从库上：

```
$ mysql -uroot -e "GRANT ALL PRIVILEGES ON *.* TO mha@'192.168.0.%' IDENTIFIED BY 'mha'";
```

## **软件下载**

MHA相关的软件资源你可以在[Github](https://github.com/yoshinorim/mha4mysql-manager/wiki/Downloads)上获取，共有4个需要下载的资源：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021400807.png)

实际上我们只会用到3个资源：

- MHA Manager 0.56 rpm RHEL6：Manager的软件包
- MHA Node 0.56 rpm RHEL6：Node的软件包
- MHA Node 0.56 traball：Manager相关脚本，如虚拟IP、邮件提醒等功能脚本

这里MHA Node 0.56 traball中的脚本不能直接使用，要经过一些列配置，比较麻烦。

所以我将它们都统一打包到了腾讯云COS上，可以直接复制以下命令在服务器上进行操作，注意！每一台服务器都执行：

```
$ cd ~

-- Manager的软件包
$ wget https://code-1302522496.cos.ap-nanjing.myqcloud.com/MHA/mha4mysql-manager-0.56-0.el6.noarch.rpm

-- Node的软件包
$ wget https://code-1302522496.cos.ap-nanjing.myqcloud.com/MHA/mha4mysql-node-0.56-0.el6.noarch.rpm
```

下面这3条命令仅在Manager服务器上执行：

```
-- VIP自动切换脚本
$ wget https://code-1302522496.cos.ap-nanjing.myqcloud.com/MHA_SCRIPTS/master_ip_failover

-- VIP手动切换脚本
$ wget https://code-1302522496.cos.ap-nanjing.myqcloud.com/MHA_SCRIPTS/master_online_change

-- 邮件报警脚本
$ wget https://code-1302522496.cos.ap-nanjing.myqcloud.com/MHA_SCRIPTS/send_report
```

## **目录配置**

下面我们在Manager服务器上创建3个目录，用于存放mha相关的资源：

```
$ mkdir -p /mha/{logs,scripts,conf}
```

将VIP脚本与邮件报警脚本转存到/mha/scripts目录下：

```
$ mv ~/{master_ip_failover,master_online_change,send_report} /mha/scripts
```

# MHA基础搭建

## **软件安装**

注意顺序！因为Manager服务器也是一个MHA的Node，所以必须先安装Node软件包。

1）所有的服务器上安装Node软件包依赖：

```
$ yum install perl-DBD-MySQL -y
$ rpm -ivh ./mha4mysql-node-0.56-0.el6.noarch.rpm
```

2）在Manager服务器上安装Manager软件包：

```
-- 必须先进行换源，否则会出现下载不到依赖包的情况
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

-- 安装软件包
$ yum install -y perl-Config-Tiny epel-release perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes

$ rpm -ivh ./mha4mysql-manager-0.56-0.el6.noarch.rpm
```

## **项目配置**

我们上面已经创建了/mha/conf目录，接下来就需要在Manager服务器上创建mha的项目配置文件了。

一个mha可以管理多套主从，只需要创建不同的项目配置文件即可，我这里的这个主从项目就命名为app1：

```
$ vim /mha/conf/app1.cnf

[server default]
# 链接其他服务器ssh的用户,主要用于心跳检测、IP漂移
ssh_user=root
# 当前Manager的工作目录
manager_workdir=/mha/logs/app1
# 当前Manager的日志目录
manager_log=/mha/logs/manager
# 执行MHA管控的用户，必须在主库上已经创建
user=mha
password=mha
# 主库中binlog的存放目录
master_binlog_dir=/db/mysql57/3306/logs
# 主库为从库复制binlog的用户
repl_user=repl
repl_password=123
# Manager对Master的心跳检测频率
ping_interval=3
# 相关脚本文件路径、自动切换VIP的脚本、手动切换VIP的脚本、发送邮件报警的脚本
# 注意！如果添加了mycat或者atlas等中间件，就可以不用添加VIP脚本master_ip_failover和master_online_change
# 因为我们根本不会链接主库
master_ip_failover_script=/mha/scripts/master_ip_failover
master_ip_online_change_script=/mha/scripts/master_online_change
report_script=/mha/scripts/send_report

[server1]
hostname=192.168.0.120
port=3306

[server2]
# 强制选主的从库
candidate_master=1
# 该从库进行选主时，relaylogs的复制情况不再起到决定因素
check_repl_delay=0
hostname=192.168.0.130
port=3306

[server3]
hostname=192.168.0.140
port=3306

[binlog1]
# 永不参与主库竞选
no_master=1
hostname=192.168.0.210
# binlog_server中存放复制Master的binlog的路径
master_binlog_dir=/data/mysql/binlog
```

注意，我们这里并没有显示的指明谁是主库，但是没有关系，MHA能够自动的鉴别主库是谁。

## **脚本配置**

上面我们指定了3个脚本，MHA将在MASTER宕机之后自动启用它们。

但是这3个脚本还有一些配置项需要我们进行修改。

首先是邮件报警脚本，在设置该脚本前我们需要设置一下邮箱，打开SMTP服务。

以QQ邮箱为示例，点击设置：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021400800.png)

点击邮箱设置，账户：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021400794.png)

往下翻，找到POP3/SMTP服务选择并开启：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021400810.png)

开启POP3/SMTP服务后生成授权码，将授权码保存下来。

然后对send_report脚本进行修改：

```
$ vim /mha/scripts/send_report

my $smtp='smtp service type';
my $mail_from='who will send this email';
my $mail_user='smtp login user';
my $mail_pass='smtp authorization code';
# my $mail_to=['to1@qq.com','to2@qq.com'];
my $mail_to='who will receive this email';

-- 修改为

my $smtp='smtp.qq.com';
my $mail_from='我的QQ邮箱@qq.com';
my $mail_user='我的QQ邮箱@qq.com';
my $mail_pass='我的授权码肯定不给你看呀';
#my $mail_to=['to1@qq.com','to2@qq.com'];
my $mail_to='对方的QQ邮箱@qq.com';   # 可以是其他的邮箱
```

下面还有2个脚本，它们分别是

- master_ip_failover：MHA自动切换主库后进行VIP漂移的脚本
- master_online_change：管理员手动切换主库后进行VIP漂移的脚本

在开始设置之前，先使用ifconfig记录一下你所使用的网卡名称，我这里是ens33（注意！这里必须主从复制中所有的服务器都具有这一块网卡才行）：

```
$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
```

先打开master_ip_failover脚本进行编辑：

```
$ vim /mha/scripts/master_ip_failover

my $vip = 'virtual IP address/virtual IP network segment';
my $key = '1';
my $ssh_start_vip = "/sbin/ifconfig network_card:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig network_card:$key down";

-- 修改为

my $vip = '192.168.0.250/24';  # 一定确保这个地址没有人用
my $key = '1';                 # 一定确保网卡名:1没有人用
my $ssh_start_vip = "/sbin/ifconfig ens33:$key $vip"; # 网卡
my $ssh_stop_vip = "/sbin/ifconfig ens33:$key down";  # 网卡
```

然后打开master_online_change脚本进行编辑：

```
$ vim /mha/scripts/master_online_change

vip=`echo 'virtual IP address/virtual IP network segment'`
key=`echo '1'`

stop_vip=`echo "ssh root@$orig_master_host /usr/sbin/ifconfig network_card:$key down"`
start_vip=`echo "ssh root@$new_master_host /usr/sbin/ifconfig network_card:$key $vip"`

-- 修改为

vip=`echo '192.168.0.250/24'` # 和master_ip_failover的$vip一致
key=`echo '1'`                # 和master_ip_failover的$key一致


stop_vip=`echo "ssh root@$orig_master_host /usr/sbin/ifconfig ens33:$key down"` # 网卡
start_vip=`echo "ssh root@$new_master_host /usr/sbin/ifconfig ens33:$key $vip"` # 网卡
```

下载doc2unix软件对3个脚本进行一次format，这是为了防止有中文信息的存在导致的错误：

```
$ yum install -y dos2unix
$ dos2unix /mha/scripts/{master_online_change,master_ip_failover,send_report}
```

然后一定要为这3个脚本添加执行权限：

```
$ chmod +x /mha/scripts/{master_online_change,master_ip_failover,send_report}
```

最后，我们需要到主库上，执行下面这条命令：

```
$ ifconfig ens33:1 192.168.0.250/24
            #1   #2   #3
            
-- 将#1设置成你刚刚通过ifconfig命令获得的网卡地址
-- 将#2设置成master_ip_failover脚本中$key的值
-- 将#3设置成master_ip_failover脚本中$vip的值
-- 注意！确保ens33:1没有被使用！
```

使用ifcofnig命令可以查看虚拟IP已经被添加上了，后期MASTER出现故障时他自动的进行转移到新MASTER上：

```
$ ifconfig

ens33:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.250  netmask 255.255.255.0  broadcast 192.168.0.255
        ether 00:0c:29:6e:f1:4b  txqueuelen 1000  (Ethernet)
```

## **binlog_server配置**

上面我们在app1的项目配置文件中添加了binlog_server：

```
[binlog1]
# 永不参与主库竞选
no_master=1
hostname=192.168.0.210
# binlog_server中存放复制Master的binlog的路径
master_binlog_dir=/data/mysql/binlog
```

所以现在我们需要到binlog_server上执行一些操作，首先是创建master_binlog_dir的目录：

```
$ mkdir -p /data/mysql/binlog
$ chown -R mysql.mysql /data/*
```

然后需要进入SLAVE1或者SLAVE2上，执行以下命令查看一下当前MASTER正在使用的binlog文件：

```
> SHOW SLAVE STATUS\G;
Master_Log_File: mysql_bin.000005
```

回到binlog_server中，必须进入刚刚创建的master_binlog_dir目录，开始拉取主库的binlog日志：

```
$ cd /data/mysql/binlog 
$ mysqlbinlog -R --host=192.168.0.120 --port=3306 --user=mha --password=mha --raw --stop-never mysql_bin.000005 &
```

## **状态检查**

好了，现在一切就绪，我们可以让MHA验证主从状态了。

在Manager服务端上对1主2从的服务器进行SSH互信检查：

```
$ masterha_check_ssh  --conf=/mha/conf/app1.cnf

-- 出现：All SSH connection tests passed successfully 则互相配置成功
```

在Manager服务器上进行主从状态检查：

```
$ masterha_check_repl  --conf=/mha/conf/app1.cnf

-- 出现：MySQL Replication Health is OK 则主从状态良好
```

确认状态都没问题后，进行下一步操作。

## **MHA命令**

MHA命令后面跟上项目配置文件即可，如下所示。

开启MHA监控：

```
$ nohup masterha_manager --conf=/mha/conf/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /mha/logs/manager.log 2>&1 &
 
--conf：指定配置文件
--remove_dead_master_conf：剔除已经死亡的节点
--ignore_last_failover：MHA默认不能短时间(8小时)多次切换主从，添加此参数可跳过时间检查
```

检查MHA是否正常运行中：

```
$ masterha_check_status --conf=/mha/conf/app1.cnf

-- 当出现is running(0:PING_OK)则MHA正在运行
```

停止当前MHA的监控工作：

```
$ masterha_stop --conf=/mha/conf/app1.cnf
```

## **alias命令**

上面不管是启动、检测、关闭MHA的命令都十分繁琐，我们可以alias一个别名方便后续使用。

在Manager服务器上，执行以下操作：

```
$ vim /etc/profile

alias mha_app1_start="nohup masterha_manager --conf=/mha/conf/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /mha/logs/manager.log 2>&1 &"

alias mha_app1_status="masterha_check_status --conf=/mha/conf/app1.cnf"

alias mha_app1_stop="masterha_stop --conf=/mha/conf/app1.cnf"

$ source /etc/profile
```

# 故障演练

## **发生故障**

现在我们要开始故障模拟了，首先在Manager上使用以下命令查看它的日志变化：

```
$ tail -f /mha/logs/manager
```

然后到MASTER服务器上，执行以下命令关闭mysqld服务：

```
$ systemctl stop mysqld.service
```

回到Manager服务器上，观察日志变化，当出现了以下字样则代表故障转移成功！已经有新的主库产生，若没下面的字样则故障转移一定是失败的：

```
Master failover to 192.168.0.130(192.168.0.130:3306) completed successfully.
```

检查MHA是否运行，会看到它完成了工作，目前已经处于停止状态了：

```
$ mha_app1_status
app1 is stopped(2:NOT_RUNNING).
[1]+  完成                  nohup masterha_manager --conf=/mha/conf/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /mha/logs/manager.log 2>&1
```

登录QQ邮箱，查看邮件是否成功发送，如果成功发送则代表send_report脚本是运行成功的：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112021400290.png)

我们再到SLAVE1上查看VIP有没有转移过来：

```
$ ifconfig

ens33:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.250  netmask 255.255.255.0  broadcast 192.168.0.255
        ether 00:0c:29:f3:f0:1f  txqueuelen 1000  (Ethernet)
```

可以看到VIP已经成功转移，这意味着通过192.168.0.250:3306，我们依然可以访问MySQL服务，这代表master_ip_failover脚本是运行成功的。

在binlog_server上运行一下命令即可进行验证：

```
$ mysql -utom -p123 -h192.168.0.250 -P3306 -e"SELECT @@server_id;"
```

## **故障恢复**

现在，我们应该将已宕机的192.168.0.120库重新启动，加入到集群中：

```
$ systemctl start mysqld.service
$ mysql -uroot
> CHANGE MASTER TO
     MASTER_HOST='192.168.0.130',  -- 注意！现在130是主库了
     MASTER_USER='repl',
     MASTER_PASSWORD='123',
     MASTER_PORT=3306,
     MASTER_CONNECT_RETRY=10,
     MASTER_AUTO_POSITION=1; 
> START SLAVE;
```

检查复制状态，确保IO_T和SQL_T都是Yes，并且记录Master_Log_File：

```
> SHOW SLAVE STATUS\G;

Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Master_Log_File: mysql_bin.000005
```

随着MHA服务的完成，binlogserver也完成了它的工作停止了继续复制master_binlog的行为，我们需要到binlog_server服务器上，删除旧MASTER上拉取的binlog日志，重新拉取新的binlog日志，当然你也可以选择不删除，这都无伤大雅：

```
$ mv /data/mysql/binlog/* /tmp/

$ mysqlbinlog -R --host=192.168.0.130 --port=3306 --user=mha --password=mha --raw --stop-never mysql_bin.000005 &

-- 拉取的日志名填写上面记录的Master_Log_File
```

MHA服务完成后，会将旧主库从配置文件中删除，所以我们需要将旧主库重新加入到配置文件中，在Manager上执行以下操作，添加旧主库到项目配置文件中：

```
$ vim /mha/conf/app1.cnf

[server1]
candidate_master=1
check_repl_delay=0
hostname=192.168.0.120
port=3306
```

做完上面的工作后，我们又可以运行MHA服务啦！只需要在Manager节点上执行以下操作即可：

```
$ mha_app1_start

$ mha_app1_status
app1 (pid:2192) is running(0:PING_OK), master:192.168.0.130
```

## **手动切换**

如果我们想手动的切换主库，必须要停止MHA服务：

```
$ mha_app1_stop
```

然后在Manager上运行以下命令：

```
$ masterha_master_switch --conf=/mha/conf/app1.cnf --master_state=alive --new_master_host=192.168.0.120 --orig_master_is_new_slave --running_updates_limit=10000 --interactive=0

--  Switching master to 192.168.0.120(192.168.0.120:3306) completed successfully. 代表切换成功
```

下一步还是到SLAVE上查看主库目前正在使用的日志：

```
$ show slave status\G;

Master_Log_File: mysql_bin.000006
```

还是老步骤，到binlog_server上重新拉取这个日志：

```
$ mv /data/mysql/binlog/* /tmp/

$ mysqlbinlog -R --host=192.168.0.120 --port=3306 --user=mha --password=mha --raw --stop-never mysql_bin.000006 &

-- 拉取的日志名填写上面记录的Master_Log_File
```

检查VIP是否成功漂移到了192.168.0.120上，如果成功则代表master_online_change脚本运行完成，在新的MASTER上执行以下命令即可：

```
$ ifconfig 

ens33:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
```

这次是我们手动切换的主从身份，所以192.168.0.130会自动变为从库，不用重新CHANGE MASTER TO，所以接下来只需在Manager服务器上执行以下重新开启MHA监控即可：

```
$ mha_app1_start

$ mha_app1_status
app1 (pid:2309) is running(0:PING_OK), master:192.168.0.120
```

至此，MHA的基础使用宣布告一段落，感谢阅读。