PHP数据库长连接mysql_pconnect的细节
====================================

PHP的MySQL持久化连接，美好的目标，却拥有糟糕的口碑，往往令人敬而远之。这到底是为啥么。近距离观察后发现，这家伙也不容易啊，要看Apache的脸色，还得听MySQL指挥。对于作为Apache模块运行的PHP来说，要实现MySQL持久化连接，首先得取决于Apache这个web服务器是否支持Keep-Alive。

Keep-Alive
----------------

Keep-Alive是什么东西？它是http协议的一部分，让我们复习一下没有Keep-Alive的http请求，从客户在浏览器输入一个有效url地址开始，浏览器就会利用socket向url对应的web服务器发送一条TCP请求，这个请求成功一次就得需要来回握三次手才能确定，成功以后，浏览器利用socket TCP连接资源向web服务器请求http协议，发送以后就等着web服务器把http返回头和body发送回来，发回来后浏览器关闭socket连接，然后做http返回头和body的解析工作，最后呈现在浏览器上的就是漂亮的页面了。这里面有什么问题呢？TCP连接需要三次握手，也就是来回请求三次方能确定一个TCP请求是否成功，然后TCP关闭呢？来回需要4次请求才能完成！每次http请求就3次握手，4次拜拜，这来来回回的不嫌累啊，多少时间和资源都被浪费在socket连接关闭上了，能不能一次socket TCP连接发送多次http请求呢？于是Keep-Alive就应运而生，http/1.0里需要客户端自己在请求头加入Connection:Keep-alive方能实现，在这里我们只考虑http1.1了，只需要设置一下Apache，让它默认就是Keep-Alive持久连接模式(Apache必须1.2+才能支持Keep-Alive)。在httpd.conf里找到KeepAive配置项，果断设置为On，MaxKeepAliveRequests果断为0(一个持久TCP最多允许的请求数，如果过小，很容易在TCP未过期的情况下，达到最大连接，那下次连接就又是新的TCP连接了，这里设置0表示不限制)，然后对于mysql_pconnect最重要的选项KeepAliveTimeout设置为15(表示15秒)。

好了，重启Apache，测试一下，赶紧写行东西：

::
    <?php
        echo "Apache进程号:". getmypid();

很简单，获取当前PHP执行者(Apache)的进程号，用浏览器浏览这个页面，看到什么？对，有看到一串进程号数字，15秒内，连续刷新页面，看看进程号有无变化？木有吧？现在把手拿开，交叉在胸前，度好时间，1秒，2秒，3，...15，16。好，过了15秒了，再去刷新页面，进程号有没有变化？变了！又是一个新的Apache进程了，为什么15秒后就变成新的进程了？记得我们在Apache里设置的KeepAliveTimeout吗？它的值就是15秒。现在我们应该大致清楚了，在web服务器默认打开KeepAlive的情况下，客户端第一次http成功请求后，Apache不会立刻断开socket，而是一直监听来自这一客户端的请求，监听多久？根据KeepAliveTimeout选项配置的时间决定，一旦超过这一时间，Apache就会断开socket了，那么下次同一客户端再次请求，Apache就会新开一个进程来相应。所以我们之前15内不停的刷新页面，看到的进程号都是一致的，表明是浏览器请求给了同一个Apache进程。

浏览器是怎么知道不需要重新进行TCP连接就可以直接发送http请求呢？因为http返回头里就会带上Connection:keep-alive，Keep-alive:15两行，意思就是让客户端浏览器明白，这次socket连接我这边还没关闭呢，你可以在15内继续使用这个连接，并发送http请求，于是乎浏览器就知道应该怎么做了。

PHP的MySQL连接
-----------------

