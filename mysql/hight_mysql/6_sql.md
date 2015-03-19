 查询性能优化    
===============

查询优化、索引优化、库表结构优化三者相辅相成，缺一不可。查询应遵循一些基本原则，需要掌握一些优化技巧，需要掌握sql执行的流程。

为什么查询速度会变慢
-----------------------

优化查询主要是优化查询的相应时间，每个查询任务由若干个小任务组成，优化的过程就是要么消除一些、要么减少一些、要么让子任务运行更快一些。

查询的生命周期大致顺序：从客户端，到服务器，然后在服务器上进行解析，生成执行计划，执行，返回结果给客户端。其中执行是最重要的阶段，包括了大量的检索数据到存储引擎的调用，调用后的数据处理，分组和排序。

查询任务完成后，依然有时间花销，包括网络、CPU计算、生成执行计划、锁等待（互斥等待），尤其是向存储引擎调用操作，这些调用需要在内存操作、在CPU操作、内存不足时导致I/O操作。

了解查询的生命周期，清楚查询的时间消耗情况是优化查询的基础。

慢查询基础：优化数据访问
-------------------------

* 确认是否在检索大量超过需要的数据，访问了太多的行，访问了太多的列；
* 确认服务器是否在分析大量超过需要的数据行；

优化数据访问，就是优化访问的数据，操作对象是要访问的数据，两方面，是否向服务器请求了大量不需要的数据，二是是否逼迫MySQL扫描额外的记录（没有必要扫描）。

    请求不需要数据的典型案例：不加LIMIT（返回全部数据，只取10条）、多表关联Select * 返回全部列（多表关联查询时*返回多个表的全部列）、还是Select *（可能写程序方面或代码复用方面有好处，但还要权衡）、重复查询相同数据（真需要这样，可以缓存下来，移动开发这个很有必要本地存储）。

    标志额外扫描的三个指标：响应时间（自己判断是否合理值）、扫描的行数、返回的行数，一般扫描行数>返回行数。

    扫描的行数需要与一个“访问类型”概念关联，就是 Explain 中的 type，explain的type结果由差到优分别是：ALL（全表扫描）、index（索引扫描）、range（范围扫描）、ref（唯一索引查询 key_col=xx）、const（常数引用）等。从“访问类型”可以明白，索引让 MySQL 以最高效、扫描行数最少的方式找到需要的记录。

    书中有个例子，说明在where中使用已是索引的列和取消该列的索引后两种结果，type由ref变为All，预估要访问的rows从10变为5073，差异非常明显。


重构查询的方式
----------------

### 将一个复杂查询拆分为数个小且简单的查询，数据返回也快。

虽然原则上，考虑到网络通信、查询解析和优化需要一些花销，所有要求数据库层完成尽可能多的工作。但对特殊情况，将特别复杂的查询分解是可以达到优化效果的，因为MySQL的链接和断开很轻量级，而且网络通信一般都足够快。

### 切分查询，将大查询切分称若干个小查询

如删除大量数据时，用一个大的语句一次性完成的话，可能需要一次性锁定很多数据、占满整个事务日志、耗尽系统资源、阻塞很多其他查询；但切割大查询则可以有效地避免这些问题，如
<pre>
mysql> DELETE FROM `message` WHERE `created` < DATE_SUB(NOW(), INTERVAL 3 MONTH);
// 切分为
rows_affected = 0
do {
  rows_affected = do_query(
    "DELETE FROM `message` WHERE `created` < DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000"
  )
} while_rows_affected > 0

// 如果需要的话，可以在每次小查询执行后暂停一下，可以更有效地分散大语句执行的压力
</pre>

### 分解关联查询：

<pre>
mysql> SELECT * FROM `tag`
  -> JOIN `tag_post` ON `tag_post`.`tag_id` = `tag`.`id`
  -> JOIN `post` ON `tag_post`.`post_id` = `post`.`id`
  -> WHERE `tag`.`tag` = 'mysql';

// 分解为
mysql> SELECT * FROM `tag` WHERE `tag` = 'mysql';
mysql> SELECT * FROM `tag_post` WHERE `tag_id` = 1234;
mysql> SELECT * FROM `post` WHERE `post`.`id` in (123, 456, 789);
</pre>

