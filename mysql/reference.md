MySQL索引
==================

数据库开发中索引的使用占了很重要的位置，好的索引会使数据库的读写效率加倍，烂的索引则会拖累整个系统甚至引发灾难。

基于索引包含的字段列数, 可分为单列索引和组合索引
* 单列索引，即一个索引只包含单个列，一个表可以有多个单列索引；
* 组合索引，即一个索引包含多个列；

索引分四类：
* index, 普通的索引, 数据可以重复;
* unique, 唯一索引, 索引列的值必须唯一, 但允许有空值, 如果是组合索引, 则列值的组合必须唯一;
* primary key, 主键索引, 它是一种特殊的唯一索引，不允许有空值,一般是在建表的时候同时创建主键索引,一个表只能有一个主键；
* fulltext, 全文索引;

创建索引
---------

一般的创建索引的语句如下：
<pre>             
// 如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length, 指定length可以加速索引查询速度，还会减少索引文件的大小，提高INSERT的更新速度。

// 普通索引的创建和删除
CREATE TABLE `table_name`(
  `field1` INT NOT NULL,
  `field2` VARCHAR(16) NOT NULL,
  INDEX [`index_name`] (`field2`(length))
);
CREATE INDEX `indexName` ON `table_name`(`field`(length));
ALTER `tableb_name` ADD INDEX [`index_name`] ON (`field_list`(length));
DROP INDEX [`index_name`] ON `table_name`; // 删除索引

CREATE TABLE `table_name`(
  `field1` INT NOT NULL,
  `field2` VARCHAR(16) NOT NULL,
  UNIQUE [`index_name`] (`field2`(length))
); 
CREATE UNIQUE INDEX `index_name` ON `table_name`(`field2`(length));
ALTER `table_name` ADD UNIQUE [`index_name`] ON (`field2`(length));

CREATE TABLE `table_name`(
  `field1` INT NOT NULL,
  `field2` VARCHAR(16) NOT NULL,
  PRIMARY KEY(`field1`)
); 
ALTER TABLE `tb_name` ADD PRIMARY KEY `index_name`(`column_list`);

// 组合索引的创建
CREATE TABLE `table_name`(
  `field1` INT NOT NULL,
  `field2` VARCHAR(16) NOT NULL,
  `field3` VARCHAR(200) NOT NULL,
  `field4` INT NOT NULL,
  PRIMARY KEY(`field1`)
); 
ALTER TABLE `table_name` ADD INDEX `field2_field3_field4`(`field2`, `field3`(10), `field4`); 

// use reference
SELECT * FROM `table_name` WHREE `field2` = 'value2' AND `field3` = 'value3';
SELECT * FROM mytable WHREE `field2` = "value2";
// no reference
SELECT * FROM `table_name` WHREE `field3` = 'value3' AND `field4` = 'value4';
SELECT * FROM mytable WHREE `field3` = "value3";
</pre>

关于组合索引
----------------

如果分别为组合索引用到的字段创建单列索引，让该表有3个单列索引，查询时和上述的组合索引效率也会大不一样，远远低于我们的组合索引。虽然此时有了三个索引，但MySQL只能用到其中的那个它认为似乎是最有效率的单列索引。另外建立这样的组合索引，其实是相当于分别建立了下面三组组合索引：field2,field3,field4; field2,field3; field2; 因为MySQL组合索引“最左前缀”的结果,简单的理解就是只从最左面的开始组合,并不是只要包含这三列的查询都会用到该组合索引。


建立索引的时机
----------------

MySQL 中会对 <，<=，=，>，>=，BETWEEN，IN 以及不以%开头的like 语句使用索引。

一般来说，在WHERE和JOIN中出现的列需要建立索引，但也不完全如此，因为MySQL只对<，<=，=，>，>=，BETWEEN，IN，以及某些时候的LIKE才会使用索引,LIKE以通配符%和_开头作查询时，MySQL不会使用索引。例如：
<pre>
// 可以为field1创建单列索引; 为field2,field3创建组合索引
SELECT `t1`.`field1` FROM `table_name1` AS `t1` LEFT JOIN `table_name2` AS `t2`
  ON `t1`.`field1` = `t2`.`field1` WHERE `t2`.`field2` = 'value2' AND `t2`.`field3` = 'value3';
