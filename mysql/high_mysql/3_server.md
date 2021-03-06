服务器性能剖析
===============

如何确认服务器是否达到了最佳性能状态；如何找出某条语句为什么执行不够快；如何诊断被我们描述成“停顿”、“卡死”、“堆积”的某些间歇性疑难故障。
通过一些工具和技巧来优化整机的性能、优化单条语句的执行速度、诊断或解决那些很难观察到的问题；专注与测量服务器的时间花费在哪里，使用的技术则是性能剖析（profiling）；如何测量系统并生成剖析报告，如何分析系统的整个堆栈，包括从应用程序到数据库服务器到单个查询；

性能优化简介
-------------

本书中将MySQL性能定义为完成某件任务所需要的时间度量，即相应时间。通过任务和时间而不是资源来测量性能。 注意区别通常所关注的每秒查询次数、CPU利用率、可扩展性等性能问题。本书中默认向MySQL发送的一切命令都称为查询，诸如SELECT/UPDATE/DELETE等等。

性能优化，在一定的工作负载下尽可能低降低相应时间。

如果认为性能是响应时间，那么我们的目标就是降低响应时间，降低每个查询的响应时间，那么第二个问题，就是先搞清楚时间花在哪里，即测量时间花在什么地方，无法测量就无法有效优化。

注意，测量范围不要搞错，如出现慢查询，就应该测量或解决慢查询，不应该去排查整个服务器的状况；另外，查询的时间包括等待时间和执行时间，一般处理和解决执行时间，等待时间长的原因很复杂。

通过工具可以显示性能剖析的结果，但注意很多情况是通过结果发现不了的，注意寻找哪些是值得优化的查询、哪些发生了异常、还有很多被隐藏的细节（如平均响应时间很小，但的确存在1～2个真正耗时的慢查询）

对应用程序进行性能剖析
----------------------

影响性能瓶颈的因素
    * 外部资源，比如调用了外部的Web服务或者搜索引擎；
    * 应用需要处理大量的数据，比如分析一个超大的XML文件；
    * 在循环中执行昂贵的操作，如滥用正则表达式；
    * 使用了低效的算法，如暴力搜索算法；

除了对MySQL服务器进行性能剖析，还有必要对应用程序本身做性能剖析，比如PHP程序；值得注意的是性能剖析也会使服务器变慢，但相对剖析程序所做得贡献来说，那点消耗已经无所谓；

推荐了进行PHP性能剖析的工具，xhprof、Ifp等，不用重新发明“轮子”。

剖析MySQL查询
--------------

对查询的剖析有两种方式，一种是剖析整个MySQL服务器负载，一种是剖析单条语句查询；

剖析服务器负载，可以通过MySQL提供的慢查询日志，现在通过设置long_query_time可以将粒度设置到微妙级别；MySQL提供的慢查询日志是开销最低、精度最高的测试查询时间的工具。新浪的SAE就提供针对MySQL的慢查询日志。

剖析单条查询，使用SHOW PROFILES、SHOW STATUS、查询慢查询日志的条目。先执行一条查询，然后使用SHOW PROFILES查看查询执行的时间，然后使用SHOW PROFILE WHERE QUERY 1，显示该条查询执行的过程，及每个过程所话费的时间，经过排序后，就可以知道 查询把时间花在那个过程上了。

诊断间歇性问题
---------------


其他剖析工具
-------------


总结
-----

