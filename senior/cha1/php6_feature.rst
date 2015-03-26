PHP 6中的新特性
=================

新特性概述

1. 支持Unicode，Unicode是有其必然，虽然Unicode占用较多的空间，但Unicode带来的便利性，远超过占用空间的缺点，尤其在国际化的今天，硬件设备越来越强大，网速也大幅度的提升，这么一点小小的缺点是可以忽略的。另外一点，PHP也可以在.ini文件中设定能不能开启支持Unicode，决定权在你自己，这是一个不错的点子，关掉Unicode的支持，PHP的性能并不会有大幅度的提升，主要的影响在于需要引用字符串的函数。
2. Register Globals 将被移除，这是一个主要的决定，老的PHP使用者会觉得Register Globals蛮方便的，但是却忽略了Register Globals会带来程序上安全性的隐患，大多数的主机上此项功能是关闭的，印象中从PHP4.3.x版开始，此项默认配置值即是关闭状态，PHP6正式移除Register Globals也代表着如果程序是PHP3时代的产物，将完全不能运用，除了改写用途外，别无他法。
3. Magic Quotes将取消，Magic Quotes主要是自动转义须要转义的字符，此项功能移除也符合大多数PHP开发者的心声。
4. Safe Mode 取消，老实说，这个模式不知道哪里不好，取消就取消吧，反正也用不到。
5. ’var’ 别名为 ‘public’，在类中的var声明变成public的别名，相信是为了兼容PHP5而作的决定，PHP6现在也可以称作为OO语言了。
6. 通过引用返回将出错，未来通过引用返回编译器将会报错 例如$a =& new b()、function &c()，OO语言默认就是引用，所以不须要再运用&了。
7. zend.ze1 compatbility mode 将被移去，（Zend.ze1相容模式将被移去），PHP5是为兼容旧有PHP4，所以在.ini中可选择是否开启相容模式，原由在于PHP5运用的是第二代分析引擎，但是相容模式并不是百分之百能分析PHP4语法，所以旧时代的产物，移除。
8. Freetype 1 and GD 1 support将不见，这两个Libs存在了很久，php6将不再支持，况且GD1早已被现在的GD2取代了。
9. dl()被移到SAPI中，dl()主要是让设计师加载extension Libs，现在被移到 SAPI 中。
10. Register Long Array去除 从PHP5起默认是关闭，在PHP6中正式移除。
11. 一些Extension的变更，例如XMLReader和XMLWriter将不再是以Extension的方式出现，他们将被移入到PHP的核心之中，并且默认是开启，ereg，extension将被放入PECL，代表着它将被移出PHP核心，这也是为了让路给新的正则表达式extension，此外，Fileinfo extension也将被导入PHP的核心之中。
12. APC将被导入核心，这是一个提高PHP性能的功能，现在它将被放入PHP核心中，并且可以选择是否启用APC。
13. 告别ASP风格的起始标签，原来是为了取悦ASP开发者转向运用PHP，现今已经不再须要这种做法了。
最后，别期望PHP6的性能可以彻底超过PHP5，有可能PHP6的执行效率会比PHP5还要来得慢，但是可以预期的是，PHP开发小组将会努力的完善PHP6，超越PHP5。

PHP6中的Unicode支持
-----------------------

Unicode改变了许多事情，字符串长度不再是其使用的存储空间的字节数了，Unicode中大量的字符可能是两个或多个字符的，因此PHP6重写了大多数的字符串函数，如strlen()。

#### Unicode语义

#### Unicode排序规则

命名空间
----------

命名空间对于PHP类来说就像文件和目录的关系，给类库添加了结构和层次关系，命名空间允许你为两个不同的类使用相同的类名称。命名空间使用了两个关键字**namespace**和**use**。要声明一个命名空间，需要在文件的顶部指定命名空间的名称，在其他任何变量、类或者函数定义的前面，一个文件只能声明一个命名空间。use允许为一个特定的命名空间起别名。如：

<pre>
<?php
namespace Vector;

class Line
{
    public function draw($x1, $y1, $x2, $y2) {}
}

// another file
<?php
require_once('Vector.php');

$line = new Vector::Line();
$line->draw(1, 1, 10, 10);

// another file
<?php
require_once('Vector.php');

use Vector::Line as Line;

$line = new Line();
$line->draw(1, 1, 10, 10);
</pre>

延迟静态绑定
-----------------

PHP继承模型中由一个存在已久的问题，就是在父类中引用扩展类的最终状态比较困难，如：

<pre>
<?php
class ParentBase
{
    static $property = 'Parent Value';

    public static function render()
    {
        return self::$property;
    }
}

class Descendant extends ParentBase
{
    static $property = 'Descendant Value';
}

echo Descendant::render();
// Parent Value
</pre>

通过引入延迟静态绑定功能，可以使用static作用域关键字访问类的属性或者方法的最终值，如：

<pre>
<?php
class ParentBase
{
    static $property = 'Parent Value';
    public static function render()
    {
        return static::$property;
    }
}

class Descendant extends ParentBase
{
    static $property = 'Descendant Value';
}
echo Descendant::render();
// Descendant Value
</pre>

通过使用静态作用域，可以强制PHP在最终的类中查找所有属性的值，除了这个延迟绑定行为，PHP还添加了get_called_class()函数，这允许检查集成的方式是从哪个派生类条用的，如：

<pre>
class ParentBase
{
    public static function render()
    {
        return get_called_class();
    }
}

class Decendant extends ParentBase {}

echo Descendant::render();
// Descendant
</pre>

具有动态特性的静态方法
----------------------

__call()函数可用来创建任意匹配类型的方法，这种方法可以处理调用类的未定义方法的情况。方法的参数是被调用的方法的名称以及传递给该方法的参数数组，现在可以用和__call()方法类似的方式创建具有动态特性的静态方法，在PHP6中，静态功能是通过实现魔术方法__callStatic()来实现，如

<pre>
<?php
class MyClass
{
    public static function __callStatic($name, $parameters)
    {
        echo $name . ' method called. Parameters: ' . PHP_EOL .
            var_export($parameters, true) . PHP_EOL;
    }
}
MyClass::bogus(1, false, 'a');
</pre>

三目运算符（ifsetor）
--------------------------

三目运算符又称“?运算符”，如：

<pre>
$value = $expresion ? $result1 : $result2;  
// 这种形式的三目运算符有种简便的写法
$value = $value1 ?: 'default';
<pre>

noXMLWriter类
-----------------

no小结
----------
