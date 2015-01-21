抽象类、接口和契约式编程
=========================

抽象类、接口和契约式编程，这些机制在概念层次上定义类之间交互作用的规则，是应用程序层次化和可扩展的基础。

抽象类
-------------

方法原型（prototype）：指不定义函数体的，只声明方法存取级别、函数关键字、函数名和参数。只有方法原型的方法又称作抽象方法。如：public function prototypeName($params);

使用abstract声明一个类为抽象类，一个抽象类中可以没有方法原型，抽象类中可以定义若干普通的方法。不能直接使用抽象类生成对象，只有继承了抽象类，并且全部实现了抽象方法的类可以使用new生成对象。此外，PHP不支持多重继承，只支持从一个基类继承。继承抽象类使用的关键字是extends。

抽象类的相关规则：

    * 某个类中只要包含至少一个抽象方法就必须声明为抽象类
    * 声明为抽象的方法，在实现的时候必须包含相同的或者更低的访问级别
    * 不能使用new关键字创建抽象类的实例
    * 被声明为抽象的方法不能包含函数体
    * 如果将扩展的类也声明为抽象类，则在扩展抽象类时，可以不用实现抽象类中的抽象方法

抽象类相关的示例：

::

    <?php
    abstract class Car
    {
        // common methods

	abstract function getMaximumSpeed();
    }

    class FastCar extends Car
    {
        function getMaximumSpeed()
	{
	    return 150;
	}
    }

    class Street
    {
        protected $speedLimit;
	protected $cars;

	public function __construct($speedLimit = 200)
	{
	    $this->cars = array();
	    $this->speedLimit = $speedLimit;
	}

	protected function isStreetLegal($car)
	{
	    if ($car->getMaximumSpeed() < $this->speedLimit) {
	        return true;
	    } else {
	        return false;
	    }
	}

	public function addCar($car)
	{
	    if ($this->isStreetLegal($car)) {
	        echo 'The Car was allowed on the road.';
		$this->cars[] = $car;
	    } else {
	        echo 'The Car is too fast and was not allowed on the road.';
	    }
	}
    }

    $street = new Street();
    $street->addCar(new FastCar());



接口
----------

interfaceof操作符
---------------------

interfaceof是一个比较操作符，接受左右两个参数，并返回一个boolean类型值，用来确定对象的某个实现是否为特定类型，或者是否从某个类继承，又或者是否实现了某个特定的接口。类型代表的是某个类的运行时定义，PHP在解析类或者接口时，会创建一个“类型”。

使用了instanceof的更强壮的代码

::

    <?php
    abstract class Car
    {
        // common methods

	abstract function getMaximumSpeed();
    }

    class FastCar extends Car
    {
        function getMaximumSpeed()
	{
	    return 150;
	}
    }

    class Street
    {
        protected $speedLimit;
	protected $cars;

	public function __construct($speedLimit = 200)
	{
	    $this->cars = array();
	    $this->speedLimit = $speedLimit;
	}

	protected function isStreetLegal($car)
	{
	    if ($car instanceof ISpeedInfo) {
	        if ($car->getMaximumSpeed() < $this->speedLimit) {
	            return true;
	        } else {
	            return false;
	        }
	    } else {
	        return false;
	    }
	}

	public function addCar($car)
	{
	    if ($this->isStreetLegal($car)) {
	        echo 'The Car was allowed on the road.';
		$this->cars[] = $car;
	    } else {
	        echo 'The Car is too fast and was not allowed on the road.';
	    }
	}
    }

    $street = new Street();
    $street->addCar(new FastCar());

no契约式编程
----------------------

no小结
-------