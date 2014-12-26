PHP中的SESSION和COOKIE
=============================

什么是COOKIE
-------------------

在互联网技术领域，COOKIE指的是小量信息，由网络服务器发送出来以存储在网络浏览器上，从而下次这位独一无二的访客又回到该网络服务器时，可从该浏览器读回此信息。这是很有用的，让浏览器记住这位访客的特定信息，如上次访问的位置、花费的时间或用户首选项（如样式表）。COOKIE 是个存储在浏览器目录的文本文件，当浏览器运行时，存储在 RAM 中。一旦客户端用户从该网站或网络服务器退出，COOKIE 也可存储在计算机的硬盘上。有一点需要说明的是：一般情况下COOKIE由服务器端生成（当然在客户端也可以生成并操作COOKIE），为了进行辨别用户身份、session跟踪而储存在用户本地终端上的数据（通常经过加密），当用户下次请求同一网站时就发送该COOKIE给服务器（前提是浏览器启用了COOKIE）。因此，COOKIE就是储存在用户本地终端上的一小量数据。

COOKIE的工作原理
---------------------

首先，服务器端在响应中利用setcookie、header来创建一个COOKIE ，然后，浏览器在它的请求中通过COOKIE header包含这个已经创建的COOKIE，并且反它返回至服务器，从而完成浏览器的论证。
举个例来说明：我们创建了一个名字为login的COOKIE来包含访问者的信息。如：Set-Cookie:login=Michael Jordan;path=/;domain=msn.com;expires=Monday,01-Mar-99 00:00:01 GMT 这里假设访问者的注册名是“Michael Jordan”，同时还对所创建的COOKIE的属性如path、domain、expires等进行了指定。

上面这个Header会自动在浏览器端计算机的COOKIE文件中添加一条记录。浏览器将变量名为“login”的COOKIE赋值为“Michael Jordon”。当再次请求同一网站时（MSN.COM），游览器会将此相应的COOKIE随头部信息一起返还给相应的服务器。

PHP如何设置COOKIE
-------------------

PHP用setcookie函数来设置COOKIE。必须注意的一点是：COOKIE是HTTP协议头的一部分，用于浏览器和服务器之间传递信息，所以必须在任何属于HTML文件本身的内容输出之前调用COOKIE函数。

PHP的setcookie函数：定义了一个COOKIE，并且把它附加在HTTP头的后面，setcookie函数的原型如下：int setcookie(string name, string value, int expire, string path, string domain, int secure); 
说明：除了name之外所有的参数都是可选的；value,path,domain 三个参数可以用空字符串代换，表示没有设置；expire和 secure两个参数是数值型的，可以用0表示；expire参数是一个标准的Unix时间标记，可以用time()或mktime() 函数取得，以秒为单位；secure参数表示这个COOKIE是否通过加密的HTTPS协议在网络上传输。

    * 当前设置的COOKIE 不是立即生效的，而是要等到下一个页面时才能看到，这是由于在设置的这个页面里COOKIE由服务器传递给客户浏览器，在下一个页面浏览器才能把COOKIE从客户的机器里取出传回服务器的原因。
    * 在同一个页面设置COOKIE，实际是从后往前，所以如果要在插入一个新的COOKIE之前删掉一个，你必须先写插入的语句，再写删除的语句，否则可能会出现不希望的结果。
    * 如果你的站点有几个不同的目录，那么如果只用不带路径的COOKIE的话，在一个目录下的页面里设的COOKIE在另一个目录的页面里是看不到的，也就是说，COOKIE是面向路径的。实际上，即使没有指定路径，WEB 服务器会自动传递当前的路径给浏览器的，指定路径会强制服务器使用设置的路径。解决这个问题的办法是在调用setcookie时加上路径和域名，域名的格式可以是“www.phpuser.com”，也可是“.phpuser.com”。
    * 不同浏览器，COOKIE所存放的位置是不一样的。
    * setcooke函数里表示value的部分，在传递时会自动被encode，也就是说，如果value的值是“test value”在传递时就变成了“test%20value”，跟URL的方法一样。当然，对于程序来说这是透明的，因为在PHP接收COOKIE的值时会自动将其decode。