分解关联缓存的优势
* 让缓存的效率更高，关联查询的话，如果某一个表发生了变化就无法使用查询缓存了；
* 减少锁竞争
* 更容易对数据库进行拆分，更容易提高性能和可扩展性；
* 查询本身的效率也可能会提升；
* 减少冗余记录的查询；
* 结合应用，分解关联查询可能更高效，如应用中可能缓存了一些数据；

查询执行的基础
----------------

查询执行的流程
    * 客户端发送一条查询给服务器；
    * 服务器检查查询缓存，如果命中则返回缓存结果，否则进入下一步；
    * 服务器端进行MySQL解析、预处理，再由优化器生成对应的执行计划；
    * MySQL根据优化器生成的计划，调用存储引擎的API执行查询；
    * 将结果返回给客户端；

### 客户端/服务器通讯协议

MySQL客户端和服务器之间的通信协议是“半双工”的
    * 在任一时刻，要么客户端向服务器发送数据，要么服务器向客户端发送数据，两个动作不会同时发生；
    * 无法进行流量控制，一端发送消息后，另一端要接收完整个消息后才能响应它；
    * 客户端将查询数据包发给服务器，发起后就只能等待结果了，另外如果查询语句特别长的时候，要考虑参数max_allowed_packet；
    * 服务器响应给客户端的数据通常有很多，客户端要完整地接收整个结果，加上LIMIT能有效地减少返回的数据；
    * 客户端只能被动地从服务端接收数据，而不能让服务端停下来，服务端发送完数据后，会释放这条查询占用的资源；
    * 客户端连接数据库的库函数通常可以缓存服务端返回的结果到内存；这样做的缺点是如果数据较大则会占用太多内存；
    * 客户端不缓存服务端的数据；缺点是服务端要等到客户端处理完成后才释放资源；

查询状态，一个MySQL链接或者说一个线程，任何时刻都有一个状态，状态描述了MySQL当前正在做什么
<pre>
// 查看状态，下面命令的Command列就是对应线程的当前状态
SHOW FULL PROCESSLIST 
</pre>

状态的含义
    * Sleep 等待客户端发起新的请求
    * Query 正在执行查询或者正在将结果发给客户端；
    * Locked 正在等待表锁，InnoDB的行锁并不会体现在进程的状态中；
    * Analyzing and statistics 正在搜集存储引擎的统计信息，并生成查询的执行计划；
    * Copying to tmp table [on disk] 正在执行查询，并且将其结果集都赋值到一个临时表中，要么是在做GROUP BY操作，要么是文件排序操作，或者是UNION操作，on disk指正在讲一个内存临时表放到磁盘上；
    * Sorting result 正在对结果集进行排序；
    * Sending data 在多个状态之间传送数据，或者在生成结果集，或者在向客户端返回数据；

### 查询缓存

如果查询缓存是打开的，MySQL会优先检查这个查询缓存中的数据，检查由哈希方式查找实现。如果命中了查询缓存，在返回查询结果之前MySQL会检查一次用户权限，权限没问题，则直接把缓存结果返回给客户端，完成查询任务。

### 查询优化处理





MySQL查询优化器的局限性
------------------------

### 关联子查询

MySQL的子查询实现得非常糟糕，尤其是WHERE中包含IN()的子查询，如
<pre>
mysql> SELECT * FROM `sakila`.`film`
  WHERE `film_id` IN (
    SELECT `film_id` FROM `sakila`.`film_actor` WHERE `actor_id` = 1
  );
</pre>
针对上面的查询，如果先执行子查询，获取子查询结果后，再执行整个语句，查询的效率倒还不错，但是MySQL的处理方式是吧外层表压到子查询中，即：
<pre>
// 可以使用EXPLAIN EXTENDED查看查询被改写成什么样子
SELECT * FROM `sakila`.`film`
WHERE EXISTS (
  SELECT * FROM `sakila`.`film_actor` WHERE `actor_id` = 1
  AND `film_actor`.`film_id` = `film`.`film_id`);
</pre>

### 索引合并优化

