在PHP中使用PDO
================

PDO(PHP Data Object) 是PHP 5新出来的东西，在PHP 6都要出来的时候，PHP 6只默认使用PDO来处理数据库，将把所有的数据库扩展移到了PECL，那么默认就是没有了常用的php_mysql.dll扩展了。因此使用PDO是大势所趋的了。

以MySQL为例，PHP连接MySQL可以通过扩展模块php_mysql.so或php_mysqli.so，也可以通过PDO来链接MySQL。php_mysql.so在新版本中将要淘汰，而MySQLi的i代表Improvement，提供了相对更高级的一些功能，就Extension而言，本身也增加了安全性。而PDO则是提供了一个Abstraction Layer来操作数据库。

首先，先来看一段用 MySQL 编写的代码：

<pre>
<?php  
mysql_connect($dbHost, $dbUser, $dbPassword);  
mysql_select_db($dbName);  
$result = mysql_query("SELECT `field` FROM `table` WHERE `field` = '$value'");  
while ($row = mysql_fetch_array($result, MYSQL_ASSOC)) {
    echo $row['field'];  
}  
mysql_free_result($result);  
?>  
</pre>
 
乍看之下没有什么问题，其实背后还是有些学问的。

这种方式不能Bind Column，$location的地方容易被SQL注入。于是后来发展出了mysql_escape_string()（备注：5.3.0 之后弃用）以及mysql_real_escape_string()来解决这个问题，不过这么一搞，整个过程会变得复杂丑陋，而且如果参数多了，可以想象会是怎样的情形……

<pre>
<?php  
$query = sprintf(
    "SELECT * FROM users WHERE user='%s' AND password='%s'",  
    mysql_real_escape_string($user),  
    mysql_real_escape_string($password)
);  
mysql_query($query);  
?>  
</pre>
 
在 MySQLi中有了不少进步，除了通过Bind Column来解决上述问题，我们可以对比一下Object oriented style（下面这段MySQLi范例的写法）和Procedural style上面MySQL范例的写法）两种写法的差别。

<pre>
<?php  
$mysqli = new mysqli($db_host, $db_user, $db_password, $db_name);  

$sql = "INSERT INTO `users` (id, name, gender, location) VALUES (?, ?, ?, ?)";  
$stmt = $mysqli->prepare($sql);  
$stmt->bind_param('dsss', $source_id, $source_name, $source_gender, $source_location);  
$stmt->execute();  
$stmt->bind_result($id, $name, $gender, $location);  
while ($stmt->fetch()) {  
    echo $id . $name . $gender . $location;  
}  

$stmt->close();  
$mysqli->close();  
?>  
</pre>
 
又发现了一些缺点，例如得Bind Result，这个就有点多余，不过这其实无关紧要，因为最大的问题还是在于这不是一个抽象（Abstraction）的方法，所以当后端更换数据库的时候，就是痛苦的开始了。

于是PDO就出现了，PDO和PDO_MySQL需要PHP环境支持，具体安装自行参考相关资料，这里不展开描述。

<pre>
<?php  
$dsn = "mysql:host=$db_host;dbname=$db_name";  
$dbh = new PDO($dsn, $db_user, $db_password);  

$sql = "SELECT `name`, `location` FROM `users` WHERE `location` = ? , `name` = ?";  
$sth = $dbh->prepare($sql);  
$sth->execute(array($location, $name));  
$result = $sth->fetch(PDO::FETCH_OBJ);  

echo $result->name . $result->location;  
$dbh = NULL;
?>  
</pre>

 
乍看之下，PDO 的代码好像也没有变短，那到底好处是什么呢？
    * PDO 连接数据库时通过Connection String来决定连接何种数据库。
    * PDO 可以通过 PDO::setAttribute 来决定连接时的设定，比如Persistent Connection，回传错误的方式（Exception，E_WARNING，NULL）。甚至是回传参数名称的大小写等等。
    * PDO 支持Bind Column的功能，除了基本的Prepare，Execute以外，也可以Bind 单一参数，并且指定参数类型。
    * PDO 是Abstraction Layer，所以就算更换储存媒介，需要花的功夫比起来是最少的。


PDOStatement::bindParam — 绑定一个参数到指定的变量名。

绑定一个PHP变量到用作预处理的SQL语句中的对应命名占位符或问号占位符。 不同于 PDOStatement::bindValue() ，此变量作为引用被绑定，并只在 PDOStatement::execute()被调用的时候才取其值。

PDOStatement::bindValue — 把一个值绑定到一个参数。

绑定一个值到用作预处理的 SQL 语句中的对应命名占位符或问号占位符。

