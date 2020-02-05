在任何一种数据库中，都会有各种各样的日志，记录着数据库工作的方方面面，以帮助数据库管理员追踪数据库曾经发生过的各种事件。MySQL 也不例外，在 MySQL 中，有 4 种不同的日志，分别是错误日志、二进制日志（BINLOG 日志）、查询日志和慢查询日志，这些日志记录着数据库在不同方面的踪迹。

## 1. 慢查询日志

### 1.1 介绍

（1）MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过 long_query_time 值的SQL，则会被记录到慢查询日志中。
（2）具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time 默认为 10 秒，最小为 0， 精度可以到微秒。
（3）由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。

### 1.2 使用

默认情况下，MySQL 数据库没有开启慢查询日志，需要我们手动来设置这个参数。
当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。
慢查询日志支持将日志记录写入文件。

#### 1.2.1开启设置

| SQL语句                                   | 描述          | 备注                                      |
| --------------------------------------- | ----------- | --------------------------------------- |
| SHOW VARIABLES LIKE '%slow_query_log%'; | 查看慢查询日志是否开启 | 默认情况下slow_query_log 的值为 OFF，表示慢查询日志是禁用的 |
| set global slow_query_log=1;            | 开启慢查询日志     |                                         |
| SHOW VARIABLES LIKE 'long_query_time%'; | 查看慢查询设定阈值   | 单位秒                                     |
| set long_query_time=1                   | 设定慢查询阈值     | 单位秒                                     |

#### 1.2.2 如永久生效需要修改配置文件 my.cnf 中[mysqld]下配置

```
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/slow-sql.log
long_query_time=3
log_output=FILE
```

## 2. 错误日志

错误日志是 MySQL 中最重要的日志之一，它记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。当数据库出现任何故障导致无法正常使用时，可以首先查看此日志。

该日志是默认开启的 ， 默认存放目录为 mysql 的数据目录（var/lib/mysql）, 默认的日志文件名为  hostname.err（hostname是主机名）。

查看日志位置指令 ： 

```sql
show variables like 'log_error%'; 
```

查看日志内容 ： 

```shell
tail -f /var/lib/mysql/xaxh-server.err
```

## 3. 二进制日志

### 3.1 概述

二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但是不包括数据查询语句。此日志对于灾难时的数据恢复起着极其重要的作用，MySQL的主从复制， 就是通过该binlog实现的。

二进制日志，默认情况下是没有开启的，需要到MySQL的配置文件中开启，并配置MySQL日志的格式。 

配置文件位置 : /usr/my.cnf

日志存放位置 : 配置时，给定了文件名但是没有指定路径，日志默认写入Mysql的数据目录。

```
#配置开启binlog日志， 日志的文件前缀为 mysqlbin -----> 生成的文件名如 : mysqlbin.000001,mysqlbin.000002
log_bin=mysqlbin

#配置二进制日志的格式
binlog_format=STATEMENT
```

### 3.2 日志格式

**STATEMENT**

该日志格式在日志文件中记录的都是SQL语句（statement），每一条对数据进行修改的SQL都会记录在日志文件中，通过Mysql提供的mysqlbinlog工具，可以清晰的查看到每条语句的文本。主从复制的时候，从库（slave）会将日志解析为原文本，并在从库重新执行一次。

**ROW**

该日志格式在日志文件中记录的是每一行的数据变更，而不是记录SQL语句。比如，执行SQL语句 ： update tb_book set status='1' , 如果是STATEMENT 日志格式，在日志中会记录一行SQL文件； 如果是ROW，由于是对全表进行更新，也就是每一行记录都会发生变更，ROW 格式的日志中会记录每一行的数据变更。

**MIXED**

这是目前MySQL默认的日志格式，即混合了STATEMENT 和 ROW两种格式。默认情况下采用STATEMENT，但是在一些特殊情况下采用ROW来进行记录。MIXED 格式能尽量利用两种模式的优点，而避开他们的缺点。
