SPL简介
============

SPL（标准PHP库）是PHP5面向对象功能中最重要的部分。包括迭代器、异常、数组重载、XML以及文件盒数据处理能力；还提供了诸如观察者模式、计数功能、用于对象标示符的辅助函数、迭代器处理功能；此外还有用于自动加载类和接口的高级功能。SPL默认是激活的，SPL的特性在PHP5.2及之后版本中得到很大的扩充。

SPL基础 
----------

SPL是一系列Zend Engine2的附加功能、内部类以及一套PHP示例。在引擎层面，SPL实现了提供所有高级功能的6个类和接口。这些接口和Exception类都具有特殊性，即它们实际上与传统的接口不同。它们拥有特别强大的功能，并且允许引擎以一种特定切特殊的方式挂接进代码中。

    * ArrayAccess：允许创建可以被当做数组的类，类似于其他语言的所引起的；
    * Exception：SPL扩展为Exception内之类提供了一系列增强功能以及更多类型；
    * Iteator：使得对象可以和诸如foreach这样的循环结构一起工作，这个接口要求实现一些列的方法，以便使对象可以符合foreach的语法结构。如返回某项是否存在、取值的顺序等；


no 迭代器
--------------

no 数组重载
--------------

no观察者模式
-------------

序列化
--------------

::
  
    interface Serializable
    {
        public function serialize();
	public function unserialize($serialized);
    }

当实现Serialize()方法时，要求返回代表这个对象的序列化以后的字符串，这个字符串通常是通过调用serialize()方法获得的；unserialize()方法运行重新构造对象，它接受序列化的字符串作为输入。可以结合get_object_vars()组合使用一遍序列化对象的所有成员。Serializable接口提供了一些其他功能，__wakeup魔术方法是在狗仔对象之后才被调用的，和它不同的是，unserialize()方法是一种特殊类型的构造器，通过将构造函数的输入保存在序列化数据中，它提供了正确构造对象的机会，__wakeup方法是在类构造之后才被调用的，它不接受任何输入。Serializable接口提供了大量高级序列化功能，和魔术方法提供的功能相比，它还具有创建更稳定的序列化场景的能力。


SPL的Serializable接口为一些高级的序列化场景提供了支持。非SPL的序列化魔术方法的__sleep和__wakeup有一些问题，魔术方法不能序列化基类的私有变量，实现的__sleep函数必须返回变量名称的一个数组，并且要包含在序列化输出结果中。通过运行调用父类的serialize()方法，Serializabl接口消除了不能序列化基类私有变量的限制。

::

    error_reporting(E_ALL);
    class Base
    {
        private $baseVar;

	public function __construct()
	{
	    $this->baseVar = 'foo';
	}
    }

    class Extender extends Base 
    {
        private $extenderVar;

	public function __construct()
	{
	    parent::__construct();
	    $this->extenderVar = 'bar';
	}

	public function __sleep()
	{
	    return array('extenderVar', 'baseVar');
	}
    }

    $instance = new Extender();
    $serialized = serialize($instance);
    echo $serialized . "\n";
    $restored = unserialize($serialized);

::
   
    error_reporting(E_ALL);
    class Base implements Serializable
    {
        private $baseVar;
	public function __construct()
	{
	    $this->baseVar = 'foo';
	}

	public function serialize()
	{
	    return serialize($this->baseVar);
	}

	public function unserialize($serialized) 
	{
	    $this->baseVar = unserialize($serialized);
	}

	public function printMe()
	{
	    echo $this->baseVar . "\n";
	}
    }

    class Extender extends Base
    {
        private $extenderVar;

	public function __construct()
	{
	    parent::__construct();
	    $this->extenderVar = 'bar';
	}

        public function serialize()
	{
	    $baseSerialized = parent::serialize();
	    return serialize(array($this->extenderVar, $baseSerialized));
	}

	public function unserialize($serialized)
	{
	    $temp = unserialize($serialized);
	    $this->extenderVar = $temp[0];
	    parent::unserialize($temp[1]);
	}
    }

    $instance = new Extender();
    $serialized = serialize($instance);
    echo $serialized . "\n";
    $restored = unserialize($serialized);
    $restored->printMe();



串行化serialize可以把变量包括对象,转化成连续bytes数据. 你可以将串行化后的变量存在一个文件里或在网络上传输. 然后再反串行化还原为原来的数据. 你在反串行化类的对象之前定义的类,PHP可以成功地存储其对象的属性和方法. 有时你可能需要一个对象在反串行化后立即执行. 为了这样的目的,PHP会自动寻找__sleep和__wakeup方法.

 

