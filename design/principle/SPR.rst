﻿单一职责原则（Single Responsibility Principle）
================================================

**定义：就一个类来说，应该仅有一个引起他变化的原因。通俗的说，即一个类只负责一项职责。**

* 问题由来：类T负责两个不同的职责：职责P1，职责P2。当由于职责P1需求发生改变而需要修改类T时，有可能会导致原本运行正常的职责P2功能发生故障。
* 解决方案：遵循单一职责原则。分别建立两个类T1、T2，使T1完成职责P1功能，T2完成职责P2功能。这样，当修改类T1时，不会使职责P2发生故障风险；同理，当修改T2时，也不会使职责P1发生故障风险。

说到单一职责原则，很多人都会不屑一顾。因为它太简单了。稍有经验的程序员即使从来没有读过设计模式、从来没有听说过单一职责原则，在设计软件时也会自觉的遵守这一重要原则，因为这是常识。在软件编程中，谁也不希望因为修改了一个功能导致其他的功能发生故障。
而避免出现这一问题的方法便是遵循单一职责原则。虽然单一职责原则如此简单，并且被认为是常识，但是即便是经验丰富的程序员写出的程序，也会有违背这一原则的代码存在。为什么会出现这种现象呢？因为有职责扩散。所谓职责扩散，就是因为某种原因，职责P被分化为粒度更细的职责P1和P2。
比如：类T只负责一个职责P，这样设计是符合单一职责原则的。后来由于某种原因，也许是需求变更了，也许是程序的设计者境界提高了，需要将职责P细分为粒度更细的职责P1，P2，这时如果要使程序遵循单一职责原则，需要将类T也分解为两个类T1和T2，分别负责P1、P2两个职责。
但是在程序已经写好的情况下，这样做简直太费时间了。所以，简单的修改类T，用它来负责两个职责是一个比较不错的选择，虽然这样做有悖于单一职责原则。（这样做的风险在于职责扩散的不确定性，因为我们不会想到这个职责P，在未来可能会扩散为P1，P2，P3，P4……Pn。
所以记住，在职责扩散到我们无法控制的程度之前，立刻对代码进行重构。）

用一个类描述动物呼吸这个场景

<pre>
class Terrestrial
{
    public function breathe($animal)
    {
       echo $animal . "呼吸空气;\n";
    }
}

$terrestrialObject = new Terrestrial();
$terrestrialObject->breathe('牛');
$terrestrialObject->breathe('羊');
$terrestrialObject->breathe('猪');
   
//运行结果：
牛呼吸空气
羊呼吸空气
猪呼吸空气
</pre>

动物除了呼吸空气外，还有鱼呼吸水的情况

<pre>
class Terrestrial
{
    public function breathe($animal)
    { 
        echo $animal . "呼吸空气;\n";
    }
}

class Aquatic
{
    public function breathe($animal)
    {
        echo $animal . "呼吸水\n";
    }
}

$terrestrialObject = new Terrestrial();
$terrestrialObject->breathe('牛');
$terrestrialObject->breathe('羊');
$terrestrialObject->breathe('猪');
$aquaticObject = new Aquatic();
$aquaticObject->breathe('鱼');

// 运行结果：
牛呼吸空气
羊呼吸空气
猪呼吸空气
鱼呼吸水
</pre>

违背单一职责原则的处理方式

在现有基础上分解类的修改花销较大，甚至还需要修改客户端代码，稍微违背单一职责原则，可以是修改花销降低

<pre>
class Terrestrial
{
    public function breathe($animal)
    {
        if ('鱼' == $animal)) {
            echo $animal . "呼吸水;\n";
        } else {
            echo $animal . "呼吸空气;\n";
        }
    }
}

$terrestrialObject = new Terrestrial();
$terrestrialObject->breathe('牛');
$terrestrialObject->breathe('羊');
$terrestrialObject->breathe('猪');
$terrestrialObject->breathe('鱼');
</pre>

违背单一职责原则的另一种处理方式

上面的处理方式，对breathe方法的修改是有风险的，可以考虑增加一个方法来避免这样带来的风险

<pre>
class Terrestrial
{
    public function breathe($animal)
    {
        echo $animal . "呼吸空气;\n";
    }

    public function breathe2($animal)
    {
        echo $animal . "呼吸水\n";
    }
}


$terrestrialObject = new Terrestrial();
$terrestrialObject->breathe('牛');
$terrestrialObject->breathe('羊');
$terrestrialObject->breathe('猪');
$terrestrialObject->breathe2('鱼');
</pre>
    
例如本文所举的这个例子，它太简单了，它只有一个方法，所以，无论是在代码级别上违反单一职责原则，还是在方法级别上违反，都不会造成太大的影响。实际应用中的类都要复杂的多，一旦发生职责扩散而需要修改类时，除非这个类本身非常简单，否则还是遵循单一职责原则的好。

遵循单一职责原的优点有：
    * 可以降低类的复杂度，一个类只负责一项职责，其逻辑肯定要比负责多项职责简单的多；
    * 提高类的可读性，提高系统的可维护性；
    * 变更引起的风险降低，变更是必然的，如果单一职责原则遵守的好，当修改一个功能时，可以显著降低对其他功能的影响。

需要说明的一点是单一职责原则不只是面向对象编程思想所特有的，只要是模块化的程序设计，都适用单一职责原则。