当WHERE子句中包含多个复杂条件的时候，MySQL能够访问单个表的多个索引以合并和交叉过滤的方式来定位需要查找的行。

### 松散索引扫描

MySQL不支持松散索引扫描，无法按照不连续的方式扫描一个索引，通常，MySQL的索引扫描需要先定义一个起点和终点，然后扫描这段索引的每一个条目；

### 最大值和最小值优化

MIN()和MAX()查询会扫描全表，可以通过变通的方式优化MIN()操作。

### 在同一个表上查询和更新

MySQL不允许对同一张表同时进行查询和更新，如下面的SQL是无法执行的
<pre>
mysql> UPDATE `tb1` AS `outer_tb1`
  SET `cnt` = (
    SELECT COUNT(*) FROM `tb1` AS `inner_tb1`
    WHERE `inner_tb1`.`type` = `outer_tb1`.`type`
  );
</pre>
可以通过使用生成表的形式绕过上面的限制，
<pre>
mysql> UPDATE `tb1`
  INNER JOIN (
    SELECT `type`, COUNT(*) AS `cnt` FROM `tb1` GROUP BY `type`
  ) AS `der` USING(`type`)
SET `tb1`.`cnt` = `der`.`cnt`;
</pre>

查询优化器的提示（hint）
------------------------

可以使用优化器提供的几个提示（hint）来控制最终的执行计划。常用的提示有

HIGH_PRIORITY和LOW_PRIORITY，指定语句的优先级；
DELAYED：对INSERT和REPLACE有效，即可返回，将数据缓存，然后在表空闲时写入；
STRAIGHT_JOIN：
SQL_SMALL_RESULT和SQL_BIG_RESULT：对SELECT有效，建议结果集放在内存或磁盘；
SQL_BUFFER_RESULT：将查询结果放到一个临时表，然后尽可能快递释放表锁；
SQL_CACHE和SQL_NO_CACHE：结果是否缓存在查询缓存；
SQL_CALC_FOUND_ROWS：
FOR UPDATE和LOCK IN SHARE MODE：
USE INDEX、IGNORE INDEX和FORCE INDEX：

optimizer_search_depth：
optimizer_prune_level：
optimizer_switch：

不建议使用优化器的提示来优化查询，在版本升级和代码维护上增加了隐患。

优化特定类型的查询
--------------------

### 优化 COUNT()查询

COUNT()有两个作用
    * 统计非NULL列的列值的数量;
    * 统计返回数据集的行数；
    
常用的是COUNT(*)，*常被误解为所有列，实际上在操作时是忽略所有列，而直接统计所有行数。COUNT(*)中的*与SELECT *中的*是不同的。如果你真想统计结果集的行数，就用 COUNT(*)而不要使用 COUNT(`colume`)。

通常以为 MyISAM执行COUNT(*)最快，实际上是有条件的，只有不用WHERE时，因为MySQL根本不用扫描数据行，也无须去计算，会直接利用存储引擎的特性去获得这个值。当带上WHERE或者要统计指定列的的数量，就需要去扫描去计算了。

简单优化
<pre>
mysql> SELECT COUNT(*) FROM `world`.`city` WHERE `id` > 5; # 假定符合条件的有10000条数据

// 优化
mysql> SELECT (SELECT COUNT(*) FROM `world`.`city`) - COUNT(*) FROM `world`.`city` WHERE `id` <= 5;
</pre>

使用近似值
业务场景不要求完全精确的COUNT值得时候，可以考虑使用近似值来代替，EXPLAIN出来的优化器估算的行数是一个不错的近似值，执行EXPLAIN并不需要去执行查询，所以成本很低。

COUNT()需要扫描大量的行才能获得精确的结果，所以比较难以优化。可以考虑在MySQL层面做索引覆盖扫描，或者考虑修改应用的架构，增加汇总表，或者增加类似Memcached这样的缓存系统。优化的过程就是“快速、精确和实现简单”三者的一个博弈。