那么，PHP的MySQL连接资源是怎么被hold住的呢，这需要查看PHP的mysql_pconnect的函数代码，我看了下，大概的做法就是mysql_pconnect根据当前Apache进程号，生成hash key，找hash表内有无对应的连接资源，没有则推入hash表，有则直接使用。有些代码片段可以说明(具体可查看PHP5.3.8源码ext/mysql/PHP_mysql.c文件690行PHP_mysql_do_connect函数)
01	#1.生成hash key
02	user=php_get_current_user();//获取当前PHP执行者(Apache)的进程唯一标识号
03	//hashed_details就是hash key
04	hashed_details_length = spprintf(&hashed_details, 0, "MySQL__%s_", user);
05	#2.如果未找到已有资源，就推入hash表，名字叫persistent_list，如果找到就直接使用
06	/* try to find if we already have this link in our persistent list */
07	if (zend_hash_find(&EG(persistent_list), hashed_details, hashed_details_length+1, (void **) &le)==FAILURE) { 
08	    /* we don't */
09	    ...
10	    ...
11	    /* hash it up(推入hash表) */
12	    Z_TYPE(new_le) = le_plink;
13	    new_le.ptr = mysql;
14	    if (zend_hash_update(&EG(persistent_list), hashed_details, hashed_details_length+1, (void *) &new_le, sizeof(zend_rsrc_list_entry), NULL)==FAILURE) {
15	        ...
16	        ...     
17	        }
18	    }
19	    else
20	    {/* The link is in our list of persistent connections(连接已在hash表里)*/
21	        ...
22	        ...
23	        mysql = (PHP_mysql_conn *) le->ptr;//直接使用对应的sql连接资源
24	        ...
25	        ...
26	    }

zend_hash_find比较容易看明白，原型是zend_hash_find(hash表,key名,key长,value);如果找到，value就有值了。

MySQL的wait_timeout和interactive_timeout
-----------------------------------------

说完Keep-Alive，该到MySQL家串串门了，说的是mysql_pconnect，怎么能绕开MySQL的设置。影响mysql_pconnect最重要的两个参数就是wait_timeout和interactive_timeout，它们是什么东西？先撇一边，首先让我们把上面的代码改动一下PHP代码

::

    <?php
        $conn = mysql_pconnect("localhost","root","123456") or die("Can not connect to MySQL");
	echo "MySQL线程号:". MySQL_thread_id($conn). "<br />";
	echo "Apache进程号". getmypid();

以上的代码没啥好解释的，让我们用浏览器浏览这个页面，看到什么？看到两个显眼的数字。一个是MySQL线程号，一个是Apache进程号，好了，15秒后再刷新这个页面，发现这两个id都变了，因为已经是新的Apache进程了，进程id是新的，hash key就变了，PHP只好重新连接MySQL，连接资源推入persistent list。如果15内刷新呢？Apache进程肯定不变，MySQL线程号会变吗？答案得问MySQL了。首先这个MySQL_thread_id是什么东西？shell方式登录MySQL后执行命令'show processlist;'，看到了什么？

	mysql> show processlist;
	+-----+------+-----------+------+--------+-----+------+-----------------+
	| Id  | User | Host      | db   | Command| Time| State| Info            |
	+-----+------+-----------+------+--------+-----+------+-----------------+
	| 348 | root | localhost | NULL | Query  |    0| NULL | show processlist|
	| 349 | root | localhost | NULL | Sleep  |    2|      | NULL            |
	+-----+------+-----------+------+--------+-----+------+-----------------+

发现了很重要的信息，这个processlist列表就是记录了正在跑的线程，忽略Info列为show processlist那行，那行是你当前shell登录MySQL的线程。PHP连接MySQL的线程就是Id为349那行，如果读者自己做测试，应该知道这个Id=349在你的测试环境里是另外一个值，我们把这个值和网页里输出的MySQL_thread_id($conn)做做比较，对！他们是一样的。接下来最重要的是观察Command列和Time列，Command = Sleep，表明什么？表明我们mysql_pconnect连接后就一直在sleep，Time字段就告诉我们，这个线程Sleep了多久，那么Sleep了多久这个线程才能作废呢？那就是wait_timeout或者interactive_timeout要做的工作了，他们默认的值都是8小时，天啊，太久了，所以如果说web服务器关掉KeepAlive支持，那个这个processlist很容易就被撑爆，就爆出那个Too many connections的错误了，max_connectiosns配置得再多也没用。为了观察这两个参数，我们可以在MySQL配置文件my.cnf里设置这两个值，找到[MySQLd]节点，在里面设置多两行