<pre>
<?php
$stm = $pdo->prepare("select * from users where user = :user");
$user = "jack";
//正确
$stm->bindParam(":user",$user);
//错误
$stm->bindParam(":user","jack");
//正确
$stm->bindValue(":user",$user);
//正确
$stm->bindValue(":user","jack");
 
//所以使用bindParam是第二个参数只能用变量名，而不能用变量值，而bindValue至可以使用具体值。
?>
</pre>

PDOStatement::bindColumn — 绑定一列到一个 PHP 变量。

安排一个特定的变量绑定到一个查询结果集中给定的列。每次调用 PDOStatement::fetch() 或 PDOStatement::fetchAll() 都将更新所有绑定到列的变量。

<pre>
<?php
function  readData ( $dbh ) {
    $sql  =  'SELECT name, colour, calories FROM fruit' ;
    try {
        $stmt  =  $dbh -> prepare ( $sql );
        $stmt -> execute ();
 
        /*  通过列号绑定  */
        $stmt -> bindColumn ( 1 ,  $name );
        $stmt -> bindColumn ( 2 ,  $colour );
 
        /*  通过列名绑定  */
        $stmt -> bindColumn ( 'calories' ,  $cals );
 
        while ( $row  =  $stmt -> fetch ( PDO :: FETCH_BOUND )) {
            $data  =  $name  .  "\t"  .  $colour  .  "\t"  .  $cals  .  "\n" ;
            print  $data ;
        }
    }
    catch ( PDOException $e ) {
        print  $e -> getMessage ();
    }
}
readData ( $dbh );
?>
</pre>




使用传统的 mysql_connect 、mysql_query方法来连接查询数据库时，如果过滤不严紧，就有SQL注入风险。虽然可以用mysql_real_escape_string()函数过滤用户提交的值，但是也有缺陷。而使用PHP的PDO扩展的 prepare 方法，就可以避免sql injection 风险。

PDO(PHP Data Object) 是PHP5新加入的一个重大功能，因为在PHP 5以前的php4/php3都是一堆的数据库扩展来跟各个数据库的连接和处理，如 php_mysql.dll。 PHP6中也将默认使用PDO的方式连接，mysql扩展将被作为辅助 。官方地址：http://php.net/manual/en/book.pdo.php


1. PDO配置

使用PDO扩展之前，先要启用这个扩展，php.ini中，去掉"extension=php_pdo.dll"前面的";"号，若要连接数据库，还需要去掉与PDO相关的数据库扩展前面的";"号（一般用的是php_pdo_mysql.dll），然后重启Apache服务器即可。

[plain] view plaincopy在CODE上查看代码片派生到我的代码片

    extension=php_pdo.dll   
    extension=php_pdo_mysql.dll  


2. PDO连接mysql数据库

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    $dbh = new PDO("mysql:host=localhost;dbname=mydb","root","password");  

默认不是长连接，若要使用数据库长连接，可以在最后加如下参数：

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    $dbh = new PDO("mysql:host=localhost;dbname=mydb","root","password","array(PDO::ATTR_PERSISTENT => true) ");   
    $dbh = null; //(释放)  


3. PDO设置属性

PDO有三种错误处理方式：

PDO::ERrmODE_SILENT不显示错误信息，只设置错误码

PDO::ERrmODE_WARNING显示警告错

PDO::ERrmODE_EXCEPTION抛出异常

可通过以下语句来设置错误处理方式为抛出异常

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    $db->setAttribute(PDO::ATTR_ERrmODE, PDO::ERrmODE_EXCEPTION);  

因为不同数据库对返回的字段名称大小写处理不同，所以PDO提供了PDO::ATTR_CASE设置项（包括PDO::CASE_LOWER，PDO::CASE_NATURAL，PDO::CASE_UPPER），来确定返回的字段名称的大小写。

通过设置PDO::ATTR_ORACLE_NULLS类型（包括PDO::NULL_NATURAL，PDO::NULL_EmpTY_STRING，PDO::NULL_TO_STRING）来指定数据库返回的NULL值在php中对应的数值。


4. PDO常用方法及其应用

PDO::query() 主要是用于有记录结果返回的操作，特别是SELECT操作

PDO::exec() 主要是针对没有结果集合返回的操作，如INSERT、UPDATE等操作

PDO::prepare() 主要是预处理操作，需要通过$rs->execute()来执行预处理里面的SQL语句，这个方法可以绑定参数，功能比较强大（防止sql注入就靠这个）

PDO::lastInsertId() 返回上次插入操作，主键列类型是自增的最后的自增ID 

PDOStatement::fetch() 是用来获取一条记录

PDOStatement::fetchAll() 是获取所有记录集到一个集合