当一个对象被串行化,PHP会调用__sleep方法(如果存在的话). 在反串行化一个对象后,PHP 会调用__wakeup方法. 这两个方法都不接受参数. __sleep方法必须返回一个数组,包含需要串行化的属性. PHP会抛弃其它属性的值. 如果没有__sleep方法,PHP将保存所有属性.


在程序执行前，serialize() 函数会首先检查是否存在一个魔术方法 __sleep.如果存在，__sleep()方法会先被调用， 然后才执行串行化（序列化）操作。这个功能可以用于清理对象，并返回一个包含对象中所有变量名称的数组。如果该方法不返回任何内容，则NULL被序列化，导致 一个E_NOTICE错误。与之相反，unserialize()会检查是否存在一个__wakeup方法。如果存在，则会先调用 __wakeup方法，预先准备对象数据。

 

__sleep方法常用于提交未提交的数据，或类似的操作。同时，如果你有一些很大的对象， 不需要保存，这个功能就很好用。__wakeup经常用在反序列化操作中，例如重新建立数据库连接，或执行其它初始化操作。

 

<?php
class Connection {
    protected $link;
    private $server, $username, $password, $db;
    
    public function __construct($server, $username, $password, $db)
    {
        $this->server = $server;
        $this->username = $username;
        $this->password = $password;
        $this->db = $db;
        $this->connect();
    }
    
    private function connect()
    {
        $this->link = mysql_connect($this->server, $this->username, $this->password);
        mysql_select_db($this->db, $this->link);
    }
    
    public function __sleep()
    {
        return array('server', 'username', 'password', 'db');
    }
    
    public function __wakeup()
    {
        $this->connect();
    }
}
?>

 

下面例子显示了如何用__sleep和 __wakeup方法来串行化一个对象. Id属性是一个不打算保留在对象中的临时属性. __sleep方法保证在串行化的对象中不包含id属性. 当反串行化一个User对象，__wakeup方法建立id属性的新值. 这个例子被设计成自我保持. 在实际开发中，你可能发现包含资源(如图像或数据流)的对象需要这些方法。

<?php
class user {
    public $name;
    public $id;
    
    function __construct() {    // 给id成员赋一个uniq id 
        $this->id = uniqid();
        }
        
    function __sleep() {       //此处不串行化id成员
        return(array('name'));
        }
        
    function __wakeup() {
        $this->id = uniqid();
        }
    }

$u = new user();

$u->name = "Leo"; 

$s = serialize($u); //serialize串行化对象u，此处不串行化id属性，id值被抛弃

$u2 = unserialize($s); //unserialize反串行化，id值被重新赋值

 

//对象u和u2有不同的id赋值

print_r($u);

print_r($u2);

?>

例三：__wakeup方法的一个缺陷需要注意，如果你打算unserialize一个对象，你

 

<?php 
class A { 
 public $b; 
 public $name; 
} 

class B extends A { 
 public $parent; 
 public function __wakeup() { 
  var_dump($parent->name); 
 } 
} 

$a = new A(); 
$a->name = "foo"; 
$a->b = new B(); 

//我们期望这里输出：foo，但实际在后面的代码执行之后，实际输出NULL.

$a->b->parent = $a; 
$s = serialize($a); 
$a = unserialize($s); 
?> 

原因: $b 对象在$name之前unserialized了. 所以在B::__wakeup执行时, $a->name还没有被赋值

所以，一定要小心你定义类中变量的执行顺序。


noSPL自动加载
---------------

对象标示符
-------------

SPL提供了一个spl_object_hash()函数，该函数为指定对象生成一个唯一标示符，这个标示符可用于作为保存对象或区分不同对象的hash key。返回的字符串对于每一个对象都是唯一的，并且对同一个对象它总是相同的。

::

<?php
$id = spl_object_hash($object);
$storage[$id] = $object;

class Foo{
     private static $test = 0;
     
    public function __construct(){
         self::$test++;
     }
 }
 
$foo1 = new Foo();
 $foo2 = new Foo();
 $foo3 = new Foo();
 
//new unique identifier
 echo spl_object_hash($foo1)."<br />\n";
 //new unique identifier
 echo spl_object_hash($foo2)."<br />\n";
 //new unique identifier
 echo spl_object_hash($foo3)."<br />\n";
 
$foo1 = new Foo();
 //new unique identifier because, when we created Foo, foo1 still existed
 echo spl_object_hash($foo1)."<br />\n";
 
$foo1 = new Foo();
 //same identifier as the first $foo1 because it was destroyed,
 //thus released the identifier
 echo spl_object_hash($foo1)."<br />\n";
 