::

    interactive_timeout = 60
    wait_timeout        = 30

配置完后，重启MySQL，shell登录MySQL，这时候show processlist可以发现只有当前线程。然后运行那个带有mysql_pconnect的PHP页面，再回来MySQL端show processlist可发现，多了一个Commond为Sleep的线程，不停的show processlist(方向键上+enter键)观察Time列的变化2，5，10...14!，突然那个Sleep线程程被kill掉了，咋回事，还没到30秒呢，噢!忘了修改一下Apache keepalive的参数了，把KeepAliveTimeOut从15改成120(只为观察，才这么改)，重启Apache。刷新那个页面，好，开始不停的show processlist，2..5..10..14，15，..20...26....28，29!线程被kill，这次是因为wait_timeout起了作用，浏览器那边停了30秒，30内如果浏览器刷新，那这个Time又会从0开始计时。这种连接不属于interactive connection(MySQL shell登录那种连接就属于interactive connection)，所以采用了wait_timeout的值。如果mysql_pconnect的第4个参数改改呢

::

    <?php
        $conn = mysql_pconnect('localhost','root','123456',MySQL_CLIENT_INTERACTIVE);
        echo "MySQL线程号:".MySQL_thread_id($conn)."<br />";
        echo "Apache进程号:".getmypid();

刷新下页面，MySQL那边开始刷show processlist，这回Time > 30也不会被kill，>60才被kill了，说明设置了MySQL_CLIENT_INTERACTIVE，就会被MySQL视为interactive connection，那么这次PHP的MySQL连接在120秒内未刷新的情况下，何时作废将取决于MySQL的interactive_timeout的配置值。

总结
-----------

PHP的mysql_pconnect要达到功效，首先必须保证Apache是支持keep alive的，其次KeepAliveTimeOut应该设置多久呢，要根据自身站点的访问情况做调整，时间太短，keep alive没啥意义，时间太长，就很可能为一个闲客户端连接牺牲很多服务器资源，毕竟hold住socket监听进程是要消耗cpu内存的。最后Apache的KeepAliveTimeOut配置得和MySQL的time out配置要有个平衡点，联系以上的观察，假设mysql_pconnect未带上第4个参数，如果Apache的KeepAliveTimeOut设置的秒数比wait_timeout小，那真正对mysql_pconnect起作用的是Apache而不是MySQL的配置。这时如果MySQL的wait_timeout偏大，并发量大的情况下，很可能就一堆废弃的connection了，MySQL这边如果不及时回收，那就很可能Too many connections了。可是如果KeepAliveTimeOut太大呢，又回到之前的问题，所以貌似Apache。KeepAliveTimeOu不要太大，但比MySQL。wait_timeout 稍大，或者相等是比较好的方案，这样可以保证keep alive过期后，废弃的MySQL连接可以及时被回收。Pdo数据库的长连接机制是否和mysql_pconnect一样?经过试验观察和源码探究，发现也是一样的处理方式。 


mysql_pconnect() 和 mysql_connect() 都是打开一个到 MySQL 服务器的连接，但有两个主要区别：一是当连接的时候本函数将先尝试寻找一个在同一个主机上用同样的用户名和密码已经打开的（持久）连接，如果找到，则返回此连接标识而不打开新连接。其次，当脚本执行完毕后到 SQL 服务器的连接不会被关闭，此连接将保持打开以备以后使用（mysql_close() 不会关闭由 mysql_pconnect() 建立的连接）

但使用Pconnect会经常的导致Mysql连接失败,提示连接太多,原因在于pconnect后,Apache不会自动关闭mysql的连接.