COOKIE周期问题
----------------------

默认情况下，即不指定过期时间的情况下，如setcookie('mycookie', 'value'); 这种没有设置任何时间的，只在浏览没有关闭时，有效，一旦关闭浏览器，COOKIE的值就丢失。

可以在生成COOKIE是指定过期时间，从而告诉浏览器，该COOKIE已过期多长时间了，如：
setcookie("WithExpire", "Expire in 1 hour", time()-3600);//已过期1小时了。
setcookie("WithExpire",'',time()-1);或者：setcookie("WithExpire",'',0); (注：在当前页，不会立即看到效果)

可以通过下面的方法，让COOKIE的值在关闭浏览器后，仍然存在一段时间：
setcookie("FullCOOKIE", "Full COOKIE value", time()+3600, "/forum", ".phpuser.com", 1); 这种只会存在1小时，就算浏览器关闭了，COOKIE的值也会一直存在1小时，1小时后，它就丢失。

还可以设置数组形式的COOKIE：如：setcookie("COOKIEArray[]", "Value 1"); setcookie("COOKIEArray[]", "Value 2"); 

读取COOKIE：PHP通过全局变量COOKIE接收所有的COOKIE信息，处理是完全透明，自动的。如：$COOKIEone = $_COOKIE['MyCOOKIE'];

删除COOKIE：
    * 调用只带有name参数的setcookie，那么名为这个name的COOKIE 将被从关系户机上删掉；
    * 设置COOKIE的失效时间为time()或time()-1，那么这个COOKIE在这个页面的浏览完之后就被删除了（其实是失效了）。
    * 在客户端手动删除（操作浏览器）。要注意的是，当一个COOKIE被删除时，它的值在当前页在仍然有效的。


PHP SESSION
一 到底什么是SESSION？
    Session:在计算机中，尤其是在网络应用中，称为“会话”。
具体到Web中的Session指的就是用户在浏览某个网站时，从进入网站到浏览器关闭所经过的这段时间，也就是用户浏览这个网站所花费的时间。因此从上述的定义中我们可以看到，Session实际上是一个特定的时间概念。
Session解决方案，就是要提供在PHP脚本中定义全局变量的方法，使得这个全局变量在同一个Session中对于所有的PHP脚本都有效。
Session 的作用就是它在 Web 服务器上保持用户的状态信息供在任何时间从任何页访问。因为浏览器不需要存储任何这种信息，所以可以使用任何浏览器，即使是像 PDA 或手机这样的浏览器设备。

二 PHP中如何使用SESSION呢？
Session 是存储在服务器端的，远程用户没办法修改 session 文件的内容，因此我们可以单纯存储一个 $admin 变量来判断是否登陆，首次验证通过后设置 $admin 值为 true，以后判断该值是否为 true，假如不是，转入登陆界面，这样就可以减少很多数据库操作了。
创建与销毁：
启动 session 会话，并创建一个 $admin 变量：
// 启动 session
session_start(); //一启动时
//注意:在session_start()之前是不能有任何输出的。
原因：一启动SESSION时，它会向HTTP header里写入一些东西，而HTTP HEADER面包含有很多规则，这些规则是规定HTML以何种方式显示所以规定在header到达浏览器前不可以有HTML内容输出
// 声明一个名为 admin 的变量，并赋空值。
$_session["admin"] = null;

简单的一个例子：
// 判断是否登陆
if (isset($_SESSION["admin"]) && $_session["admin"] === true)
{
echo "您已经成功登陆";
}
else
{
// 验证失败，将 $_session["admin"] 置为 false
$_session["admin"] = false;
die("您无权访问");
}
如果要登出系统怎么办?销毁 session 即可。
session_start();
// 这种方法是将原来注册的某个变量销毁
unset($_session["admin"]);
// 这种方法是销毁整个 session 文件
session_destroy();

