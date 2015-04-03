PHP引用
========

引用是什么 
-----------

在 PHP 中引用意味着用不同的名字访问同一个变量内容。这并不像C的指针，替代的是，引用是符号表别名。注意在PHP中，变量名和变量内容是不一样的，因此同样的内容可以有不同的名字。最接近的比喻是Unix的文件名和文件本身——变量名是目录条目，而变量内容则是文件本身。引用可以被看作是Unix文件系统中的hardlink。 

引用做什么 
-----------

PHP的引用允许用两个变量来指向同一个内容。意思是，当这样做时：

<pre>
$a =& $b;
</pre>

这意味着 $a 和 $b 指向了同一个变量。需要注意的是
    * $a和$b在这里是完全相同的，这并不是$a指向了$b或者相反，而是$a和$b指向了同一个地方。 
    * 如果具有引用的数组被拷贝，其值不会解除引用。对于数组传值给函数也是如此。 
    * 如果对一个未定义的变量进行引用赋值、引用参数传递或引用返回，则会自动创建该变量。 

对未定义的变量使用引用

<pre>
function foo(&$var) { }

foo($a); // $a is "created" and assigned to null

$b = array();
foo($b['b']);
var_dump(array_key_exists('b', $b)); // bool(true)

$c = new StdClass;
foo($c->d);
var_dump(property_exists($c, 'd')); // bool(true)
</pre>

同样的语法可以用在函数中，它返回引用，以及用在new运算符中（PHP 4.0.4以及以后版本）： 

<pre>
$bar =& new fooclass();
$foo =& find_var($bar);
</pre>

自 PHP 5 起，new 自动返回引用，因此在此使用 =& 已经过时了并且会产生 E_STRICT 级别的消息。 

如果在一个函数内部给一个声明为global的变量赋于一个引用，该引用只在函数内部可见。可以通过使用$GLOBALS数组避免这一点。 

<pre>
$var1 = "Example variable";
$var2 = "";

function global_references($use_globals)
{
    global $var1, $var2;
    if (!$use_globals) {
        $var2 =& $var1; // visible only inside the function
    } else {
        $GLOBALS["var2"] =& $var1; // visible also in global context
    }
}

global_references(false);
echo "var2 is set to '$var2'\n"; // var2 is set to ''
global_references(true);
echo "var2 is set to '$var2'\n"; // var2 is set to 'Example variable'
</pre>  

把 global $var; 当成是 $var =& $GLOBALS['var']; 的简写。从而将其它引用赋给 $var 只改变了本地变量的引用。 


Note: 

如果在 foreach 语句中给一个具有引用的变量赋值，被引用的对象也被改变。 


Example #3 引用与 foreach 语句



<?php
$ref = 0;
$row =& $ref;
foreach (array(1, 2, 3) as $row) {
    // do something
}
echo $ref; // 3 - last element of the iterated array
?>  



引用做的第二件事是用引用传递变量。这是通过在函数内建立一个本地变量并且该变量在呼叫范围内引用了同一个内容来实现的。例如： 




<?php
function foo(&$var)
{
    $var++;
}

$a=5;
foo($a);
?>  
将使 $a 变成 6。这是因为在 foo 函数中变量 $var 指向了和 $a 指向的同一个内容。更多详细解释见引用传递。 

引用做的第三件事是引用返回。 

引用不是什么 
-------------

如前所述，引用不是指针。这意味着下面的结构不会产生预期的效果： 

<pre>
function foo(&$var)
{
    $var =& $GLOBALS["baz"];
}
foo($bar);
</pre>  

这将使 foo 函数中的 $var 变量在函数调用时和 $bar 绑定在一起，但接着又被重新绑定到了 $GLOBALS["baz"] 上面。不可能通过引用机制将 $bar 在函数调用范围内绑定到别的变量上面，因为在函数 foo 中并没有变量 $bar（它被表示为 $var，但是 $var 只有变量内容而没有调用符号表中的名字到值的绑定）。可以使用引用返回来引用被函数选择的变量。 

引用传递 
--------

可以将一个变量通过引用传递给函数，这样该函数就可以修改其参数的值。语法如下：

<pre>
function foo(&$var)
{
    $var++;
}

$a=5;
foo($a); // $a is 6 here
</pre>  

注意在函数调用时没有引用符号——只有函数定义中有。光是函数定义就足够使参数通过引用来正确传递了。在最近版本的 PHP 中如果把 & 用在 foo(&$a); 中会得到一条警告说"Call-time pass-by-reference"已经过时了。 

以下内容可以通过引用传递： 
    * 变量，例如 foo($a)  
    * New 语句，例如 foo(new foobar())  
    * 从函数中返回的引用，例如： 

<pre>
function &bar()
{
    $a = 5;
    return $a;
}
foo(bar());
</pre>  

任何其它表达式都不能通过引用传递，结果未定义。例如下面引用传递的例子是无效的： 




<?php
function bar() // Note the missing &
{
    $a = 5;
    return $a;
}
foo(bar()); // 自 PHP 5.0.5 起导致致命错误
foo($a = 5) // 表达式，不是变量
foo(5) // 导致致命错误
?>  
这些条件是 PHP 4.0.4 以及以后版本有的。

引用返回 
---------

引用返回用在当想用函数找到引用应该被绑定在哪一个变量上面时。不要用返回引用来增加性能，引擎足够聪明来自己进行优化。仅在有合理的技术原因时才返回引用！要返回引用，使用此语法： 

<pre>
class foo {
    public $value = 42;

    public function &getValue() {
        return $this->value;
    }
}

$obj = new foo;
$myValue = &$obj->getValue(); // $myValue is a reference to $obj->value, which is 42.
$obj->value = 2;
echo $myValue;                // prints the new value of $obj->value, i.e. 2.
</pre>  

本例中getValue函数所返回的对象的属性将被赋值，而不是拷贝，就和没有用引用语法一样。 

和参数传递不同，这里必须在两个地方都用 & 符号——指出返回的是一个引用，而不是通常的一个拷贝，同样也指出 $myValue 是作为引用的绑定，而不是通常的赋值。  

如果试图这样从函数返回引用：return ($this->value);，这将不会起作用，因为在试图返回一个表达式的结果而不是一个引用的变量。只能从函数返回引用变量——没别的方法。如果代码试图返回一个动态表达式或 new 运算符的结果，自 PHP 4.4.0 和 PHP 5.1.0 起会发出一条 E_NOTICE 错误。 

取消引用 
---------

当unset一个引用，只是断开了变量名和变量内容之间的绑定。这并不意味着变量内容被销毁了。例如： 

<pre>
$a = 1;
$b =& $a;
unset($a);
</pre>  

不会unset $b，只是$a。再拿这个和Unix的unlink调用来类比一下可能有助于理解。 

引用定位 
---------

许多PHP的语法结构是通过引用机制实现的，所以上述有关引用绑定的一切也都适用于这些结构。一些结构，例如引用传递和返回，已经在上面提到了。其它使用引用的结构有： 

global 引用 

当用 global $var 声明一个变量时实际上建立了一个到全局变量的引用。也就是说和这样做是相同的： 

<pre>
$var =& $GLOBALS["var"];
</pre>  

这意味着，例如，unset $var不会unset全局变量。 

$this 在一个对象的方法中，$this 永远是调用它的对象的引用。 
 