### 优化关联查询

    * 确保ON或USING子句中的列上有索引，在创建索引时就要考虑到关联的顺序，在创建索引的时候就要考虑到关联的顺序，当表A和表B用列c关联时，如果优化器的关联顺序是B、A，那么就不需要在B表的对应列上建索引；没有用到的索引只会带来额外的负担，一般只需要在关联顺序中的第二个表的相应列上创建索引；
    * 确保任何的GROUP BY和ORDER BY中的表达式只涉及到一个表中的列，这样MySQL才有可能使用索引来优化这个过程；
    * MySQL升级的时候要注意：关联语法、运算符优先级等其他可能会发生变化的地方；

### 优化子查询

尽可能使用关联查询代替子查询，当然MySQL5.6或MariaDB可以考虑使用子查询。

### 优化GROUP BY和DISTINCT

这两类查询可以互相转化，可以使用索引来优化，无法使用索引时，GROUP BY使用两种策略来完成：使用临时表或者文件排序来做分组。可以通过配置SQL_BIG_RESULT和SQL_SMALL_RESULT来让优化器按希望的方式运行。

如果对关联查询做分组，并且按照查找表中的某个列进行分组，那么通常采用查找表的标识列分组的效率会比其他列更高，如
<pre>
mysql> SELECT `actor`.`first_name`, `actor`.`last_name`, COUNT(*) 
  FROM `sakila`.`film_actor`
  INNER JOIN `sakila`.`actor` USING(`actor_id`)
  GROUP BY `actor`.`first_name`, `actor`.`last_name`;

// 优化

mysql> SELECT `actor`.`first_name`, `actor`.`last_name`, COUNT(*)
  FROM `sakila`.`film_actor`
  INNER JOIN `sakila`.`actor` USING(`actor_id`)
  GROUP BY `film_actor`.`actor_id`;
</pre>

### 优化LIMIT分页

LIMIT配合ORDER BY是常见的处理分页的SQL，如果有对应的索引，通常效率会不错，没有索引的话则会需要做大量的文件排序操作。

对于偏移量非常大的情况，如LIMIT 10000, 20，MySQL需要查询10020条记录，并且会抛弃前10000条记录。这种语句的优化，要么在页面中限制分页的数量，要么是优化大偏移量的性能。

优化办法是尽可能使用索引覆盖扫描，而不是查询所有的列，然后根据需要做一次关联操作再返回所需的列，如
<pre>
mysql> SELECT `film_id`, `description` FROM `sakila`.`film` GROUP BY `title` LIMIT 50, 5;

// 优化
mysql> SELECT `film`.`film_id`, `film`.`description`
  FROM `sakila`.`film`
  INNER JOIN (
    SELECT `flim_id` FROM `sakila`.`film` ORDER BY `title` LIMIT 50, 5
  ) AS `lim` USING(`film_id`);
</pre>

还可以将LIMIT查询转换为已知位置的查询，让MySQL通过范围扫描获得到对应的结果。如
<pre>
mysql> SELECT `film_id`, `description` FROM `sakila`.`film`
  WHERE poosition BETWEEN 50 AND 54 ORDER BY `position`;
</pre>

还可以直接从记录到的书签位置开始扫描，如
<pre>
mysql> SELECT * FROM `sakila`.`rental`
  ORDER BY `rental_id` DESC LIMIT 20;

mysql> SELECT * FROM `sakila`.`rental`
  WHERE `rental_id` < 16030
  ORDER BY `rental_id` DESC LIMIT 20;
</pre>

此外，还可以通过预先计算的汇总表优化，或者关联到一个冗余表，冗余表中只包含主键列和需要做排序的数据列。

### 优化SQL_CALC_FOUND_ROWS

### 优化UNION查询

MySQL总是通过创建并填充临时表的方式来执行UNION查询，所有优化比较困难。经常需要手工将WHERE、LIMIT、ORDER BY等子句“下推”到UNION的各个子查询，以便优化器可以充分利用这些条件进行优化，如直接将这些子句冗余地写一份到各个子查询。

除非确实需要服务器消除重复的行，否则就一定要使用UNION ALL，如果没有ALL，MySQL会给临时表加上DISTINCT选项，增加查询代价。

### 静态查询分析

Percona Toolkit中的pt-queryadvisor能够解析查询日志、分析查询模式，然后给出所有可能存在问题的查询，并给出建议。

### 使用用户自定义变量