// 可以为field1创建索引
SELECT * FROM `table_name` WHERE `field1` LIKE 'somestring%';
// 无法使用索引
SELECT * FROM `table_name` WHERE `field1` LIKE '%somestring';
</pre>

此外，查看索引的使用情况
<pre>
show status like 'Handler_read%';
// handler_read_key:这个值越高越好，越高表示使用索引查询到的次数 
// handler_read_rnd_next:这个值越高，说明查询低效
</pre>

索引失效
1、WHERE字句的查询条件里有不等于号（WHERE column!=...），MYSQL将无法使用索引
2、类似地，如果WHERE字句的查询条件里使用了函数（如：WHERE DAY(column)=...），MYSQL将无法使用索引
3、在JOIN操作中（需要从多个数据表提取数据时），MYSQL只有在主键和外键的数据类型相同时才能使用索引，否则即使建立了
 索引也不会使用
4、如果WHERE子句的查询条件里使用了比较操作符LIKE和REGEXP，MYSQL只有在搜索模板的第一个字符不是通配符的情况下才能
使用索引。比如说，如果查询条件是LIKE 'abc%',MYSQL将使用索引；如果条件是LIKE '%abc'，MYSQL将不使用索引。
5、在ORDER BY操作中，MYSQL只有在排序条件不是一个查询条件表达式的情况下才使用索引。尽管如此，在涉及多个数据表的查
询里，即使有索引可用，那些索引在加快ORDER BY操作方面也没什么作用。
6、如果某个数据列里包含着许多重复的值，就算为它建立了索引也不会有很好的效果。比如说，如果某个数据列里包含了净是
些诸如“0/1”或“Y/N”等值，就没有必要为它创建一个索引。

7、索引有用的情况下就太多了。基本只要建立了索引，除了上面提到的索引不会使用的情况下之外，其他情况只要是使用在
WHERE条件里，ORDER BY 字段，联表字段，一般都是有效的。 建立索引要的就是有效果。 不然还用它干吗？ 如果不能确定在
某个字段上建立的索引是否有效果，只要实际进行测试下比较下执行时间就知道。

    * 如果条件中有or，即使其中有条件带索引也不会使用(这也是为什么尽量少用or的原因)，注意：要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引；
    * 对于多列索引，不是使用的第一部分，则不会使用索引
    * like查询是以%或_开头
    * 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引
    * 如果mysql估计使用全表扫描要比使用索引快,则不使用索引


下面是MYSQL占用CPU高处理的一个例子，希望对遇到类似问题的朋友们有点启发。一般来说MYQL占用CPU高，多半是数据库查询代码问题，查询数据库过多。所以一方面要精简代码，另一方面最好对频繁使用的代码设置索引
下面是MYSQL占用CPU高处理的一个例子，希望对遇到类似问题的朋友们有点启发。一般来说MYQL占用CPU高，多半是数据库查询代码问题，查询数据库过多。所以一方面要精简代码，另一方面最好对频繁使用的代码设置索引。

今天早上起来 机器报警 一查负载一直都在4以上

top了一下 发现 mysql 稳居 第一 而且相当稳定 我擦
重启一下mysql不行
mysql> show processlist;一下
发现xxx网站有两条 查询语句 一直 在列，我擦 该站 也就30多万条记录 量也不大 不可能是机器性能问题

忽然 记得以前在网上看过说是 tmp_table_size值太小会造成这种情况；
于是mysql -pxxx -e "show variables;" >tmp
一看是默认的32M（显示出来的是字节数）
于是翁就开心的改了起来 增加到256 重启 mysql 。。结果很失望

不行啊 还得再来
select 一下该表 发现 里面 都是论坛留言的东西 量还挺大
于是：
mysql> show columns from bbs_message;
+-----------+--------------+------+-----+---------+----------------+
| Field | Type | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+----------------+
| msg_id | int(11) | NO | PRI | NULL | auto_increment |
| board_id | int(11) | NO | MUL | 0 | |
| parent_id | int(11) | NO | MUL | 0 | |
| root_id | int(11) | NO | MUL | 0 | |

