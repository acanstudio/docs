推荐本SQL优化的书《Effective MySQL之SQL语句最优化》。
主要讲解：如何去分析SQL的性能、索引的原理、如何创建合适的索引、如何去分析线上系统的性能瓶颈。

另外还介绍了几个辅助工具：
mysqldumpslow 来分析慢查询日志；
Google开源的mysql-slow-query-log-parser 分析慢查询日志；
应用程序中使用MySQL Proxy来收集SQL语句、QEP、查询执行时间；
开源Maatkit检查数据库中的重复索引；
Google的MySQL补丁，引入SHOW INDEX_STATISTICS 用来分析索引；
MySQL补丁microsecond-mysql-client可以对SQL进行微妙级别的监控；
开源sqlstats插件，统计SQL语句；

电子版下载地址：http://download.csdn.net/detail/bbirdsky/8207119

书不厚也就200页，以下是此书的目录：

DBA五分钟速成
================

DBA的基本技能
    * 确认运行缓慢的查询
    * 识别缺失的索引
    * 应用新的索引
    * 验证新的索引

识别性能问题 
-------------

当应用程序太慢了，在确定不存在物理系统资源瓶颈之后，应优先检查MySQL数据库了。

#### 寻找运行缓慢的SQL语句

<pre>
# 获取当前正在执行的一些sql语句的状态
# 通常能看到影响性能的sql语句
mysql> SHOW FULL PROCESSLIST\G;
</pre>

#### 确认低效查询

要确认这个查询是否每次执行都很缓慢，验证这个低效查询并不是由于锁定或者系统瓶颈等其他原因造成的。

重复执行sql语句
<pre>
mysql> SELECT * FROM `inventory` WHERE `item_id` = 16102176;
Empty set (3.19sec)

# 如果执行时间大于10毫秒，mysql工具会显示出语句执行时间
# 重复执行sql语句只适用于SELECT语句，对于UPDATE或DELETE，可以重写为SELECT语句
</pre>

生成一个查询执行计划（Query Execution Plan,QEP），MySQL执行一个SQL查询，会对该SQL语句进行语法检查，然后构造一个QEP，QEP决定了MySQL从底层存储引擎获取信息的方式。
<pre>
mysql> EXPLAIN SELECT * FROM `inventory` WHERE `item_id` = 16102176\G;

**************************1.row******************
           id: 1
  select_type: SIMPlE
        table: inventory
         type:ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 787338
        Extra: Using where

# EXPLAIN通常不实际执行SQL语句，
# 不过当优化器需要执行sql的一部分以便决定如何构造QEP时，
# EXPLAIN也会执行sql，此时select_type会显示DERIVED字样
# key显示NULL，type显示ALL，这样的sql通常需要优化，
# rows的值对sql的性能影响也很显著，不过需要注意的是InnoDB引擎的rows是个估算值
</pre>

优化查询 
---------

#### 添加一个索引

基于WHERE子句在表上添加索引是优化sql的一个方法，需要指出的是，添加索引之前应充分验证索引带来的影响。

<pre>
mysql> ALTER TABLE `inventory` ADD INDEX (`item_id`);

# 添加索引的数据定义语音(DDL)通常执行时间较长，而且是阻塞操作，
# 其他数据操作语音(DML)会因阻塞而无法完成
# 索引还可能会引起其他DML语句的性能
</pre>

添加索引之前通常应该至少做两项检查：首先验证表现有的结构，然后确认表的大小。
<pre>
# 查看指定表结构
mysql> SHOW CREATE TABLE `inventory\G

# 查看表的当前状态，包括存储引擎、大小等信息
mysql> SHOW TABLE STATUS LIKE 'inventory'\G
</pre>

#### 确认优化

重新运行sql语句验证性能是否明显改善
<pre>
mysql> SELECT * FROM `inventory` WHERE `item_id` = 16102716;
# 执行时间小于10毫秒

mysql> EXPLAIN SELECT * FROM `inventory` WHERE `item_id` = 16102716;
# type: ref; key: item_id; rows:1 
</pre>

基本的分析命令
===============

EXPLAIN命令
------------

通过这个命令深入了解MySQL的基于开销的优化器，还可以获得很多可能被优化器考虑到的访问策略的细节，以及当运行sql语句时哪种策略预计会被优化器采用。需要注意的是，QEP是动态的，由sql语句每次执行时的实际情况确定，MySQL不会将一个QEP和某个给定sql预计绑定。

理想情况下，应该对系统中的每条sql语句都运行EXPLAIN命令。EXPLAIN暴露出的问题
    * key: NULL，没有使用索引
    * rows: 很多行，过多的行很影响性能
    * possible_keys，很多需要评估的索引

#### EXPLAIN PARTITIONS

<pre>
# 对用于满足在partitions列中的查询的特定表分区提供附加信息。
mysql> EXPLAIN PARTITIONS SELECT * FROM `audit_log` WHERE `yr` IN (2011, 2012)\G
partitions: p2, p3
</pre>

#### EXPLAIN EXTENDED

提供了额外的filtered列
<pre>
mysql> EXPLAIN EXTENDED SELECT `t1`.`name` FROM `test1` `t1` INNER JOIN `test2` `t2` USING(`uid`)\G
filtered: 100.00

# 查看最后一次的警告信息
mysql> SHOW WARNINGS\G
</pre>

SHOW CREATE TABLE命令
----------------------

查看表结构，包括雷和索引的定义细节，是否为NULL、数据类型、缺省值、字符集、存储引擎等信息。
<pre>
mysql> SHOW CREATE TABLE `wp_options`\G
# 数据库、表、列和索引的引号'`'不是必需的，
# 只有当使用的对象名是一个保留字或者其中包含空格是才是必须的

