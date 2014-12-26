SPL异常
=========================

逻辑异常
------------------------

SPL有两个核心的异常类，即LogicException和RuntimeException，LogicException类直接从Exception类派生，没有添加任何附加方法。这样派生主要是为了区分编译期逻辑异常和由传递给应用程序的非法数据导致的异常。逻辑异常适用于在应用程序编写有误时抛出。可以用来创建一个记录编程异常的系统，并且与运行时数据导致的异常独立开来。下面通过一个示例来介绍逻辑异常的使用场景

::

    class Lazy
    {
        protected $_loaded = false;
	protected $_name;

	public function materialize()
	{
	    $this->_loaded = true;
	    $this->_name = 'Kevin';
	}

	public function getName()
	{
	    if (!$this->_loaded) {
	        throw new LoginException('Call materialize() before accessing');
	    }
	    return $this->_name;
	}
    }

    $lazy = new Lazy();
    echo $lazy->getName();

跟踪记录逻辑异常的示例：

::

    $logfile = 'log.txt';

    try {
        throw new LogicException('Demo');
    } catch (LogicException $le) {
        $file = new SplFileObject($logfile, 'a+');
	$file->fwrite($le->__toString());
    }

运行时异常
-------------------

class RuntimeException extends Exception; 运行时异常，用于处理所有运行时发生的异常问题。如：

::

    function purchase()
    {
        // 启动事务并获得锁
	// 再次检查余额以防止竞争条件出现
	if (checkSufficientFunds()) {
	    // 插入数据并提交事务
	} else {
	    // 中断事务
	    throw new RumtimeException('The account has insufficient funds.');
	}
    }

无效函数调用异常和无效方法
-----------------------------

class BadFunctionCallException extends LogicException; 无效函数调用异常，用于非法的函数调用时；
class BadMethodCallException extends BadFunctionCallException; 无效方法调用异常，用于非法的方法调用时；

域异常
----------------------

class DomainException extends LogicException; 用来处理域异常的。域是指产生函数有效输出值的函数的所有可能值。如

::

    function mathy($x)
    {
        if ($x === 2) {
	    throw new DomainException('X cannot be 2');
	}
	return (1 / ($x - 2));
    }

这里域是不为2的任何数字，因为2会导致被零除的错误发生。因此，当函数接受一个特定的值的有效集合，但却收到无效值时，应该抛出一个DomainException异常。

范围异常
--------------

class RangeException extends RuntimeException; 用来处理范围异常。和域类似，函数的范围是一个数学术语，但他不代表函数输入值的有效范围。如：

::
 
    function mathy($x)
    {
        if ($x < 0) {
	    throw new RangeException('$x must be 0 or greater');
	}
    }

这是一个典型的RangeException使用不当的例子，这里应该由DomainException来处理，因为这里出现问题的是函数的域，而不是函数的范围。函数的范围是指结果（输出值）的所有可能值的集合，RangeException用于抛出与函数结果相关的异常。如：

::
 
    class Sensor
    {
        public static function getTemperature()
        {
            return 51;
        }
    }

    class Monitor
    {
        public static function watch()
	{
	    $temp = Sensor::getTemperature();
	    if (($temp < -50) || ($temp > 50)) {
	        throw new RangeException('The sensor broke down.');
	    }
	}
    }

    Monitor::watch();


无效参数异常
-------------------

class InvalidArgumentException extends LoginException; 无效参数异常，当函数或方法接收一个无效参数时，应抛出一个无效参数异常。与DoaminException的区别在于它不处理数值集合，只处理不兼容类型的集合。如：

::

    function sum($a, $b)
    {
        if (!is_numeric($a) || !is_numeric($b)) {
	    throw new InvalidArgumentException('Invalid Argument');
	}
	return ($a + $b);
    }

    echo sum(1, 2);
    echo sum('a', 2);

长度异常
------------

在发生与长度有关的问题时，程序应该抛出LengthException异常。 class LengthException extends LogicException; 这里的长度可能是字符串长度、数组中过多或过少的元素、执行时间限制或者文件大小，如：

::

    function printmax10($str)
    {
        if (strlen($str) > 10) {
	    throw new LengthException('Input was too long');
	}
	echo $str;
    }

    printmax10('asdf');
    printmax10('abcdefghijk');


溢出异常
--------------

class OverflowException extends RuntimeException; 溢出异常，PHP会自动处理大多数的溢出场景，比如证书溢出或缓存溢出。OverflowException主要用于处理算术溢出场景或者待存储值会导致存储位置溢出的场景。如：

::

    function sumThenInsertDemo($a, $b)
    {
        $sum = $a + $b;
	if ($sum >= 10) {
	    throw new OverflowException("$a + $b will overflow storage");
	}
	$link = pg_connect(...);
	pg_query($link, 'INSERT INTO test values (' . $sum . ')');
    }

向下溢出异常
-------------------

算术向下溢出会在数值过小以至于不能维持精度并且函数结果导致精度受损时发生，有UnderflowException异常类处理：class UnderflowException extends RuntimeException;

PHP的浮点数的精度依赖于操作系统、编译设置以及代码运行的处理器。如：

::

    echo (1 - 0.9) . "\n";    // 0.1
    echo (1 - 0.99) . "\n";   // 0.01
    echo (1 - 0.999) . "\n";  // 0.001
    echo (1 - 0.9999) . "\n"; // 9.999999999999E-05

可以看到，大于三个小数位的时候，会出现精度受损的问题。在对精度要求较高的时候，需要考虑精度可能存在的异常问题，在不能保持精度的情况下应该抛出一个UnderflowException异常。程序员需要明确所编写代码的精度限制，并且在达到这一限制时抛出合适的向下溢出异常，否则，可能损害后面的程序，而且这个类型的错误很难跟踪到。如：

::

    function scale($a)
    {
        return strlen(strstr($a, '.')) - 1;
    }

    function sum($a, $b)
    {
        if ((scale($a) > 3) || (scale($b) > 3)) {
	    throw new UnderflowException('Input scale exceeded');
	}
	return $a + $b;
    }

    echo sum(1, -0.9) . "\n";
    echo sum(1, -0.99) . "\n";
    echo sum(1, -0.999) . "\n";
    echo sum(1, -0.9999) . "\n";

SPL异常小结
-------------

    * LogicException：处理在编译期可以检测到的异常，也就是编写的应用程序代码不正确的时候；
    * RuntimeException：处理只能在运行时检测到的异常；
    * BadFunctionCallException：处理由非法的函数调用导致的异常；
    * DomainException：处理域异常，域是指对于一个函数来说所有有效的输入值的集合；
    * RangeException：处理范围异常，范围指对于一个函数来说所有有效的输出值的集合；
    * InvalidArgumentExcepion：处理函数或方法接受到一个无效参数的情况；
    * LengthException：处理由长度问题导致的异常，例如数组中拥有了过多的元素的情况；
    * OverflowException：处理由数学溢出或存储位置溢出导致的异常；
    * UnderflowException：处理由于数值太小以至于不能保持精度从而引起精度受损而导致的异常，使用浮点数时可能会发生这种情况；