三 PHP SESSION的周期问题。
实际上，服务用SESSION来判断是否是同一个用户时，是通过 Session ID 来判断的，什么是 Session ID，就是那个 Session 文件的文件名，Session ID 是随机生成的，因此能保证唯一性和随机性，确保 Session 的安全。一般如果没有设置 Session 的生存周期，则 Session ID 存储在内存中，关闭浏览器后该 ID 自动注销，重新请求该页面后，重新注册一个 session ID。
所以：保存SESSION就是保存SESSION ID，但问题是比你想象的要麻烦一点。

结合COOKIE来保存SESSION：
如果客户端没有禁用 COOKIE，则 COOKIE 在启动 Session 会话的时候扮演的是存储 Session ID 和 session 生存期的角色。
我们来手动设置 session 的生存期：
session_start();
// 保存一天
$lifeTime = 24 * 3600;
setCOOKIE(session_name(), session_id(), time() + $lifeTime, "/");
（另特别注明：实际上一旦启动SESSION后，服务就会在产生一个SESSION ID 放在HTTP HEADER里面发送给客户端，不过只在浏览没有关闭时有效）
函数设置：
其实 Session 还提供了一个函数 session_set_COOKIE_params(); 来设置 Session 的生存期的，该函数必须在 session_start() 函数调用之前调用：
// 保存一天
$lifeTime = 24 * 3600;
session_set_COOKIE_params($lifeTime);
session_start();
$_session["admin"] = true;
客户端禁用COOKIE后怎么办呢？
如果禁用了COOKIE后，实际SESSION也就是暂时没有办法使用了，所有生存周期都是浏览器进程了（但是SESSION文件仍然是存在于服务器上 的），只要关闭浏览器，再次请求页面又得重新注册 Session。但是可以在没有关闭浏览器时，传递SESSION ID。通过 URL 或者通过隐藏表单来传递，PHP 会自动将 session ID 发送到 URL 上，URL 形如：http://www.openphp.cn/index.php?PHPSESSID=bba5b2a240a77e5b44cfa01d49cf9669，其中 URL 中的参数 PHPSESSID 就是 Session ID了，我们可以使用 $_GET 来获取该值，从而实现 session ID 页面间传递。
// 保存一天
$lifeTime = 24 * 3600;
// 取得当前 session 名，默认为 PHPSESSID
$sessionName = session_name();
// 取得 session ID
$sessionID = $_GET[$sessionName];
// 使用 session_id() 设置获得的 session ID
session_id($sessionID);
session_set_COOKIE_params($lifeTime);
session_start(); //以上设置均在SESSION_START之前完成
$_session["admin"] = true

(3)存放路径的设置：
所有用户的 Session 都保存在系统临时文件夹里，将给维护造成困难，而且降低了安全性，我们可以手动设置 Session 文件的保存路径，session_save_path()就提供了这样一个功能。我们可以将 session 存放目录指向一个不能通过 Web 方式访问的文件夹，当然，该文件夹必须具备可读写属性。
// 设置一个存放目录
$savePath = "./session_save_dir/";
// 保存一天
$lifeTime = 24 * 3600;
session_save_path($savePath);
session_set_COOKIE_params($lifeTime);
session_start();
$_session["admin"] = true;
?>
同 session_set_COOKIE_params(); 函数一样，session_save_path() 函数也必须在 session_start() 函数调用之前调用。

    PHP SESSION操作的高级部分：

使用用户自定级别的SESSION存储函数：
session_set_save_handler ( callback $open , callback $close , callback $read , callback $write , callback $destroy , callback $gc )
这个通常是很有用的，让你自定函数来存储，处理SESSION。通知php使用自定义的session处理函数来操作session，而不使用php预置的方法，不过，session_set_save_handler必须在session_start之前。

