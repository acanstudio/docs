单例模式和工厂模式
===================

模式对OOP开发人员尤其有用，因为它有助于创建稳定的API，并且仍然保持一定的灵活性。一种模式可以帮助我们定义负责完成特定任务的对象，还可以允许我们全部修改掉某个类而不用修改与这个类打交道的代码，前者被称为类的职责，后者被称为类的多态性。

职责和单例模式 
------------------------

在OOP中，一个对象只负责一个特定的任务。单例模式被认为是职责模式，这是因为它将创建对象的控制权委托到一个单一的访问点上，在任何时候，应用程序中都只会有这个类仅有的一个实例存在，这可以防止我们去打开数据库的多个连接或者不必要地使用多余的系统资源，单利模式在维持应用状态的同步方面也尤其有用。
    * 它们必须拥有一个构造函数，并且标记为private；
    * 它们拥有一个保存类的实例的静态成员变量；
    * 它们拥有一个访问这个实例的公共静态方法；
    * 它们拥有一个空的私有的__clone()；

<pre>
class Database
{
    private $_db;
    static $_instance;

    private function __construct()
    {
        $this->_db = pg_connect('dbname=example_db');
    }

    private __clone() {}

    public static function getInstance()
    {
        if (!(self::$_instance instance of self)) {
            self::$_instance = new self();
        }

        return self::$_instance;
    }

    public function quire() {}
}


$db = Database::getInstance();
$db->query($sql);
</pre>

如果某个类不需要使用__construct方法，那么这个类就不太适合使用单例模式，这种情况可以使用纯静态类，只需要提供一个没有函数体的私有构造函数，并且去掉getInstance()和$_instance成员。

工厂模式
-----------

工厂类时指包含了一个专门用来创建其他对象的方法的类，工厂类在多态性编程实践中是至关重要的，它允许动态地替换类，修改配置，并且通常会使应用程序更加灵活。

工厂模式通常用来返回符合类似接口的不同的类，常见的用法就是创建多态的提供者，从而允许我们基于应用程序逻辑或者配置设置类决定应该实例化哪一个类。如

<pre>
// 对象从工厂类返回
class MyObject
{
}

class MyFactory
{
    public static function factory()
    {
        return new MyObject();
    }
}

$instance = MyFactory::factory();
</pre>

