命令模式（Command Pattern）
==============================

作用：将一个请求封装为一个对象，从而可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤消的操作。可变的方面是:何时，怎样满足一个请求；命令模式是对命令的封装。命令模式把发出命令的责任和执行命令的责任分割开，委派给不同的对象。请求的一方发出请求要求执行一个操作；接收的一方收到请求，并执行操作。命令模式允许请求的一方和接收的一方独立开来，使得请求的一方不必知道接收请求的一方的接口，更不必知道请求是怎么被接收，以及操作是否被执行、何时被执行，以及是怎么被执行的。

命令模式结构图

.. image:: /img/Command.jpg

命令模式中主要角色

    * 命令（Command）角色：声明了一个给所有具体命令类的抽象接口。这是一个抽象角色。
    * 具体命令（ConcreteCommand）角色：定义一个接受者和行为之间的弱耦合；实现Execute()方法，负责调用接收考的相应操作。Execute()方法通常叫做执行方法。
    * 客户（Client）角色：创建了一个具体命令(ConcreteCommand)对象并确定其接收者。
    * 请求者（Invoker）角色：负责调用命令对象执行请求，相关的方法叫做行动方法。
    * 接收者（Receiver）角色：负责具体实施和执行一个请求。任何一个类都可以成为接收者，实施和执行请求的方法叫做行动方法。

命令模式的优点

    * 命令模式把请求一个操作的对象与知道怎么执行一个操作的对象分离开。
    * 命令类与其他任何别的类一样，可以修改和推广。
    * 可以把命令对象聚合在一起，合成为合成命令。
    * 可以很容易的加入新的命令类。

命令模式的缺点：可能会导致某些系统有过多的具体命令类。

命令模式适用场景

    * 抽象出待执行的动作以参数化对象。Command模式是回调机制的一个面向对象的替代品。
    * 在不同的时刻指定、排列和执行请求。
    * 支持取消操作。
    * 支持修改日志。
    * 用构建在原语操作上的高层操作构造一个系统。Command模式提供了对事务进行建模的方法。Command有一个公共的接口，使得你可以用同一种方式调用所有的事务。同时使用该模式也易于添加新事务以扩展系统。

命令模式与其它模式比较

    * 合成模式(composite模式):Composite模式可被实现宏命令
    * 原型模式(prototype模式)：如果命令类带有clone(或在之前的文章中提到的copy方法)方法，命令就可以被复制。在命令模式支持多次取消操作时可能需要用到此模式，以复制当前状态的Command对象

命令模式PHP示例

::

    <?php
    /**
     * 命令模式 2010-08-21 sz
     * @author phppan.p#gmail.com  http://www.phppan.com                                                       
     * 哥学社成员（http://www.blog-brother.com/）
     * @package design pattern
     */
 
    /**
     * 命令角色
     */
    interface Command
    {
        /**
         * 执行方法
         */
        public function execute();
    }
 
    /**
     * 具体命令角色
     */
    class ConcreteCommand implements Command
    {
        private $_receiver;
 
        public function __construct(Receiver $receiver)
	{
            $this->_receiver = $receiver;
        }
 
        /**
         * 执行方法
         */
        public function execute()
	{
            $this->_receiver->action();
        }
    }
 
    /**
     * 接收者角色
     */
    class Receiver
    {
        /* 接收者名称 */
        private $_name;
 
        public function __construct($name)
	{
            $this->_name = $name;
        }
 
        /**
         * 行动方法
         */
        public function action()
	{
            echo $this->_name, ' do action.<br />';
        }
    }
 
    /**
     * 请求者角色
     */
    class Invoker
    {
        private $_command;
  
        public function __construct(Command $command)
	{
            $this->_command = $command;
        }
 
        public function action()
	{
            $this->_command->execute();
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
            $receiver = new Receiver('phpppan');
            $command = new ConcreteCommand($receiver);
            $invoker = new Invoker($command);
            $invoker->action();
        }
    }
 
    Client::main(); 
    ?>

