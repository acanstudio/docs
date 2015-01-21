代理模式（Proxy Pattern）
==============================

【意图】
为另一个对象提供一个替身货占位符以控制对这个对象的访问，就是用一个对象来代表另一个对象。提供其他对象一个代理或占位符，来控制该对象的访问权限

【装饰模式结构图】

【装饰模式中主要角色】
抽象构件(Component)角色：定义一个对象接口，以规范准备接收附加职责的对象，从而可以给这些对象动态地添加职责。
具体构件(Concrete Component)角色：定义一个将要接收附加职责的类。
装饰(Decorator)角色：持有一个指向Component对象的指针，并定义一个与Component接口一致的接口。
具体装饰(Concrete Decorator)角色：负责给构件对象增加附加的职责。

【装饰模式的优缺点】
装饰模式的优点:
1、比静态继承更灵活；
2、避免在层次结构高层的类有太多的特征
装饰模式的缺点：
1、使用装饰模式会产生比使用继承关系更多的对象。并且这些对象看上去都很想像，从而使得查错变得困难。

【装饰模式适用场景】
 代理模式（Proxy）根据种类不同，效果也不尽相同：

    远程（Remote）代理：为一个位于不同的地址空间的对象提供一个局域代表对象。这个不同的地址空间可以是在本机器中，也可是在另一台机器中。远程代理又叫做大使（Ambassador）。好处是系统可以将网络的细节隐藏起来，使得客户端不必考虑网络的存在。客户完全可以认为被代理的对象是局域的而不是远程的，而代理对象承担了大部份的网络通讯工作。由于客户可能没有意识到会启动一个耗费时间的远程调用，因此客户没有必要的思想准备。
    虚拟（Virtual）代理（图片延迟加载的例子）：根据需要创建一个资源消耗较大的对象，使得此对象只在需要时才会被真正创建。使用虚拟代理模式的好处就是代理对象可以在必要的时候才将被代理的对象加载；代理可以对加载的过程加以必要的优化。当一个模块的加载十分耗费资源的情况下，虚拟代理的好处就非常明显。
    Copy-on-Write代理：虚拟代理的一种。把复制（克隆）拖延到只有在客户端需要时，才真正采取行动。
    智能引用（Smart Reference）代理：当一个对象被引用时，提供一些额外的操作，比如将对此对象调用的次数记录下来等。


【装饰模式与其它模式】
     代理模式（Proxy）VS 装饰者（Decorator）

     意图：它们都提供间接访问对象层，都保存被调用对象的引用。

     代理模式（Proxy）：为另一个对象提供一个替身或占位符以控制对这个对象的访问，简而言之就是用一个对象来代表另一个对象。

    装饰者（Decorator）：动态地给一个对象添加一些额外的职责，就增加功能来说，Decorator模式比生成子类更为灵活，它避免了类爆炸问题，像装饰者（Decorator），代理模式（Proxy）组成一个对象并提供相同的接口，但代理模式并不关心对象动态职能的增减。

    在代理模式（Proxy）中Subject定义了主要的功能，而且Proxy根据Subject提供功能控制对象的访问权限。在装饰者（Decorator）中Component只是提供了其中的一些功能，需要通过装饰链动态给对象增加职能。
【装饰模式PHP示例】

:: 
 
    <?php
    /**
     * 装饰模式 2010-06-03 sz
     * @author phppan.p#gmail.com
     * http://www.phppan.com 哥学社成员（http://www.blog-brother.com/）
     * @package design pattern
     */
 
    /**
     * 抽象构件角色
     */
    interface Component
    {
        /**
         * 示例方法
         */
        public function operation();
    }
 
    /**
     * 装饰角色
     */
    abstract class Decorator implements Component
    {
        protected  $_component;
 
        public function __construct(Component $component)
	{
            $this->_component = $component;
        }
 
        public function operation()
	{
            $this->_component->operation();
        }
    } 
 
    /**
     * 具体装饰类A
     */
    class ConcreteDecoratorA extends Decorator
    {
        public function __construct(Component $component)
	{
            parent::__construct($component); 
        }
 
        public function operation()
	{
            parent::operation();    //  调用装饰类的操作
            $this->addedOperationA();   //  新增加的操作
        }
 
        /**
         * 新增加的操作A，即装饰上的功能
         */
        public function addedOperationA()
	{
            echo 'Add Operation A <br />';
        }
    }
 
    /**
     * 具体装饰类B
     */
    class ConcreteDecoratorB extends Decorator
    {
        public function __construct(Component $component)
	{
            parent::__construct($component); 
        }
 
        public function operation()
	{
            parent::operation();
            $this->addedOperationB();
        }
 
        /**
         * 新增加的操作B，即装饰上的功能
         */
        public function addedOperationB()
	{
            echo 'Add Operation B <br />';
        }
    }
 
    /**
     * 具体构件
     */
    class ConcreteComponent implements Component
    {
        public function operation()
	{
            echo 'Concrete Component operation <br />';
        } 
    } 
 
    /**
     * 客户端
     */
    class Client
    { 
        /**
         * Main program.
         */
        public static function main() {
            $component = new ConcreteComponent();
            $decoratorA = new ConcreteDecoratorA($component);
            $decoratorB = new ConcreteDecoratorB($decoratorA);
 
            $decoratorA->operation();
            $decoratorB->operation();
        }
    }
 
    Client::main();
    ?>

从以上示例可以看出：
1、装饰类中有一个属性$_component，其数据类型是Component;
2、装饰类实现了Component接口;
3、接口的实现是委派给父类，但并不是单纯的委派，还有功能的增强;
4、具体装饰类实现了抽象装饰类的operation方法。