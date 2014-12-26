状态模式（State）
=================

【意图】
允许一个对象在其内部状态改变时改变它的行为。对象看起来似乎修改了它的类
状态模式变化的位置在于对象的状态

【状态模式结构图】
状态模式

状态模式

【状态模式中主要角色】
抽象状态(State)角色：定义一个接口，用以封装环境对象的一个特定的状态所对应的行为
具体状态（ConcreteState)角色：每一个具体状态类都实现了环境（Context）的一个状态所对应的行为
环境(Context)角色：定义客户端所感兴趣的接口，并且保留一个具体状态类的实例。这个具体状态类的实例给出此环境对象的现有状态

【状态模式的优点和缺点】
1、它将与特定状态相关的行为局部化
2、它使得状态转换显示化
3、State对象可被共享

【状态模式适用场景】
1、一个对象的行为取决于它的状态，并且它必须在运行时刻根据状态改变它的行为
2、一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态。这个状态通常用一个或多个枚举常量表示。通常，有多个操作包含这一相同的条件结构。State模式模式将每一个条件分支放入一个独立的类中。这使得你可以要所对象自身的情况将对象的状态作为一个对象，这一对象可以不依赖于其他对象而独立变化

【状态模式与其它模式】
单例模式(singleton模式)：具体状态对象通常是单例模式
享元模式(flyweight模式)：享元模式解释了何时以及怎样共享状态对象

【状态模式PHP示例】

::

    <?php 
    /**
     * 状态模式的PHP简单实现 2010-07-25 sz
     * @author 胖子 phppan.p#gmail.com  http://www.phppan.com                                                     
     * 哥学社成员（http://www.blog-brother.com/）
     * @package design pattern
     */
 
    /**
     * 抽象状态角色
     */
    interface State
    {
        /**
         * 方法示例
         */
        public function handle(Context $context);
    }
 
    /**
     * 具体状态角色A
     * 单例类
     */
    class ConcreteStateA implements State
    {
        /* 唯一的实例 */
        private static $_instance = null;
   
        private function __construct(){}
 
        /**
         * 静态工厂方法，返还此类的唯一实例
         */
        public static function getInstance()
	{
            if (is_null(self::$_instance)) {
                self::$_instance = new ConcreteStateA();
            }
 
            return self::$_instance;
        }
 
        public function handle(Context $context)
	{
            echo 'Concrete Sate A handle method<br />';
            $context->setState(ConcreteStateB::getInstance());
        }
    }
 
    /**
     * 具体状态角色B
     * 单例类
     */
    class ConcreteStateB implements State
    {
        /* 唯一的实例 */
        private static $_instance = null;
 
        private function __construct(){}
 
        /**
         * 静态工厂方法，返还此类的唯一实例
         */
        public static function getInstance()
	{
            if (is_null(self::$_instance)) {
                self::$_instance = new ConcreteStateB();
            }
 
            return self::$_instance;
        }
 
        public function handle(Context $context)
	{
            echo 'Concrete Sate B handle method<br />';
            $context->setState(ConcreteStateA::getInstance());
        }
    }
 
    /**
     * 环境角色
     */
    class Context
    {
        private $_state;
 
        /**
         * 默认为StateA
         */
        public function __construct()
        {
            $this->_state = ConcreteStateA::getInstance();
        }
 
        public function setState(State $state)
        {
            $this->_state = $state;
        }
 
        public function request()
        {
            $this->_state->handle($this);
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
        public static function main()
        {
            $context = new Context();
            $context->request();
            $context->request();
            $context->request();
            $context->request();
        }
    }
 
    Client::main();
    ?>