命令模式协作
    
    * Client创建一个ConcreteCommand对象并指定它的Receiver对象
    * 某Invoker对象存储该ConcreteCommand对象
    * 该Invoker通过调用Command对象的execute操作来提交一个请求。若该命令是可撤消的，ConcreteCommand就在执行execute操作之前存储当前状态以用于取消命令。
    * ConcreteCommand对象对调用它的Receiver的一些操作以执行该请求。

宏命令：在这里，我们以一个简单的增加和粘贴功能为例，将这两个命令组成一个宏命令。我们可以新建复制命令和粘贴命令，然后将其添加到宏命令中去。如下所示代码：

::

    <?php
    /**
     * 命令模式之宏命令 2010-08-22 00:10  sz
     * @author phppan.p#gmail.com  http://www.phppan.com                                                   
     * 哥学社成员（http://www.blog-brother.com/）
     * @package design pattern
     */
 
    /**
     * 命令角色
     */
    interface Command
    {
        /**
         * 执行方法
         */
        public function execute();
    }
 
    /**
     * 宏命令接口
     */
    interface MacroCommand extends Command
    {
        /**
         *  宏命令聚集的管理方法，可以删除一个成员命令
         * @param Command $command
         */
        public function remove(Command $command);
 
        /**
         * 宏命令聚集的管理方法，可以增加一个成员命令
         * @param Command $command
         */
        public function add(Command $command);
    }
 
    class DemoMacroCommand implements MacroCommand
    {
        private $_commandList;
 
        public function __construct()
	{
            $this->_commandList = array();
        }
 
        public function remove(Command $command)
	{
            $key = array_search($command, $this->_commandList);
            if ($key === FALSE) {
                return FALSE;
            }
 
            unset($this->_commandList[$key]);
            return TRUE;
        }
 
        public function add(Command $command)
	{
            return array_push($this->_commandList, $command);
        }
 
        public function execute()
	{
            foreach ($this->_commandList as $command) {
                $command->execute();
            }
        }
    }
 
    /**
     * 具体拷贝命令角色
     */
    class CopyCommand implements Command
    {
        private $_receiver;
  
        public function __construct(Receiver $receiver)
	{
            $this->_receiver = $receiver;
        }
 
        /**
         * 执行方法
         */
        public function execute()
	{
            $this->_receiver->copy();
        }
    }
 
    /**
     * 具体拷贝命令角色
     */
    class PasteCommand implements Command
    {
        private $_receiver;
   
        public function __construct(Receiver $receiver)
	{
            $this->_receiver = $receiver;
        }
 
        /**
         * 执行方法
         */
        public function execute()
	{
            $this->_receiver->paste();
        }
    }
 
    /**
     * 接收者角色
     */
    class Receiver
    {
        /* 接收者名称 */
        private $_name;
 
        public function __construct($name)
	{
            $this->_name = $name;
        }
 
        /**
         * 复制方法
         */
        public function copy()
	{
            echo $this->_name, ' do copy action.<br />';
        }
 
        /**
         * 粘贴方法
         */
        public function paste()
	{
            echo $this->_name, ' do paste action.<br />';
        }
    }
 
    /**
     * 请求者角色
     */
    class Invoker
    {
        private $_command;
 
        public function __construct(Command $command)
	{
            $this->_command = $command;
        }
 
        public function action()
	{
            $this->_command->execute();
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
            $receiver = new Receiver('phpppan');
            $pasteCommand = new PasteCommand($receiver);
            $copyCommand = new CopyCommand($receiver);
 
            $macroCommand = new DemoMacroCommand();
            $macroCommand->add($copyCommand);
            $macroCommand->add($pasteCommand);
 
            $invoker = new Invoker($macroCommand);
            $invoker->action();
 
            $macroCommand->remove($copyCommand);
            $invoker = new Invoker($macroCommand);
            $invoker->action();
        }
    }
 
    Client::main(); 
    ?>

个人觉得在命令的委派操作上，与访问者模式(Visitor模式)有相似之处。