一直在show processlist 里面出现的 就是 select * from bbs_message where board_id=xxx and parent_id=xxx
和 select * from bbs_message where parent_id=xxx
只要这两条一出现 cpu就上去了
于是 从索引入手：
增加两条索引
mysql> alter table bbs_message add index parentid(parent_id);
alter table bbs_message add index chaxunid(board_id,parent_id);
最后查看一下索引结果：
mysql> show index from bbs_message;
+-------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment |
+-------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| bbs_message | 0 | PRIMARY | 1 | msg_id | A | 2037 | NULL | NULL | | BTREE | |
| bbs_message | 1 | rootid | 1 | root_id | A | 49 | NULL | NULL | | BTREE | |
| bbs_message | 1 | chaxunid | 1 | board_id | A | 3 | NULL | NULL | | BTREE | |
| bbs_message | 1 | chaxunid | 2 | parent_id | A | 135 | NULL | NULL | | BTREE | |
| bbs_message | 1 | parentid | 1 | parent_id | A | 127 | NULL | NULL | | BTREE | |
+-------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
5 rows in set (0.00 sec)
退出 在 top 一下 负载一直在0.x 很稳定

MySQL建表与索引使用规范
--------------------------

    * MySQL建表，字段需设置为非空，需设置字段默认值。
    * MySQL建表，字段需NULL时，需设置字段默认值，默认值不为NULL。
    * MySQL建表，如果字段等价于外键，应在该字段加索引。
    * MySQL建表，不同表之间的相同属性值的字段，列类型，类型长度，是否非空，是否默认值，需保持一致，否则无法正确使用索引进行关联对比。
    * MySQL使用时，一条SQL语句只能使用一个表的一个索引。所有的字段类型都可以索引，多列索引的属性最多15个。
    * 如果可以在多个索引中进行选择，MySQL通常使用找到最少行的索引，索引唯一值最高的索引。
    * 建立索引index（part1,part2,part3）,相当于建立了 index(part1),index(part1,part2)和index（part1,part2,part3）三个索引。
    * MySQL针对like语法必须如下格式才使用索引： SELECT * FROM t1 WHERE key_col LIKE 'ab%' ；
    * SELECT COUNT() 语法在没有where条件的语句中执行效率没有SELECT COUNT(col_name)快，但是在有where条件的语句中执行效率要快。
    * 在where条件中多个and的条件中，必须都是一个多列索引的key_part属性而且必须包含key_part1。各自单一索引的话，只使用遍历最少行的那个索引。
    * 在where条件中多个or的条件中，每一个条件，都必须是一个有效索引。
    * ORDER BY 后面的条件必须是同一索引的属性，排序顺序必须一致（比如都是升序或都是降序）。
    * 所有GROUP BY列引用同一索引的属性，并且索引必须是按顺序保存其关键字的。
    * JOIN 索引，所有匹配ON和where的字段应建立合适的索引。
    * 对智能的扫描全表使用FORCE INDEX告知MySQL，使用索引效率更高。
    * 定期ANALYZE TABLE tbl_name为扫描的表更新关键字分布 。
    * 定期使用慢日志检查语句，执行explain，分析可能改进的索引。
    * 条件允许的话，设置较大的key_buffer_size和query_cache_size的值（全局参数），和sort_buffer_size的值（session变量，建议不要超过4M）。

主键的命名采用如下规则：
主键名用pk_开头，后面跟该主键所在的表名。主键名长度不能超过30个字符。如果过长，可对表名进行缩写。缩写规则同表名的缩写规则。主键名用小写的英文单词来表示。
 
外键的命名采用如下规则：
外键名用fk_开头，后面跟该外键所在的表名和对应的主表名（不含t_）。子表名和父表名自己用下划线（_）分隔。外键名长度不能超过30个字符。如果过长，可对表名进行缩写。缩写规则同表名的缩写规则。外键名用小写的英文单词来表示。

索引的命名采用如下规则：
    * 索引名用小写的英文字母和数字表示。索引名的长度不能超过30个字符。
    * 主键对应的索引和主键同名。
    * 唯一性索引用uni_开头，后面跟表名。一般性索引用ind_开头，后面跟表名。
    * 如果索引长度过长，可对表名进行缩写。缩写规则同表名的缩写规则

index 相关语法
例:
CREATE INDEX log_url ON logaudit_log(url);
show index from logaudit_log
drop index log_request_time on logaudit_log

