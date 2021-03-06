DIC与IoC
==========

要理解这两个概念，首先要搞清楚以下几个问题：
    * 参与者都有谁？一般有三方参与者，一个是某个对象；一个是IoC/DI的容器；另一个是某个对象的外部资源。某个对象指的就是任意的、普通的对象; IoC/DI的容器简单点说就是指用来实现IoC/DI功能的一个框架程序；对象的外部资源指的就是对象需要的，但是是从对象外部获取的，都统称资源，比如：对象需要的其它对象、或者是对象需要的文件资源等等。
    * 谁依赖于谁？某个对象需要的外部资源；
    * IoC/DI容器的功能是什么？IoC/DI容器用来提供对象需要的外部资源，如果外部资源是对象的话，那么这个对象将由容器来创建；
    * 谁注入于谁？很明显是IoC/DI的容器把某个对象需要的外部资源注入该对象；
    * 到底注入什么？就是注入某个对象所需要的外部资源
    * 谁控制谁？当然是IoC/DI的容器来控制对象了
    * 控制什么？主要是控制对象实例的创建
    * 为何叫反转（有反转就应该有正转了）？反转是相对于正向而言的，那么什么算是正向的呢？考虑一下常规情况下的应用程序，如果要在A里面使用C，你会怎么做呢？当然是直接去创建C的对象，也就是说，是在A类中主动去获取所需要的外部资源C，这种情况被称为正向的。那么什么是反向呢？就是A类不再主动去获取C，而是被动等待，等待IoC/DI的容器获取一个C的实例，然后反向的注入到A类中。
    * 依赖注入和控制反转是同一概念吗？根据上面的讲述，应该能看出来，依赖注入和控制反转是对同一件事情的不同描述，从某个方面讲，就是它们描述的角度不同。依赖注入是从应用程序的角度在描述，可以把依赖注入描述完整点：应用程序依赖容器创建并注入它所需要的外部资源；而控制反转是从容器的角度在描述，描述完整点：容器控制应用程序，由容器反向的向应用程序注入应用程序所需要的外部资源。

其实IoC/DI对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC/DI容器来创建并注入它所需要的资源了。这么小小的一个改变其实是编程思想的一个大进步，这样就有效的分离了对象和它所需要的外部资源，使得它们松散耦合，有利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活

**IOC控制反转：说的是创建对象实例的控制权从代码控制剥离到IOC容器控制，实际就是你在xml文件控制，侧重于原理。**
**DI依赖注入：说的是创建对象实例时，为这个对象注入属性值或其它对象实例，侧重于实现。**

IOC是个比较大的概念, 以下是它的两种实现
    * 依赖查找：容器提供回调接口和上下文环境给组件。EJB和Apache Avalon都是使用这种方式。
    * 依赖注入(DI)：组件不做定位查询，只是提供普通的Java方法让容器去决定依赖关系。容器全权负责组件的装配，它会把符合依赖关系的对象通过 JavaBean 属性或者构造子传递给需要的对象。通过 JavaBean 属性注射依赖关系

理解IoC，我们需要知道Control是什么，它又是怎样被Inversion的。其实，IoC是用来说明“程序库”和“框架”区别的最好证据。在使用程序库的时候，控制权是掌握在我们手中的，我们编写的代码调用程序库的实现，完成相应的功能，想想我们使用JDK的情况。使用框架的时候，控制权则掌握在框架手中，我们的代码最终是由框架调用，一个常见的例子是Servlet，我们编写的Servlet代码是放在整个Servlet的框架中，由Web容器进行调用。这就是差异所在。我们更习惯于自己掌控一切，因此，对框架掌握控制权的这种情况，我们用“Inversion”来形容，这也是Martin Fowler在那篇给DI正名的文章中提到，所有框架都是IoC的原因。

Spring的核心容器是一个框架，所以，我们可以说它是IoC，但是就如前面所说，每个框架都有IoC，所以，仅仅用IoC是不足以说明一切的。Spring核心容器完成的是组件组装的过程，这是它和其它普通框架区别最为显著的地方。如果说用IoC描述这个框架，那么，这里所指的Control实际上是组件的组装过程。

站在Spring核心容器的层面上看，它完成组装过程是把组件所依赖的零部件给组件安装上去。站在单个组件层面上看，它所需要的零部件是由外部给它安装的，这个过程就像是把“Dependency”这管药水用注射器“Injection”到组件的身体中去，所以，我们称之为“Dependency Injection”。

完成组件组装的容器也不只是注入一种形式，还有一种常见的方式是“Dependency Lookup”，即每个组件自己去查找自己所需要的内容。至于到哪去找，也有不同的实现方式，有固定到某个地方（比如使用静态方法），有把查找点通过DI的方式注入进来等等。

相关概念
#### 依赖倒置原则（Dependence Inversion Principle, DIP）

DIP是一种软件设计的指导思想。传统软件设计中，上层代码依赖于下层代码，当下层出现变动时，
上层代码也要相应变化，维护成本较高。而DIP的核心思想是上层定义接口，下层实现这个接口，
从而使得下层依赖于上层，降低耦合度，提高整个系统的弹性。这是一种经实践证明的有效策略。

#### 控制反转（Inversion of Control, IoC）

IoC就是DIP的一种具体思路，DIP只是一种理念、思想，而IoC是一种实现DIP的方法。
IoC的核心是将类（上层）所依赖的单元（下层）的实例化过程交由第三方来实现。
一个简单的特征，就是类中不对所依赖的单元有诸如
``$component = new yii\component\SomeClass（）`` 的实例化语句。

#### 依赖注入（Dependence Inversion, DI）

DI是IoC的一种设计模式，是一种套路，按照DI的套路，就可以实现IoC，就能符合DIP原则。
DI的核心是把类所依赖的单元的实例化过程，放到类的外面去实现。

#### 控制反转容器（IoC Container）

当项目比较大时，依赖关系可能会很复杂。
而IoC Container提供了动态地创建、注入依赖单元，映射依赖关系等功能，减少了许多代码量。
Yii 设计了一个 ``yii\di\Container`` 来实现了 DI Container。

#### 服务定位器（Service Locator）

Service Locator是IoC的另一种实现方式，
其核心是把所有可能用到的依赖单元交由Service Locator进行实例化和创建、配置，
把类对依赖单元的依赖，转换成类对Service Locator的依赖。
DI 与 Service Locator并不冲突，两者可以结合使用。
目前，Yii2.0把这DI和Service Locator这两个东西结合起来使用，或者说通过DI容器，实现了Service Locator。