PDOStatement::fetchColumn() 是获取结果指定第一条记录的某个字段，缺省是第一个字段

PDOStatement::rowCount() :主要是用于PDO::query()和PDO::prepare()进行DELETE、INSERT、UPDATE操作影响的结果集，对PDO::exec()方法和SELECT操作无效。


5.PDO操作MYSQL数据库实例

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php   
    $pdo = new PDO("mysql:host=localhost;dbname=mydb","root","");   
    if($pdo -> exec("insert into mytable(name,content) values('fdipzone','123456')")){   
    echo "insert success";   
    echo $pdo -> lastinsertid();   
    }   
    ?>  

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php   
    $pdo = new PDO("mysql:host=localhost;dbname=mydb","root","");   
    $rs = $pdo -> query("select * from table");   
    $rs->setFetchMode(PDO::FETCH_ASSOC); //关联数组形式  
    //$rs->setFetchMode(PDO::FETCH_NUM); //数字索引数组形式  
    while($row = $rs -> fetch()){   
        print_r($row);   
    }   
    ?>   

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php  
    foreach( $db->query( "SELECT * FROM table" ) as $row )  
    {  
        print_r( $row );  
    }  
    ?>  

统计有多少行数据：

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php  
    $sql="select count(*) from table";  
    $num = $dbh->query($sql)->fetchColumn();  
    ?>  

prepare方式：

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php  
    $query = $dbh->prepare("select * from table");  
    if ($query->execute()) {  
        while ($row = $query->fetch()) {  
            print_r($row);  
        }  
    }  
    ?>  

prepare参数化查询：

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php  
    $query = $dbh->prepare("select * from table where id = ?");  
    if ($query->execute(array(1000))) {   
        while ($row = $query->fetch(PDO::FETCH_ASSOC)) {  
            print_r($row);  
        }  
    }  
    ?>  

使用PDO访问MySQL数据库时，真正的real prepared statements 默认情况下是不使用的。为了解决这个问题，你必须禁用 prepared statements的仿真效果。下面是使用PDO创建链接的例子：
[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php  
    $dbh = new PDO('mysql:dbname=mydb;host=127.0.0.1;charset=utf8', 'root', 'pass');  
    $dbh->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);  
    ?>  

setAttribute()这一行是强制性的，它会告诉 PDO 禁用模拟预处理语句，并使用 real parepared statements 。这可以确保SQL语句和相应的值在传递到mysql服务器之前是不会被PHP解析的（禁止了所有可能的恶意SQL注入攻击）。

虽然你可以配置文件中设置字符集的属性(charset=utf8)，但是需要格外注意的是，老版本的 PHP（ < 5.3.6）在DSN中是忽略字符参数的。


完整的代码使用实例：

[php] view plaincopy在CODE上查看代码片派生到我的代码片

    <?php  
    $dbh = new PDO("mysql:host=localhost; dbname=mydb", "root", "pass");  
    $dbh->setAttribute(PDO::ATTR_EMULATE_PREPARES, false); //禁用prepared statements的仿真效果  
    $dbh->exec("set names 'utf8'");   
    $sql="select * from table where username = ? and password = ?";  
    $query = $dbh->prepare($sql);   
    $exeres = $query->execute(array($username, $pass));   
    if ($exeres) {   
        while ($row = $query->fetch(PDO::FETCH_ASSOC)) {  
            print_r($row);  
        }  
    }  
    $dbh = null;  
    ?>  

上面这段代码就可以防范sql注入。为什么呢？

当调用 prepare() 时，查询语句已经发送给了数据库服务器，此时只有占位符 ? 发送过去，没有用户提交的数据；当调用到 execute()时，用户提交过来的值才会传送给数据库，它们是分开传送的，两者独立的，SQL攻击者没有一点机会。


但是我们需要注意的是以下几种情况，PDO并不能帮助你防范SQL注入。

不能让占位符 ? 代替一组值，这样只会获取到这组数据的第一个值，如：

[sql] view plaincopy在CODE上查看代码片派生到我的代码片

    select * from table where userid in ( ? );  

如果要用in來查找，可以改用find_in_set()实现

[sql] view plaincopy在CODE上查看代码片派生到我的代码片

    $ids = '1,2,3,4,5,6';  
    select * from table where find_in_set(userid, ?);  

不能让占位符代替数据表名或列名，如：

[sql] view plaincopy在CODE上查看代码片派生到我的代码片

    select * from table order by ?;  

不能让占位符 ? 代替任何其他SQL语法，如：

[sql] view plaincopy在CODE上查看代码片派生到我的代码片

    select extract( ? from addtime) as mytime from table;  