sql执行效率检测 mysql explain
explain显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。
使用方法，在select语句前加上explain就可以了：
如：explain select surname,first_name form a,b where a.id=b.id
分析结果形式如下：
table |  type | possible_keys | key | key_len  | ref | rows | Extra
EXPLAIN列的解释：
table
显示这一行的数据是关于哪张表的
type
这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL
possible_keys
显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句
key
实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE
INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引
key_len
使用的索引的长度。在不损失精确性的情况下，长度越短越好
ref
显示索引的哪一列被使用了，如果可能的话，是一个常数
rows
MYSQL认为必须检查的用来返回请求数据的行数
Extra
关于MYSQL如何解析查询的额外信息。将在表4.3中讨论，但这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢
extra列返回的描述的意义
Distinct
一旦MYSQL找到了与行相联合匹配的行，就不再搜索了
Not exists
MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，
就不再搜索了
Range checked for each
Record（index map:#）
没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一
Using filesort
看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行
Using index
列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候
Using temporary
看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上
Where used
使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题
不同连接类型的解释（按照效率高低的顺序排序）
system
表只有一行：system表。这是const连接类型的特殊情况
const
表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待
eq_ref
在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用
ref
这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好
range
这个连接类型使用索引返回一个范围中的行，比如使用>或
FAQ
1
表中包含 10 万条记录，有一个 datetime 类型的字段。
取数据的语句：
SELECT * FROM my_table WHERE created_at < '2010-01-20';
用 EXPLAIN 检查，发现 type 是 ALL， key 是 NULL，根本没用上索引。
可以确定的是，created_at 字段设定索引了。
什么原因呢？
用 SELECT COUNT(*) 看了一下符合 WHERE 条件的记录总数，居然是 6W 多条！！
难怪不用索引，这时用索引毫无意义，就好像 10 万条记录的用户表，有个性别字段，不是男就是女，在这种字段设置索引是错误的决定。
稍微改造一下上述语句：
SELECT * FROM my_table WHERE created_at BETWEEN '2009-12-06' AND '2010-01-20';
这回问题解决！
符合条件的记录只有几百条，EXPLAIN 的 type 是 range，key 是 created_at，Extra 是 Using where 。
自己总结个准则，索引的目的就是尽量缩小结果集，这样才能做到快速查询。

6万条记录符合条件，已经超出总记录数的一半，这时索引已经没有意义了，因此 MySQL 放弃使用索引。
这与设置 gender 字段，并加上索引的情况相似，当你要把所有男性记录都选取出来，符合条件的记录数约占总数的一半，MySQL 同样不会使用这个索引。
唯一值越多的字段，使用索引的效果越好。
设置联合索引时，唯一值越多的，越应该放在“左侧”。

索引的不足之处和使用时注意事项
---------------------------------

    * 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。
    * 建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快。
    * 索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。

使用索引时，有以下一些技巧和注意事项：

    * 索引不会包含有NULL值的列，只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为NULL。
    * 使用短索引，对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR(255)的列，如果在前10个或20个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。
    * 索引列排序，MySQL查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。
    * like语句操作，一般情况下不鼓励使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引而like “aaa%”可以使用索引。
    * 不要在列上进行运算，select * from users where YEAR(adddate)<2007; 将在每个行上进行运算，这将导致索引失效而进行全表扫描，因此我们可以改成 select * from users where adddate<‘2007-01-01’; 
    * 不使用NOT IN和<>操作



所有MySQL列类型可以被索引。根据存储引擎定义每个表的最大索引数和最大索引长度。
所有存储引擎支持每个表至少16个索引，总索引长度至少为256字节。大多数存储引擎有更高的限制。

索引的存储类型
----------------

存储类型只有两种(btree和hash)
hash 索引结构的特殊性，其检索效率非常高，索引的检索可以一次定位，不像btree(B-Tree)索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以 hash 索引的查询效率要远高于 btree(B-Tree) 索引。存储类型和存储引擎

    * MyISAM, btree
    * InnoDB, btree
    * MEMORY/Heap, hash btree 默认情况MEMORY/Heap存储引擎使用hash索引

