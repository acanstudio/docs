IoC_控制反转(Inversion of Control)
====================================

**IoC就是Inversion of Control，即控制反转。**在面向对象的开发中，IoC意味着将你设计好的类交给系统去控制，而不是在你的类内部控制。这称为控制反转。依赖注入和控制反转说的是同一个东西，是一种设计模式，这种设计模式用来减少程序间的耦合。

下面就以php的角度来描述一下这个概念。

假设我们这里有一个类，类里面需要用到数据库连接，按照最最原始的办法，我们可能是这样写这个类的：

<pre>
class example
{
    private $_db;
    public function __construct()
    {
        require_once('/path/to/Db.php');
        $this->_db = new Db($config);
    }

    public function getList()
    {}
}
</pre>

这种实现最大的弊端是，example类和它所依赖的Db类耦合度太高，Db类发生变化了，会影响到example类。为了解决这个问题，工厂模式出现了，我们创建了一个Factory方法，并通过Factory::getDb()方法来获得db组件的实例：

<pre>
class Factory
{
    public static function getDb()
    {}
}

class example
{
    private $_db;
    public function __construct()
    {
        $this->_db = Factory::getDb();
    }

    public function getList()
    {}
}
</pre>

这样的弊端是，虽然example与Db解耦了，但exmaple与Factory又耦合了，工厂类只是帮你把数据库连接信息给包装起来了，虽然当数据库信息发生变化时只要修改Factory::getDb()方法就可以了，但是突然有一天工厂方法需要改名，或者getDb方法需要改名，你又怎么办？一种解决方式是：我们不从example类内部实例化Db组件，我们依靠从外部的注入，什么意思呢？看下面的例子：

<pre>
class example
{
    private $_db;
    public function getList()
    {}

    public function setDb($connection)
    {
        $this->_db = $connection;
    }
}

$example = new example();
$example->setDb(Factory::getDb());//注入db连接
$example->getList();
</pre>

这样一来，example类完全与外部类解除耦合了，你可以看到Db类里面已经没有工厂方法或Db类的身影了。我们通过从外部调用example类的setDb方法，将连接实例直接注入进去。这样example完全不用关心db连接怎么生成的了。这就叫依赖注入，实现不是在代码内部创建依赖关系，而是让其作为一个参数传递，这使得我们的程序更容易维护，降低程序代码的耦合度，实现一种松耦合。

这还没完，我们再假设example类里面除了db还要用到其他外部类，我们通过：

<pre>
$example->setDb(Factory::getDb());//注入db连接
$example->setFile(Factory::getFile());//注入文件处理类
$example->setImage(Factory::getImage());//注入Image处理类
...

// 简便起见，考虑用一个工厂方法

class Factory {
   public static function getExample()
   {
        $example = new example();
        $example->setDb(Factory::getDb());//注入db连接
        $example->setFile(Factory::getFile());//注入文件处理类
        $example->setImage(Factory::getImage());//注入Image处理类
        return $expample;
    }
}


$example=Factory::getExample();
$example->getList();
</pre>

似乎完美了，但是怎么感觉又回到了上面第一次用工厂方法时的场景？这确实不是一个好的解决方案，所以又提出了一个概念：容器，又叫做IoC容器、DI容器。我们本来是通过setXXX方法注入各种类，代码很长，方法很多，虽然可以通过一个工厂方法包装，但是还不是那么爽，好吧，我们不用setXXX方法了，这样也就不用工厂方法二次包装了，那么我们还怎么实现依赖注入呢？ 这里我们引入一个约定：在example类的构造函数里传入一个名为Di $di的参数，如下：

<pre>
class example
{
    private $_di;
    public function __construct(Di &$di)
    {
        $this->_di = $di;
    }

    //通过di容器获取db实例
    pubilc function getList()
    {}
}

$di = new Di();
$di->set("db",function(){
   return new Db("localhost","root","root","test"); 
});
$example = new example($di);
$example->getList();
</pre>

Di就是IoC容器，所谓容器就是存放我们可能会用到的各种类的实例，我们通过$di->set()设置一个名为db的实例，因为是通过回调函数的方式传入的，所以set的时候并不会立即实例化db类，而是当$di->get('db')的时候才会实例化，同样，在设计di类的时候还可以融入单例模式。
这样我们只要在全局范围内申明一个Di类，将所有需要注入的类放到容器里，然后将容器作为构造函数的参数传入到example，即可在example类里面从容器中获取实例。当然也不一定是构造函数，你也可以用一个 setDi(Di $di)的方法来传入Di容器，总之约定是你制定的，你自己清楚就行。

另外，在example里面放入DI对象依赖也是不好的。可以考虑吧所有涉及到实例化对象的地方，都不要用new class()去实例化了。
我们用一个 叫 DI 的容器对象 去实例化所有对象。假设我们现在需要 用ID容器 去实例化 example 对象。

<pre>
class example {
    
    private $_db;
    public function __construct($db)
    {
        $this->_db = $db;
    }

    public function getList(){}
}
</pre>

DI对象有一个方法 ，用途是分析类的构造函数的参数。这里我们分析example的构造函数，发现有一个叫db的参数。DI就会去实例化db对象。然后把实例化的db对象传入example的构造函数。经过这个步骤。example对象就创建出来了， 而且自动传入了 db对象。

过程简单描述如下：
    * 将 db 对象注册到 DI 容器。
    * DI对象分析 example 类。 发现需要一个叫 db 的对象
    * DI查找 db 对象， 发现找到了， 实例化 db 对象。
    * 调用 example 对构造函数，传入 db。 这样 example 就实例化完成了。$example = DI->invoke('example')