先来看看APACHE的工作模式，Windows 下,Apache使用一个主进程,加一个辅进程,再由辅进程派生N个线程的方式来提供服务,线程的数量可以在httpd.conf里配置: ThreadsPerChild 500,如果指定为500线程,则apache一启动时就会启动500个线程,但最多也只使用500个线程,如果同时连接数量超过500个(可能300个用户访问就有500个连接,判断当前连接的方法,可以使用netstat -na|grep 80|grep EST|wc -l或者使用apache的status module),那么,多余的连接将会在等待或者连接失败.(所以,Windows下Apache的主要配置参数应该是ThreadsPerChild, 先根据当前的连接数,再看看有没有必要调大一些,一般PC服务器设置为1000算是比较大了.)
Nix下,Apache使用进程的方式来运行,原理相同,需要调整进程数量的参数有几个,比如ServerLimit.

再来看看Apache+PHP+Mysql_pconnect的工作方式，每当客户端向服务端发送一个连接请求(包括图片,HTML,PHP等)，apache将会用一个线程来接受这个请求，如果是请求的是一个PHP文件,且 PHP文件里使用了PConnect,则当前线程会判断当前线程有没有打开过pconnect,如果有打开过,则使用原来的mysql connect,如果没有打开过,则新建一个connect,并且,连接断开后,线程仍在运行,而且保持Mysql connect.按这种方式运行一段时间后,完全有可能所有apache的线程都打开过有Pconnect的Php页面,所以,如果apache的 ThreadsPerChild=500的话,则500个线程都找开了mysql连接,并且没有关闭,则就要求,mysql的连接数必须大于或等于 500,如果小于这个值,将会导致PHP页面提示数据库连接失败.

所以,得出结论,Apache+PHP+Mysql下使用 pconnect时,mysql的max_connect必须大于或等于apache的最大线程(进程)数.在一个访问量很大的站点,使用 pconnect可能不太现实,最好的办法是,尽可能的将数据库内容生成为静态文件,而不需要每个页面都连接数据库,并且使用mysql_connect (即使将绝大多数页面生成为静态文件,但仍有mysql_pconnect时,同样要求mysql的max_connect大于apache的线程数,所以这种情况下使用pconnect非常不可取).

只从程序上来讲，很空洞。你去自己实际验证下，建立一个pconnect的连接。然后你去mysql下show processlist，看看，然后再建立几个，再去mysql下show processlist。你会发现有很多死连接处于sleep状态。 当这种状态的数量超过了mysql设置的最大连接数，mysql除了root就谁也连不上了。因为mysql总是会为root留一个位置。 这种长连接只有到达my.cnf里面设置的超时时间后才会自动断开，默认好像是8天吧，忘记了。然后你的mysql error日志中会记录一个warnning，翻译过来就是胎死腹中的连接。 所以说实际开发中，如果你的应用属于多用户的，一定不要使用这种连接。除非是那种单人应用的系统，任何时候都只有一个人连接，那这种长连接方式效率会比短 连接更合适。

 

我的理解：至于我一直不明白的mysql_pconnect何时关闭，我想只有数据库关闭的时候才关闭吧。。。。
mysql_connect()根本不是在程序执行完毕就关闭的，因为加入我第一个次执行程序用mysql_connect连接，第二次将连接去 掉，还是能够查询。说明mysql_connect()根本不是在程序执行完毕就关掉，而是有一个保持时间，这就是所谓的，mysql连接时长。至于mysql_pconnect是一直持续保持连接，没有时长限制，直到数据库关闭连接。mysql_pconnect每次连接时候，会先查找是否有可用连接，原话：当连接的时候本函数将先尝试寻找一个在同一个主机上用同样的用户名和密码已经打开的（持久）连接，如果找到，则返回此连接标识而不打开新连接。那么mysql_connect是否也会在连接时候先查找可用连接呢？通过查找php手册，找到相关。原话：如果用同样的参数第二次调用 mysql_connect()，将不会建立新连接，而将返回已经打开的连接标识。