虽然 hash 索引效率高，但是 hash 索引本身由于其特殊性也带来了很多限制和弊端，主要有以下这些。
    * hash 索引仅仅能满足=，<=>，IN，IS NULL或者IS NOT NULL查询，不能使用范围查询。由于 hash 索引比较的是进行 hash 运算之后的 hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 hash 算法处理之后的 hash 值的大小关系，并不能保证和hash运算前完全一样。
    * hash 索引无法被用来避免数据的排序操作。由于 hash 索引中存放的是经过 hash 计算之后的 hash 值，而且hash值的大小关系并不一定和 hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；
    * hash 索引不能利用部分索引键查询。对于组合索引，hash 索引在计算 hash 值的时候是组合索引键合并后再一起计算 hash 值，而不是单独计算 hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，hash 索引也无法被利用。
    * hash 索引在任何时候都不能避免表扫描。前面已经知道，hash 索引是将索引键通过 hash 运算之后，将 hash运算结果的 hash 值和所对应的行指针信息存放于一个 hash 表中，由于不同索引键存在相同 hash 值，所以即使取满足某个 hash 键值的数据的记录条数，也无法从 hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果。
    * hash 索引遇到大量hash值相等的情况后性能并不一定就会比B-Tree索引高。对于选择性比较低的索引键，如果创建 hash 索引，那么将会存在大量记录指针信息存于同一个 hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下

B-Tree 索引是 MySQL 数据库中使用最为频繁的索引类型，除了 Archive 存储引擎之外的其他所有的存储引擎都支持 B-Tree 索引。不仅仅在 MySQL 中是如此，实际上在其他的很多数据库管理系统中B-Tree 索引也同样是作为最主要的索引类型，这主要是因为 B-Tree 索引的存储结构在数据库的数据检 索中有非常优异的表现。
    一般来说， MySQL 中的 B-Tree 索引的物理文件大多都是以 Balance Tree 的结构来存储的，也就是所有实际需要的数据都存放于 Tree 的 Leaf Node ，而且到任何一个 Leaf Node 的最短路径的长度都是完全相同的，所以我们大家都称之为 B-Tree 索引当然，可能各种数据库（或 MySQL 的各种存储引擎）在存放自己的 B-Tree 索引的时候会对存储结构稍作改造。
如 Innodb 存储引擎的 B-Tree 索引实际使用的存储结构实际上是 B+Tree ，也就是在 B-Tree 数据结构的基础上做了很小的改造，在每一个Leaf Node 上面出了存放索引键的相关信息之外，还存储了指向与该 Leaf Node 相邻的后一个 LeafNode 的指针信息，这主要是为了加快检索多个相邻 Leaf Node 的效率考虑。 
    在 Innodb 存储引擎中，存在两种不同形式的索引，一种是 Cluster 形式的主键索引（ Primary Key ），另外一种则是和其他存储引擎（如 MyISAM 存储引擎）存放形式基本相同的普通 B-Tree 索引，这种索引在 Innodb 存储引擎中被称为 Secondary Index 。
    在 Innodb 中如果通过主键来访问数据效率是非常高的，而如果是通过 Secondary Index 来访问数据的话， Innodb 首先通过 Secondary Index 的相关信息，通过相应的索引键检索到 Leaf Node之后，需要再通过 Leaf Node 中存放的主键值再通过主键索引来获取相应的数据行。
MyISAM 存储引擎的主键索引和非主键索引差别很小，只不过是主键索引的索引键是一个唯一且非空 的键而已。而且 MyISAM 存储引擎的索引和 Innodb 的 Secondary Index 的存储结构也基本相同，主要的区别只是 MyISAM 存储引擎在 Leaf Nodes 上面出了存放索引键信息之外，
再存放能直接定位到 MyISAM 数据文件中相应的数据行的信息（如 Row Number ），但并不会存放主键的键值信息。


我们这里建表
*/

CREATE TABLE mytable(  
 
id INT,   
 
username VARCHAR(16),

city VARCHAR(16),

age INT
 
);

/*
索引分单列索引和组合索引。单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。组合索引，即一个索包含多个列。
MySQL索引类型包括：
（1）普通索引，这是最基本的索引，它没有任何限制。它有以下几种创建方式：
*/
-- 创建索引
CREATE INDEX indexName ON mytable(username(10));               -- 单列索引
-- CREATE INDEX indexName ON mytable(username(10),city(10));   -- 组合索引
-- indexName为索引名，mytable表名，username和city为列名，10为前缀长度，即索引在该列从最左字符开始存储的信息长度，单位字节
-- 如果是CHAR，VARCHAR类型，前缀长度可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 前缀长度，下同。

