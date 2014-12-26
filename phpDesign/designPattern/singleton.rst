单例模式（Singleton Pattern）
==============================

【意图】
保证一个类仅有一个实例，并且提供一个访问它的全局访问点【GOF95】
单例模式有三个特点：
1、一个类只有一个实例
2、它必须自行创建这个实例
3、必须自行向整个系统提供这个实例

【单例模式结构图】
单例模式

【单例模式中主要角色】
Singleton 定义一个Instance操作，允许客户访问它的唯一实例。Instance是一个类方法。负责创建它的唯一的实例。

【单例模式的优点】
1、对唯一实例的受控访问
2、缩小命名空间 单例模式是对全局变量的一种改进。它避免了那些存储唯一实例的全局变量污染命名空间
3、允许对操作和表示的精华 单例类可以有子类。而且用这个扩展类的实例来配置一个应用是很容易的。你可以用你所需要的类的实例在运行时刻配置应用。
4、允许可变数目的实例（多例模式）
5、比类操作更灵活

【单例模式适用场景】
1、当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时
2、当这个唯一实例应该是通过子类化可扩展的。并且用户应该无需更改代码就能使用一个扩展的实例时。

【单例模式与其它模式】
工厂方法模式(factory method模式)：单例模式使用工厂模式来提供自己的实例。
抽象工厂模式(abstract factory模式)：抽象工厂模式可以使用单例模式，将具体工厂类设计成单例类。
建造者模式(Builder模式)：建造模式可以将具体建造类设计成单例模式。
……

【单例模式PHP示例】

::

    <?php
    /**
     * 单例模式 2010-06-06 sz
     * @author phppan.p#gmail.com  http://www.phppan.com
     * 哥学社成员（http://www.blog-brother.com/）
     * @package design pattern
     */
  
    /**
     * 懒汉式单例类
     */
    class Singleton
    {
        /**
         * 静态成品变量 保存全局实例
         */
        private static  $_instance = NULL;
 
        /**
         * 私有化默认构造方法，保证外界无法直接实例化
         */
        private function __construct(){}
 
        /**
         * 静态工厂方法，返还此类的唯一实例
         */
        public static function getInstance()
	{
            if (is_null(self::$_instance)) {
                self::$_instance = new Singleton();
            }
 
            return self::$_instance;
        }
 
        /**
         * 防止用户克隆实例
         */
        public function __clone()
	{
            die('Clone is not allowed.' . E_USER_ERROR);
        }
 
        /**
         * 测试用方法
         */
        public function test()
	{
            echo 'Singleton Test!';
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
            $instance = Singleton::getInstance();
            $instance->test();
        }
    }
 
    Client::main();
    ?>

PHP中不支持饿汉式的单例模式
因为PHP不支持在类定义时给类的成员变量赋予非基本类型的值。如表达式，new操作等等