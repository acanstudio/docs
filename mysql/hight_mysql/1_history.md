MySQL架构与历史
================

MySQL的存储引擎
-----------------

MySQL将每个数据库（schema）保存为数据目录下的一个子目录，创建表示，MySQL会在数据库子目录下创建一个和表同名的.frm文件保存表的定义；数据库和表的命名是否大小写敏感，跟具体的平台相关；不同的存储引擎保存数据和索引的方式是不同的，但表的定义则是在MySQL服务层同一处理；

表的相关信息
<pre>
mysql> SHOW TABLE STATUS LIKE 'user' \G

Name: 表名；
Engine：表的存储引擎类型，（旧版使用Type）；
Row_format：行的格式，MyISAM可选的值为Dynamic（行长度可变）、Fixed（固定长度）、Compressed（存在于压缩表中）；
Rows：表中的行数，InnoDB，该值为估算值，MyISAM是精确值；
Avg_row_length：平均每行包含的字节数；
Data_length：数据的大小（字节为单位）；
Max_data_length：表数据的最大容量，和存储引擎有关；
Index_length：索引的大小（以字节为单位）；
Data_free：对于MyISAM，表示已经分配单目前没有使用的空间，包括了之前删除的行，以及后续可以被INSERT利用的空间；
Auto_increment：下一个AUTO_INCREMENT的值；
Create_time：表的创建时间；
Update_time：表数据的最后修改时间；
Check_time：使用CHECK TABLE命令或myisamchk工具最后一次检查表的时间；
Collation：表的默认字符集和字符列排序规则；
Checksum：如果启用，保存的是整个表的事实校验和；
Create_options：创建表时指定的其他选项；
Comment：其他额外信息；

MySQL时间线（Timeline）
------------------------

版本3.23（2001）
标识找MySQL的诞生，开始获得广泛使用。在平面文件（Flat File）实现SQL查询；引入MyISAM代替ISAM引擎；可以使用InnoDB，但需要手工编译；引入全文索引和复制，复制时MySQL称为互联网应用的数据库系统的关键特性。

版本4.0（2003）
支持新的语法，如UNION和夺标DELETE语法；重写了复制，在备库使用了两个线程来实现复制，避免了一个线程做所有复制工作的模式下任务切换导致的问题；InnoDB称为标配，包含了行锁定、外键等；引入查询缓存；支持通过SSL进行连接。

版本4.1（2005）
引入更多新的语法，如子查询，INSERT ON DUPLICATE KEY UPDATE；支持UTF-8字符集；支持新的二进制协议和prepared语句。

版本5.0（2006）
出现企业级特性，视图、触发器、存储过程和存储函数；移除ISAM引擎；引入Federated等新引擎。

版本5.1（2008）
Sun收购MySQL AB后发布的首个版本，研发时间长达五年；引入了分区、基于行的复制；引入plugin API（包括可插拔存储引擎的API）；移除BerkeyDB引擎；Oracle收购的InnoDB Oy发布了InnoDB plugin。

版本5.5（2010）
Oracle收购Sun之后发布的首个版本；改善集中在性能、扩展性、复制、分区、对Windows系统的支持；InnoDB称为默认的存储引擎；移除了更多的不建议使用的特性；增加了PERFORMANCE_SCHEMA库，包含了一些可测量的性能指标的增强；增加了复制、认证和审计API；半同步复制插件进入实用阶段；InnoDB在架构方面做了较大的改进，如多个子缓冲池（buffer pool）。

版本5.6（还未发布）
首次对查询优化器进行大规模的改进；更多的插件API，如全文索引等；复制得改进；PERFORMANCE_SCHEMA库增加了更多的性能指标；InnoDB做了大量的改进。

版本6.0（已经取消）
包括大量新特性，包括在线备份、服务器层面对所有存储引擎的外键支持；子查询的改进和线程池；这些新特性陆续出现在5.5和5.6版本中。

早起的MySQL是一种破坏性创新，有诸多限制，并且很多功能只能说是二流的。但它的特性支持和较低的使用成本，使其成为互联网应用最为广泛的数据库。5.x版本的早期，MySQL引入视图和存储过程等特性，期望成为“企业级”数据库，但并不成功；Sun和Oracle的收购，也使得社区人士有所担心；5.5版本可以说是MySQL质量最高的版本；MySQL更好地王企业级应用的方向发展，5.6承诺在功能和性能方面将有显著提升。

MySQL的开发模式
-----------------

发布GA（General Available）版本前的开发版本已经包含了GA版的新特性，充分测试后，会以GA版本正式发布；实验室预览版本里的部分特性不保证会在正式版中发布。

MySQL遵循GPL开源协议，全部源代码（除了一些商业版本的插件）都会开放给社区。（不同版本的策略被Sun废除了）以插件的形式提供收费服务。

将更多的特性做成插件的开发模式是一个亮点，如果InnoDB的全文索引功能以API的方式实现，那么就可能以同样的API实现Sphinx或者Lucene的插件。