-- 修改表结构来创建索引
ALTER TABLE mytable ADD INDEX indexName (username(10));
-- ALTER TABLE mytable ADD INDEX indexName (username(10),city(10));
-- 此处 indexName 索引名可不写，系统自动赋名 username ，username_2 ，username_3，...

-- 创建表的时候直接指定
CREATE TABLE mytable(

id INT,

username VARCHAR(16),

city VARCHAR(16),

age INT,

INDEX indexName (username(10))-- INDEX indexName (username(10),city(10))

);
-- 此处 indexName 索引名同样可以省略

/*
（2）唯一索引，它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式（仅仅在创建普通索引时关键字 INDEX 前加 UNIQUE）：
*/
-- 创建索引
CREATE UNIQUE INDEX indexName ON mytable(username(10));
-- 修改表结构来创建索引
ALTER TABLE mytable ADD UNIQUE INDEX indexName (username(10));-- 也可简写成 ALTER TABLE mytable ADD UNIQUE indexName (username(10));
-- 创建表的时候直接指定
CREATE TABLE mytable(

id INT,

username VARCHAR(16),

city VARCHAR(16),

age INT,

UNIQUE INDEX indexName (username(10)) -- 也可简写成 UNIQUE indexName (username(10))

);


/*
（3）主键索引，它是一种特殊的唯一索引，不允许有空值。在建表的时候同时创建的主键即为主键索引
主键索引无需命名，一个表只能有一个主键。主键索引同时可是唯一索引或者全文索引，但唯一索引或全文索引不能共存在同一索引
*/
-- 修改表结构来创建索引
ALTER TABLE mytable ADD PRIMARY KEY (id);
-- 创建表的时候直接指定
CREATE TABLE mytable(

id INT,

username VARCHAR(16),

city VARCHAR(16),

age INT,

PRIMARY KEY(id) 

);

/*
（4）全文索引，InnoDB存储引擎不支持全文索引
*/
-- 创建索引
CREATE FULLTEXT INDEX indexName ON mytable(username(10));
-- 修改表结构来创建索引
ALTER TABLE mytable ADD FULLTEXT INDEX indexName (username(10));-- 也可简写成 ALTER TABLE mytable ADD FULLTEXT indexName (username(10));
-- 创建表的时候直接指定
CREATE TABLE mytable(

id INT,

username VARCHAR(16),

city VARCHAR(16),

age INT,

FULLTEXT INDEX indexName (username(10)) -- 也可简写成 FULLTEXT indexName (username(10))

)ENGINE=MYISAM;
-- 建表时创建全文索引，要设置该表的存储引擎为MYISAM,新版mysql默认InnoDB存储引擎不支持全文索引


-- 删除索引
DROP INDEX indexName ON mytable;


/*
Mysql自动使用索引规则：
btree索引
当使用 <，<=，=，>=，>，BETWEEN，IN，!=或者<>，以及某些时候的LIKE才会使用btree索引，因为在以通配符%和_开头作查询时，MySQL不会使用索引。btree索引能用于加速ORDER BY操作
hash索引
当使用=，<=>，IN，IS NULL或者IS NOT NULL操作符时才会使用hash索引，并且不能用于加速ORDER BY操作，并且条件值必须是索引列查找某行该列的整个值

对where后边条件为字符串的一定要加引号，字符串如果为数字mysql会自动转为字符串，但是不使用索引。
mysql目前不支持函数索引，只能对列的前一部分（指定长度前缀）进行索引，对于char和varchar列，使用前缀索引（该列从起始字符到指定长度字符位置建立索引）将大大节省空间。
mysql列建议列是非null的。说是如果是允许null的列，对索引会有影响（索引不会包括有NULL值）。因为它们使得索引、索引的统计信息以及比较运算更加复杂。应该用0、一个特殊的值或者一个空串代替空值。
尽量不使用NOT IN和<>操作

username,city,age建立这三列的组合索引，其实是相当于分别建立了下面三组组合索引：

username,city,age

username,city

username

使用组合索引，比如where等条件，列名必须从组合索引最左列至右连续的列名做条件才可以，组合索引类似单一索引前缀

*/
-- 调用组合索引
SELECT * FROM mytable WHERE username = 'admin' AND city = 'DaLian';
-- 多表联查中的条件也可运用索引



