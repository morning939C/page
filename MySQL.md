# MySQL

## 1、mysql中锁的种类以及实现原理

## 2、主从配置

### 准备条件

1. 数据库版本需要一致
2. 网络互通

### Master配置

#### 文件配置

```shell
# 主库宿主机上需要增加配置
vim /etc/my.cnf

[mysqld]
bin-log=mysql-bin # 开启bin-log
server-id=1 # 机器ID，默认为1，主从库机器该值不能相同
binlog_format=MIXED
sync_binlog=1
expire_logs_days=0
binlog-do-db=test # test为数据库的名称

```

#### 创建授权用户

```shell
# 文件配置修改完成之后需要重启mysql
systemctl restart mysqld

# 查看Mysql的运行状态
systemctl status mysqld

# 进入mysql
mysq -uroot -p

# 创建一个从机用户，并指定可以远程连接
create user 'slave'@'%' identified by '密码';
# 设置slave远程从机账户拥有一个可以复制的权限
grant replication slave, replication client on *.* to 'slave'@'%';

```

#### 查看主库宿主机的日志和master状态

```shell
# 查看log_bin日志是否开启 若value为ON表示开启；
show variabels like 'log_bin';

# 查看当前宿主机是否为master状态
show master status;
# 记住结果中File和Position
```

### Slave配置

#### 文件配置

```shell
# 修改文件中的server-id的值
vim /etc/my.cnf

[mysqld]
server-id=2
bin-log=mysql-bin # 不是必须
```

#### 执行相关命令

```shell
# 若修改配置文件之后需要重启Mysql
# 进入MySQL后，需要先关掉slave
stop slave;
# 执行下面的语句
change master to 
master_host='主库IP',
master_port=3306, # 同样是主库的端口号
master_user='slave', # 主库中之前创建的用户
master_password='密码', # 上面用户的密码
master_log_file='File', # 之前主库查看master状态时File的值
master_log_pos=157; # master-log_pos的值为之前查看master状态时Position的值
```

#### 查看同步状态

```shell
show slave status\G
# 必须保证Slave_IO_Running:Yes 和 Slave_SQL_Running:Yes，则搭建成功
```

### 遇到的问题

- Slave_IO_Running和Slave_SQL_Running都为No；

>原因是主从库中的uuid相同，将从库的auto.cnf文件删除后重启就可以了
>
>```shell
># 查看uuid的命令
>show variables like '%uuid%';
>
># 备份auto.cnf
>cd /var/lib/mysql
>mv auto.cnf ./auto.cnf.bak
># 删除auto.cnf
>rm -rf auto.cnf
>
># 重启从库mysql
>systemctl restart mysqld
>```
>
>

- Slave_IO_Running的值为CONNECTING ,且报错信息为`Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connect`

  > 原因为该Mysql版本为8.0，slave用户的plugin是**caching_sha2_password**，需要改为**mysql_native_password**
  >
  > ```shell
  > alter user 'slave'@'%' identified with mysql_native_password by '密码';
  > flush privileges;
  > ```
  >
  > 执行以上命令后解决。