session_set_save_handler("open", "close", "read", "write", "destroy", "gc");
session_start();
说明：
open: Open function, this works like a constructor in classes and is executed when the session is being opened. The open function expects two parameters, where the first is the save path and the second is the session name.
即：session_start()调用

   close: Close function, this works like a destructor in classes and is executed when the session operation is done.
即：程序结束时调用

         read:session_start()调用
write：程序结束时调用
sess_destroy：session_destroy()调用
sess_gc：操作系统gc进程调用

                什么是会话，会话处理

 

         可能看到这里，有朋友会说“你在上面不是讨论了【会话】了吗？怎么又再问呢?”。确实，我也很迷惑，为什么呢？

        上文提到“Session:在计算机中，尤其是在网络应用中，称为“会话”。”，也就是我们通俗的把SESSION叫为“会话”。另外我们已知道，HTTP是一种无状态的协议，所以当要创建更好，复杂的WEB应用程序时，就比较麻烦了。因此，出现了最开始我提到的COOKIE技术，但是COOKIE大小的限制，以及允许的COOKIE数量实现上的各种不一致，等等这些原因促使开发人员提出更好的解决方案：会话处理。

           会话处理的实现方式是为每位网站访问者分配一个称为会话ID（SID,即SESSION ID）的唯一标识属性，然后将此SID与任意数量的数据关联，如果按关系数据库的术语来说，可以将SID视为将其他所有用户属性捆绑在一起的主键。可问题是：怎么把这个SID持续地与某个用户关联呢？两种方案：

   1 URL传递(即客户端禁用COOKIE后，传递SID的方式)

              例 ：
session.php页面代码如下：
<?php
session_id('12345678');   //(1)手动设置定 SESSION ID(这个设定必须在SESSION_START之前)
session_start();   //(2) 开始会话
$current_id = session_id(); //(3) 获得设定的SESSION ID值
var_dump($current_id); //(4) 打印看看(实际上，如果你没有手动设置SESSIONID的话，你不断刷新页面，此处的值也是在断变化的)
$_SESSION['test'] = 'not found?'; //(5) 设置一个SESSION 全局变量
echo '<a href=getCOOKIE.php?id='.$current_id.'>getCOOKIE</a>'; //（6） 传递到另一个页

getCOOKIE.php页面代码为：
if(isset($_GET['id'])){
$ID = $_GET['id'];
}else{
echo '没有传递SID';
session_start();
var_dump($_SESSION); //没有传递SID时，我们也打印看看
exit;
}
session_id($ID); //再次说明：SESSION_ID手动设置SID时，必须在SESSION_START之前
session_start();
var_dump($_SESSION);

         运行代码方式： 1 从SESSION.php页面链接到getCOOKIE页，我们看到可以打印出SESSION里存的变量以及其值。
2 如果我们直接运行getCOOKIE页，则我们会看打印的结果是为空。

2 COOKIE存储
session.php页面代码如下：
<?php
session_start();
$current_id = session_id();
var_dump($current_id);
$_SESSION['testtwo'] = '第二次测试';

getCOOKIE.php页面代码为：
<?php
session_start();
var_dump($_COOKIE);
var_dump($_SESSION);

这样：存在COOKIE中时，PHP帮我们自动的存储与获得SESSION ID了，所以可以易得到原注册的变量。

另外有一点是要特别说明的：
就算是用户没有禁用COOKIE，如果用户单开个浏览器窗口(即：不同的进程，意思说不用选项卡来打开，而是完全单独的打开一个窗口)，也是得不到刚注册的变量的.
1 如果你是连续用选项卡方式打开getCOOKIE.php页，那么你会正常得到注册的变量以及其值。（在一个进程里打开）

2 如果你是单独的新开一个窗口来执行getCOOKIE.php页，那么你就得不到session.php页里注册的变量了。（在多个进程里打开）

所以要做的十全十美，还得另考虑：
详见：《如何完全抛开COOKIE使用session》http://hi.baidu.com/fc_lamp/blog/item