# 通过mysqldump工具生成表结构
$ mysqldump -uuser -p --no-data [data-of-schema] > schema.sql

# MySQL提供的辅助数据库`INFORMATION_SCHEMA`中有很多根表结构相关的表
# TABLES、COLUMNS、TABLE_CONSTRAINTS、KEY_COLUMN_USAGE、
# PEFERENTIAL_CONSTRAINTS、PARTITIONS
</pre>

SHOW INDEXES命令
-----------------

查看索引的信息，包括索引的类型和当前报告的MySQL索引的基数。

<pre>
mysql> SHOW INDEXES FROM `wp_posts`;

# 也可以从INFORMATION_SCHEMA.STATISTICS表中获取同样的信息
# Cardinality列的值非常重要，表示在索引中每一列唯一值的数量的估计值
</pre>

SHOW TABLE STATUS命令
-----------------------

查看数据表的底层大小以及表结构，其中包括存储引擎、版本、数据和索引大小、行的平均长度以及行数。

<pre>
mysql> SHOW TABLE STATUS LIKE `wp_posts`\G

# 也可以从INFORMATION_SCHEMA.TABLES表中获取底层表的信息
SET @schema = IFNULL(@schema, DATABASE());
SET @table='inventory';
SELECT @schema AS `table_schema`, CURDATE() AS `today`;
SELECT `table_name`, `engine`, `row_format` AS `format`, `table_rows`
    `avg_row_length` AS `avg_row`, round((data_lengt+index_length) / 1024/ 1024, 2) AS `total_mb`,
    round((data_length) / 1024 / 1024, 2) AS `data_mb`
    round((index_length) / 1024 / 1024, 2) AS `index_mb`
FROM `INFOMATION_SCHEMA`.`tables` WHERE `table_schema` = @schema AND `table_name` = @table \G

# INFORMATION_SCHEMA表的SELECT查询可能需要执行很长时间
</pre>

SHOW STATUS命令
----------------

查看MySQL服务器的当前内部状态信息，可以从全局角度帮助用户确定MySQL服务器的负载的各项指标。

<pre>
# SHOW [GLOBAL | SESSION] STATUS，默认为SESSION
mysql> SHOW GLOBAL STATUS LIKE 'Created_tmp_%tables';
mysql> SHOW SESSION STATUS LIKE 'Created_tmp_%tables';
mysql> SHOW STATUS LIKE 'Created_tmp_%tables';

# 用户也可以从INFORMATION_SCHEMA.GLOBAL_STATUS表和SESSION_STATUS表中获取同样信息

mysql> FLUSH STATUS;
mysql> SELECT ...;
mysql> SHOW STATUS LIKE 'handler_read%';
# 基于Handler_read_*的相关值查看sql使用索引的情况
</pre>

SHOW VARIABLES命令
-------------------

查看系统变量的当前值，有些值会影响到sql语句的执行方式，如tmp_table_size限制了内部创建临时表的最大内存使用量。通过了解当前会话或者特殊设置的全局变量的值并用SET命令动态地更改这些变量的值，可以改变sql语句的性能。

<pre>
# SHOW [GLOBAL | SESSION] VARIABLES，默认为SESSION
mysql> SHOW VARIABLES LIKE 'tmp_table_size';
mysql> SHOW SESSION VARIABLES LIKE 'tmp_table_size';
mysql> SHOW GLOBAL VARIABLES LIKE 'tmp_table_size';

# 用户也可以从INFORMATION_SCHEMA.GLOBAL_VARIABLES表和SESSION_VARIABLES表中获取同样信息
</pre>

INFORMATION_SCHEMA
-------------------

-----------------------------------------------------
命令               | 对应INFORMATION_SCHEMA中的表
---------------------------------------------------
SHOW CREATE TABLE  | TABLES COLUMNS KEY_COLUMN_USAGE PEFERENTIAL_CONSTRAINTS
---------------------------------------------------------------
SHOW INDEXES       | STATISTICS
----------------------------------------------------------------
SHOW TABLE STATUS  | TABLES
------------------------------------------------------------------
SHOW STATUS        | GLOBAL_STATUS SESSION_STATUS
--------------------------------------------------------------
SHOW VARIABLES     | GLOBAL_VARIABLES SESSION_VARIABLES


