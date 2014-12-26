备忘录模式（Memento Pattern）
==============================

【意图】
在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样可以在以后把该对象的状态恢复到之前保存的状态。【GOF95】

【备忘录模式结构图】
备忘录模式

【备忘录模式中主要角色】
1、备忘录(Memento)角色：
存储发起人(Originator)对象的内部状态，而发起人根据需要决定备忘录存储发起人的哪些内部状态。
备忘录可以保护其内容不被发起人(Originator)对象之外的任何对象所读取。

2、发起人(Originator)角色：
创建一个含有当前的内部状态的备忘录对象
使用备忘录对象存储其内部状态

3、负责人(Caretaker)角色：
负责保存备忘录对象，不检查备忘录对象的内容

【备忘录模式的优点和缺点】
备忘录模式的优点：
1、有时一些发起人对象的内部信息必须保存在发起人对象以外的地方，但是必须要由发起人对象自己读取。
2、简化了发起人(Originator)类。发起人(Originator)不再需要管理和保存其内部状态的一个个版本，客户端可以自行管理它们所需要的这些状态的版本
3、当发起人角色的状态改变的时候，有可能这个状态无效，这时候就可以使用暂时存储起来的备忘录将状态复原。

备忘录模式的缺点：
1、如果发起人角色的状态需要完整地存储到备忘录对象中，那么在资源消耗上面备忘录对象会很昂贵。
2、当负责人角色将一个备忘录存储起来的时候，负责人可能并不知道这个状态会占用多大的存储空间，从而无法提醒用户一个操作是否会很昂贵。
3、当发起人角色的状态改变的时候，有可能这个状态无效。

【备忘录模式适用场景】
1、必须保存一个对象在某一个时刻的（部分）状态，这样以后需要时它才能恢复到先前的状态。
2、如果一个用接口来让其它对象直接得到这些状态，将会暴露对象的实现细节并破坏对象的封装性。

【备忘录模式与其它模式】
1、命令模式(command模式)：Command模式也可以用来恢复对象的状态，一般Command模式可以支持多级状态的回滚，Memento只是简单的恢复（快照）。在Command模式的每一个undo中，可以使用Memento来保存对象的状态。
2、迭代器模式(Iterator模式)：备忘录可以用于迭代

【备忘录模式PHP示例】

::

    <?php
    /**
     * 备忘录模式 2010-10-09 sz
     * @author phppan.p#gmail.com  http://www.phppan.com                                                       
     * 哥学社成员（http://www.blog-brother.com/）
     * @package design pattern
     */
 
    /**
     * 发起人(Originator)角色
     */
    class Originator
    {
        private $_state;
 
        public function __construct()
	{
            $this->_state = '';
        }
 
        /**
         * 创建备忘录
         * @return Memento 包含当前状态的备忘录对象
         */
        public function createMemento()
	{
            return new Memento($this->_state);
        }  
 
        /**
         * 将发起人恢复到备忘录对象记录的状态上
         * @param Memento $memento
         */
        public function restoreMemento(Memento $memento)
	{
            $this->_state = $memento->getState();
        }
 
        public function setState($state)
	{
            $this->_state = $state;
        }
 
        public function getState()
	{
            return $this->_state;
        }
 
        /**
         * 测试用方法，显示状态
         */
        public function showState()
	{
            echo "Original Status:", $this->getState(), "<br />";
        }
    }
 
    /**
     * 备忘录(Memento)角色
     */
    class Memento
    {
        private $_state;
 
        public function __construct($state)
	{
            $this->setState($state);
        }
 
        public function getState()
	{
            return $this->_state;
        }
 
        public function setState($state)
	{
            $this->_state = $state;
        } 
    }
 
    /**
     * 负责人(Caretaker)角色
     */
    class Caretaker
    {
        private $_memento;
 
        public function getMemento()
	{
            return $this->_memento;
        }
 
        public function setMemento(Memento $memento)
	{
            $this->_memento = $memento;
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
            /* 创建目标对象 */
            $org = new Originator();
            $org->setState('open');
            $org->showState();
    
            /* 创建备忘 */
            $memento = $org->createMemento();
  
            /* 通过Caretaker保存此备忘 */
            $caretaker = new Caretaker();
            $caretaker->setMemento($memento);
 
            /* 改变目标对象的状态 */
            $org->setState('close');
            $org->showState();
 
            /* 还原操作 */
            $org->restoreMemento($caretaker->getMemento());
            $org->showState();
        }
    }
 
    Client::main();
    ?>