-- 全文索引的使用
SELECT * , MATCH (username,city) AGAINST ('name100 name200 city500 thisisname') FROM mytable WHERE MATCH (username,city) AGAINST ('name100 name200 city500 thisisname');
-- 返回 mytable 表在 MATCH 的参数中的任意列中包含 AGAINST 字符串参数中被 "空格" "," 和 "." 分割的任意单词的行(断字的字符： "空格" "," 和 "." 但是不用这些符号断字的语言，如中文，就得自行手动断字。)
-- 表列中的单词也是以空格分割来区分
-- 这里指定 MATCH...AGAINST 两次。这不会引起附加的开销，因为 MySQL 优化器会注意到两次是同样的 MATCH...AGAINST 调用，并只调用一次全文搜索代码。
/*
函数 MATCH() 对照一个列名集(一个 FULLTEXT 索引中的一个或多个列的列名)。搜索字符串做为 AGAINST() 的参数给定。搜索以忽略字母大小写的方式执行，预设搜寻是不分大小写，若要分大小写，列的字符集设置要从utf8改成utf8_bin。
虽然同一个表格可以有不同字符集的字段，但是同一个FULLTEXT 索引里的字段必须是同一个字符集与collation。
任何在 stopword 列表上出现的，或太短的(3 个字符或更少的)的单词将被忽略。(可以覆写内建的 stopword 列表。可以修改最少四个字符的设定。 )
搜索的词有一个权重性，如果搜索的词在表中包含它的行太多，则这个搜索词将有较低的权重(可能甚至有一个零权重)，否则，它将得到一个较高的权重。然后，权重将被结合用于计算行的相似性。
如果搜索词在表中超过一半的行中出现。则它被有效地处理为一个 stopword (即，一个零语义值的词)，不搜索该词。

MATCH...AGAINST可以跟所有MySQL语法搭配使用，像是JOIN或是加上其他过滤条件。

对于表中的每行记录，MATCH...AGAINST 返回一个相关性值。即，返回的每行与搜索条件之间的相似性尺度。

当 MATCH() 被使用在一个 WHERE 子句中时，返回的结果被自动地以相关性从高到底的次序排序。如果即没有 WHERE 也没有 ORDER BY 子句，返回行是不排序的。 
相关性值是正值的浮点数字。零相关性意味着不相似。
相关性的计算是基于：查找单词在表行中的数目、在行中唯一词的数目、在集中词的全部数目和包含一个特殊词的行的数目。

到 4.0.1 时，MySQL 也可以使用 IN BOOLEAN MODE 修饰语来执行一个逻辑全文搜索。
IN BOOLEAN MODE的特色： 
不剔除50%以上符合的row。 
不自动以相关性反向排序。 
可以对没有FULLTEXT 索引的字段进行搜寻，但会非常慢。 
限制最长与最短的字符串。 
套用stopwords。

逻辑全文搜索支持下面的操作符：
+ 一个领头的加号表示，返回的结果的每行都必须包含有该单词。
- 一个领头的减号表示，包含该单词的行不能出现在返回的结果中。
> 操作符增加包含该单词返回行相似性值的基值。
< 操作符减少包含该单词返回行相似性值的基值。
() 被括号包含的多个词只相当一个词，即在查询时括号里只有一个词可代表该括号与括号外的词相结合做查询，但括号中每个词都会依次被轮到代表该括号，所以与括号外单词会产生多种结合形式，依次做查询条件。
~ 将其相关性由正转负，表示拥有该字会降低相关性，但不像 - 将之排除，只是排在较后面。
* 一个星号是截断操作符，它应该被追加到一个词后，不加在前面。作用类似 LIKE 语句中的 %
"" 把被双引号包含的多个词作为一个词

这里是一些示例，在返回结果中：
1.+apple +juice ... 两个词均在被包含
2.+apple macintosh ... 包含词 “apple”，但是如果同时包含 “macintosh”，它的排列将更高一些
3.+apple -macintosh ... 包含 “apple” 但不包含 “macintosh”
4.+apple +(>pie <strudel) ... 包含 “apple” 和 “pie”，或者包含的是 “apple” 和 “strudel” (以任何次序)，但是“apple pie” 排列得比 “apple strudel” 要高一点
5.apple* ... 包含 “apple”，“apples”，“applesauce” 和 “applet”
6."some words" ... 可以包含 “some words of wisdom”，但不是 “some noise words”
*/
SELECT *,MATCH (username,city) AGAINST ('>>name300 +thisisname -city100' IN BOOLEAN MODE) FROM mytable WHERE MATCH (username,city) AGAINST ('>>name300 +thisisname -city100' IN BOOLEAN MODE);
/*
全文索引的限制
MATCH() 函数的所有参数必须是从来自于同一张表的列，同时必须是同一个FULLTEXT 索引中的一部分，除非 MATCH() 是 IN BOOLEAN MODE 的。

MATCH() 列必须确切地匹配表的某一 FULLTEXT 索引中定义的列，除非 MATCH() 是 IN BOOLEAN MODE 的。

AGAINST() 的参数必须是一个常量字符串。



MySQL全文搜寻设定： 
大部分的参数都是启动参数，也就是修改后必须重新启动MySQL。 
有些参数修改必须重新产生索引文件。 
mysql> SHOW VARIABLES LIKE 'ft%'; 

ft_boolean_syntax    + -><()~*:""&| 
ft_min_word_len    4 
ft_max_word_len    84 
ft_query_expansion_limit   20 ft_stopword_file    (built-in) 

ft_min_word_len：最短的索引字符串，默认值为4，修改后必须重建索引文件。 
ft_max_word_len：最长的索引字符串，默认值因版本而不同，余同上一点。 
[mysqld] 
ft_min_word_len=1 
ft_stopword_file：stopword档案路径，若留空白不设定表示要停用stopword过滤，修改后必须重新启动MySQL和重建索引；stopword档案内容可以用分行空白与逗号区隔stopword，但底线和单引号视为合法的字符串字符。 
50%的门坎限制：配置文件在storage/myisam/ftdefs.h，将 #define GWS_IN_USE GWS_PROB 改为 #define GWS_IN_USE GWS_FREQ，然后重新编译MySQL，因为近低门坎会影响数据的精准度，所以不建议如此，可用IN BOOLEAN MODE即可以避开50%的限制。 
ft_boolean_syntax：改变IN BOOLEAN MODE的查询字符，不用重新启动MySQL也不用重建索引。 
修改字符串字符的认定，譬如说将「-」认定为字符串的合法字符： 
方法一：修改storage/myisam/ftdefs.h的true_word_char()与misc_word_char()，然后重新编译MySQL，最后重建索引。 
方法二：修改字符集档，然后在FULLTEXT index的字段使用该字符集，最后重建索引。 
重建索引： 
每个有FULLTEXT index的表格都要这么做。 
mysql> REPAIR TABLE tbl_name QUICK; 
要注意如果用过myisamchk，会导致上述的设定值回复成默认值，因为myisamchk不是用MySQL的设定值。 
解法一：将修改过得设定值加到myisamchk的参数里。 
shell> myisamchk --recover --ft_min_word_len=1 tbl_name.MYI 
解法二：两边都要设定。 
[mysqld] 
ft_min_word_len=1 
[myisamchk] 
ft_min_word_len=1 
解法三：用REPAIR TABLE、ANALYZE TABLE、OPTIMIZE TABLE与ALTER TABLE取代myisamchk语法，因为这些语法是由MySQL执行的。

中文全文索引可以建两个表，一个表字段里存中文，一个表对应字段存汉语拼音，两表行必须对应，数据一致，插入时中文转化下汉语拼音，两表都插入
查询时也转化下，全文索引查汉语拼音，然后找到中文表对应行
或者使用mysqlcft中文全文索引插件



查看索引使用情况
如果索引正在工作，Handler_read_key的值将很高，这个值代表了一个行被索引值读的次数，很低的值表明增加索引得到的性能改善不高，因为索引并不经常使用。
Handler_read_rnd_next的值高则意味着查询运行低效，并且应该建立索引补救。这个值的含义是在数据文件中读下一行的请求数。如果你正进行大量的表扫描，该值较高。通常说明表索引不正确或写入的查询没有利用索引。

语法：SHOW STATUS LIKE 'Handler_read%';


MyISAM表的数据文件和索引文件是自动分开的
InnoDB的数据和索引是存储在同一个表空间里面，但可以有多个文件组成

虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。

建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快。
