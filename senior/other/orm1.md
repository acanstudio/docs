ORM基础
=========

基本概念
----------

**ORM（Object Relational Mapping），对象关系映射。指的是面向对象的对象模型和关系型数据库的数据结构之间的相互转换。**

ORM可以被认为是基于关系型数据库的数据存储，实现一个虚拟的面向对象的数据访问接口。理想情况下，基于这样一个面向对象的接口，持久化一个OO对象应该不需要要了解任何关系型数据库存储数据的实现细节。

我们需要面向对象来描述我们的业务，我们也需要关系型数据库来存储我们的数据。而ORM就是协调业务和数据的一个抽象层。

ORM的可行性问题分析，即从理论上，从数学上，面向对象的对象模型和关系型数据库的数据结构之间，到底能不能做到完美的映射？要回答这个问题，我们要解决两个难题。

#### O/R阻抗失衡

第一个难题，叫“O/R阻抗失衡(O/R Impedance Mismatch)”，指的是OO对象模型和关系型数据库的数据结构之间的，设计理念上的差异。OO的设计理念，是以最正确的语义来描述真实世界；而关系型数据库的设计理念，则是从数学的角度，如何更有效的存储和管理数据。由于两者设计理念的差异，导致它们尽管从数据结构上，可能很相近，但是关注点往往是不同的。

例如：从OO的角度，凡是语义明确，每一个对象语义上的属性，就应该定义为一个属性；从关系型数据库的角度，有可能考虑到一些属性从来不会被作为查询条件，而把多个语义上的属性，以一定的格式，存在一个数据表的字段中，也有可能因为一组语义上的属性使用的频度完全不同与另一组属性，即使他们语义上属于一个对象，也有可能将他们拆分成两个数据表来存储。从OO的角度，对象只关心自己的固有属性，不需要被唯一标识；但是，从关系型数据库的角度，一般一个数据表中的每一行数据都需要要一个唯一标识，且很可能是语义上没有意义的，如自增长的标识。从OO角度的优化，一般遵循SOLID这样的原则，以更正确的语义来组织对象；从关系型数据库角度的优化，往往为了查询性能，来修改字段的类型、长度，修改索引，甚至分表、分库。

一个ORM工具想要通过简单的配置，完美的解决“O/R阻抗失衡”问题，在我看来几乎是不可能的。但是，一定程度上，通过灵活的配置支持绝大多数常见的映射策略，再加上提供可供用户通过自定代码来扩展的接口的话，应该还是可以相当接近完美的。

#### 文化阻抗失衡

第二个难题，叫“文化阻抗失衡(Cultural Impedance Mismatch)”，指的是关系型数据专家和面向对象专家之间的文化差异。关于这个难题的最经典的争论是“到底应该以关系型数据库的数据结构来驱动，还是以OO的对象模型来驱动程序的开发？”。

关于这个争论，OO专家的主要观点是：我的业务才是程序的核心，数据库只是为我持久化数据的需求服务的，所以设计OO对象模型的时候，我不应该考虑如何存储到关系型数据库的问题；

而数据库专家的主要观点是: 数据才是公司的核心财富，数据的稳定性远远强于OO对象模型的稳定性，由OO对象模型来驱动数据库架构的设计，根本保证不了性能，数据库维护的成本根本不可接受。

其实，争论的真正原因即在于“关系型数据专家和面向对象专家之间的文化差异”。Scott W. Ambler的文章Why Data Models Shouldn't Drive Object Models(And Vice Versa)，很好的解答了这个问题。

归根结底，既然OO和关系型数据库都是不可缺少的部分，需要协同工作，希望做到完美的ORM，不仅仅需要好的ORM工具，更需要无论是OO专家还是数据库专家互相了解对方的技术和设计理念。对一个OO专家，一个ORM工具，可以对其所支持的常见的映射，提供直接的支持，但是，应用这些ORM工具的OO专家，必须了解关系型数据库，知道如何在不影响OO对象的语义的前提下，如何设计一个更容易映射到关系型数据库的对象模型，知道如何选择ORM工具所支持的映射方式，以及如何用自定代码扩展ORM所不能方便支持的映射方式；同样的，对一个关系型数据库的专家，也需要了解OO的语义，和可能发生的重构，从而，使得所设计的数据库结构，更容易映射到OO对象，更容易响应OO对象模型的重构。如此才是真正的和谐，真正的完美。

综上所述，无论对一个OO专家还是对一个关系型数据库专家，都需要了解OO，并了解关系型数据库，了解ORM的基本原理。离开全面的知识，一个再完美的ORM工具也无法被正确使用，拥有这些知识，则能够充分利用已有的ORM工具来加速自己的工作，并且合理的或扩展现有的ORM工具，或用自定代码，实现ORM工具能力之外的ORM映射。这才是理想的ORM实践。下一章节，我们就来谈谈一下，ORM的基本原理。

如何进行O/R Mapping
---------------------

#### 简单映射

1. Class <-> Table

一个Class一般可以映射为一个Table，一个Class的实例对应Table的一行数据。但是，一个Table中的每行数据，一般都需要有一个主键来唯一标识这行数据，而一个Class的每个实例，则不一定需要一个唯一标识。

2. Property <-> Field

一个Class的Property一般可以直接映射为Table的一个Field。但是，他们的数据类型不一定直接匹配。如果他们代表的数据类型的语义上可转换，则Field的类型，应大于等于Property的数据类型。如果他们代表的类型语义上不可转换，则需要在应用程序层面，进行自定义的转换。

#### 继承映射

1. 单表映射整个继承体系

用一张数据库表存储整个继承体系中的所有Class的数据，数据表需要额外的标志字段来区分一行记录应该映射到继承体系中的哪一个Class，适合继承体系层次较少，总记录数相对较少，子类对父类的属性扩展也相对不那么频繁的情形。

单表映射整个继承体系的优点是读/写继承体系中的每个Class的数据，都只需操作一张表，性能较好，并且，新增继承类，或扩展Class属性都只需要增减一张表的字段就可以了，易于维护；主要缺点是，因为继承体系中所有的Class共享一张表，表中会有比较多的NULL字段值的数据，浪费了一些存储空间，同时，如果记录数过多，表就会更庞大，也会影响表的读写性能。

2. 一个Class映射一个具体表

所谓一个Class映射一个具体表就是每个Class对应一张数据表，并且，每个数据表冗余包含其父类的所有属性字段，并且，子类和父类共享相同的主键值。一个Class一个具体表方案适合需要较高查询性能，继承体系层次不太复杂，并且基类包含较少的属性而子类扩展较多属性，并且能够承受一定的数据库冗余的情况。

一个Class映射一个具体表方案的优点主要就是查询性能好，读操作只需操作一张表，和实体数据的对应结构清晰，数据库表迁移和维护会比较方便；主要的缺点是数据冗余较大，因为每次插入一条子类数据时，同时要插入一份子类包含的父类字段的数据到所有父类层次表中。

3. 一个Class映射一个扩展表

所谓一个Class映射一个扩展表是指继承体系中的每个Class对应一张数据表，但是，每个子类不冗余包含父类的所有属性，而只是包含扩展的属性和共享的主键值。一个Class映射一个扩展表方案适合继承体系非常复杂，结构易变，并希望最大程度减少数据冗余的情形。

一个Class映射一个扩展表方案的优点是结构灵活，新增子类或插入中间的继承类都很方便，冗余数据最少；但是缺点是，无论读还是写操作都会涉及到子类和所有的父类。读操作时，必须自然链接查询所有的父类对应的数据表，而插入或更新数据时，也需要写所有的父类表。

4. 通用的表结构映射所有的Class

这种方案其实不仅支持用一张表存储一个继承体系，它甚至可以支持，用一张表存储任意数量的不同Class。它的原理是元数据驱动。这张表的每一行，包含一个类型的标识字段，一个表示Class的属性名称的字段和一个表示Class的属性值的字段。在运行时，通过唯一标识取出描述一个Class实例的所有Property的值，再根据Property的名称来映射。

#### 关联映射

1. 一对一关联、一对多关联（包含一对一和一对多的自关联）

所谓一对一关联，实际上还可以分为三种情形，即0..1 - 1，1 – 1，1 – 0..1三种情形；而一对多关联则分为* - 1和1 - *。

以下三种方案中第1)种为最常用的映射方案，后面几种是在某些特殊情形下可参考的方案：

1) 最常用的方案为需要其他对象引用的类对应的表增加一个到被引用对象对应表的外键即可，只不过，与表对应的实体类代码中，对于一对多情形下的“多”这一端，需定义成集合类型；

2) 在Hibernate称为“组件（Component）映射”，举例来说，假如Person类包含一个Address成员类型的属性，而Address由City，Street，ZipCode三个成员属性组成，假如Address除了与Person关联不被其他对象使用，则我们可以考虑只用一张数据表Person来持久化Person和Address这两张表，Person数据表包含Person类中除Address的属性和Address类中的所有属性的集合，当然，这时需要在元数据中特别指明映射关系；

3) 还有一种针对上面的方案中的Person，Address两个类的持久化方案则是将Address类型的所有属性先序列化，再存入Person表的字段Address中，这样也可以只用一张表来持久化两个类，当然，本方案中这种被序列化对象成员数据量应尽量小；

4) 还有一种方案是共享同一主键值的一对一关联。即将原本可以同属于一个表中相对使用不太频繁的字段提出来放在另一张表中，这样，这两张表的记录就可以通过一个相同的主键进行关联。

2. 多对多关联（包含多对多的自关联）

所谓多对多关联自然就是* - *这种情形了。一般都需要一张包含关联双方主键的关联表，在取数据时，需要链接该关联表和数据表。

一个PHP实现的ORM操作MySQL数据库实例
------------------------------------

ORM用面向对象的方式来操作数据库。归根结底，还是对于SQL语句的封装。

首先，我们的数据库有如下一张表：

<pre>
CREATE TABLE `tb_user` (
  `userid` varchar(100),
  `username` varchar(50),
  `gender` varchar(5),
  `birthay` date,
  `info` varchar(200),
  `imgurl` varchar(100),
  `islock` int(11)
);
</pre>

我们希望能够对这张表，利用setUserid("11111")，即可以设置userid；getUserid()既可以获得对象的userid。所以，我们需要建立model对象，与数据库中的表对应。

由于每张表所对应的model都应该是有set/get操作，所以，我们用一个父类BasicModel进行定义。其他model都是继承至这个model。

BasicModel的代码如下：
<pre>
/**
 * author:Tammy Pi 
 * function:Model类的基类，封装set/get操作 
 */  
class BasicModel{  
      
    private $map = null;  
      
    function tbUser()
    {  
        $this->map = array();  
    }     
      
    function __set($key, $value)
    {  
        $this->map[$key] = $value;  
    }  
      
    function __get($key)
    {  
        return $this->map[$key];  
    }  
      
    function __call($name, $arguments)
    {  
        if(substr($name, 0, 3) == 'set') {  
            $this->__set(strtolower(substr($name,3)),$arguments[0]);  
        }else{  
            return $this->__get(strtolower(substr($name,3)));  
        }  
    }  
}  

// 那么，与tb_user表相互对应的model类TbUser则对它进行继承。
require_once("BasicModel.php");  
class TbUser extends BasicModel{}  
</pre>

要用ORM进行操作数据库的一些基本操作
    * findByWhere($where)进行查询，返回的为对象数组；
    * save($tbUser)进行保存;
    * delete($obj)进行删除;
    * update($obj)进行更新操作。

本质上，就是用户传入的是对象，我们再利用代码将对象转换为SQL语句。本质上，执行的还是SQL语句。接口和具体实现如下：

<pre>
<?php   
interface IBasicDAO
{
    public function findByWhere($where);  
      
    public function findWhereOrderBy($where,$order,$start=null,$limit=null);  
      
    public function save($obj);  
      
    public function delete($obj);  
      
    public function update($obj);  
}  

// 具体实现another file      
require_once("IBasicDAO.php");  
       
class BasicDAO implements IBasicDAO
{  
    protected $modelName = null;  
    private $tableName = null;  
      
    private $h = "localhost";  
    private $user = "root";  
    private $pass = "root";  
    private $db = "db_toilet";  
      
    // 获得连接  
    public function getConnection()
    {
        $conn = mysqli_connect($this->h,$this->user,$this->pass,$this->db);  
        return $conn;  
    }  
      
    // 初始化  
    public function init()
    {
        // 根据model的名字得到表的名字  
        $this->tableName = strtolower(substr($this->modelName, 0, 2)) . "_" . strtolower(substr($this->modelName, 2));  
    }  
      
    // 获得一个表的列名  
    public function getColumn($tableName)
    {  
        $sql = "show columns from ".$tableName;  
        $conn = $this->getConnection();  
        $columns = array();  
          
        if($conn!=null) {
            $rtn = mysqli_query($conn,$sql);  
            while($rtn !== false && ($row = mysqli_fetch_array($rtn)) != null) {  
                $columns[] = $row[0];  
            }  
              
            mysqli_close($conn);  
        }  
          
        return $columns;  
    }  
      
    // 条件查询  
    public function findByWhere($where)
    {
        $columns = $this->getColumn($this->tableName);  
          
        $sql = "select * from ".$this->tableName." where ".$where;  
        $conn = $this->getConnection();  
        $arr = array();  
        if( $conn != null) {  
            $rtn = mysqli_query($conn,$sql);  
            while($rtn!==false&&($row=mysqli_fetch_array($rtn))!=null){  
                $index = -1;  
                $obj = new $this->modelName();  
                foreach($columns as $column){  
                    $obj->{"set".ucfirst($column)}($row[++$index]);  
                }  
                  
                $arr[] = $obj;  
            }  
            mysqli_close($conn);  
        }  
          
        return $arr;  
    }  
      
      
    // 分页查询；支持排序  
    public function findWhereOrderBy($where,$order,$start=null,$limit=null){  
        $columns = $this->getColumn($this->tableName);  
        $sql = "select * from ".$this->tableName." where ".$where." order by ".$order;  
        if($start!=null&&$limit!=null){  
            $sql .= "limit ".$start.",".$limit;  
        }  
          
        $conn = $this->getConnection();  
        $arr = array();  
        if($conn!=null){  
            $rtn = mysqli_query($conn,$sql);  
            while($rtn!==false&&($row=mysqli_fetch_array($rtn))!=null){  
                $index = -1;  
                $obj = new $this->modelName();  
                foreach($columns as $column){  
                    $obj->{"set".ucfirst($column)}($row[++$index]);  
                }  
                $arr[] = $obj;  
            }  
            mysqli_close($conn);  
        }  
          
        return $arr;  
    }  
      
    // 保存操作  
    public function save($obj){  
        $columns = $this->getColumn($this->tableName);  
        $conn = $this->getConnection();  
        $tag = false;  
        if($conn!=null){  
            $sql = "insert into ".$this->tableName."(";  
            foreach($columns as $column){  
                $sql .= $column.",";  
            }  
              
            $sql = substr($sql,0,strlen($sql)-1).") values(";  
            foreach($columns as $column){  
                $value = $obj->{"get".ucfirst($column)}();  
                //判断$value的类型  
                if($value==null){  
                    $sql .= "null,";  
                }else if(preg_match("/^[0-9]*$/", $value)){  
                    //是数字  
                    $sql .= $value.",";  
                }else{  
                    $sql .= "'".$value."',";  
                }  
            }  
              
            $sql = substr($sql,0,strlen($sql)-1);  
            $sql .= ")";  
              
            //执行sql语句  
            mysqli_query($conn,$sql);  
            $tag = true;  
              
            mysqli_close($conn);  
        }  
          
        return $tag;  
    }  
      
    // 删除操作  
    public function delete($obj){  
        $conn = $this->getConnection();  
        $tag = false;  
        if($conn!=null){  
            $sql = "delete from ".$this->tableName." where ";  
            $columns = $this->getColumn($this->tableName);  
            $value = $obj->{"get".ucfirst($columns[0])}();  
            if($value!=null){  
                //是数字  
                if(preg_match("/^[0-9]*$/", $value)){  
                    $sql .= $columns[0]."=".$value;  
                }else{  
                    $sql .= $columns[0]."='".$value."'";  
                }  
                //执行  
                mysqli_query($conn,$sql);  
                $tag = true;  
            }  
            mysqli_close($conn);  
        }  
          
        return $tag;  
    }  
      
    // 更新操作  
    public function update($obj)
    {
        $conn = $this->getConnection();  
        $columns = $this->getColumn($this->tableName);  
        $tag = false;  
          
        if($conn!=null){  
            $sql = "update ".$this->tableName." set ";  
            for($i=1;$i<count($columns);$i++){  
                $column = $columns[$i];  
                $value = $obj->{"get".ucfirst($columns[$i])}();  
                if($value==null){  
                    $sql .= $column."=null,";  
                }else if(preg_match("/^[0-9]*$/",$value)){  
                    $sql .= $column."=".$value.",";  
                }else{  
                    $sql .= $column."='".$value."',";  
                }  
            }  
              
            $sql = substr($sql,0,strlen($sql)-1);  
            $sql .= " where ";  
            $tempColumn = $columns[0];  
            $tempValue = $obj->{"get".ucfirst($columns[0])}();  
              
            if(preg_match("/^[0-9]*$/", $tempValue)){  
                $sql .= $tempColumn."=".$tempValue;  
            }else{  
                $sql .= $tempColumn."='".$tempValue."'";  
            }  
              
            //执行操作  
            mysqli_query($conn,$sql);  
            $tag = true;  
            mysqli_close($conn);  
        }  
          
        return $tag;  
    }  
}  
</pre>

那么，对tb_user表进行操作时，主要利用的是TbUserDAO，它将modelName设置为"TbUser"，代码就得知操作的表为tb_user，然后就可以进行一系列操作了。

<pre> 
<?php   
require_once("BasicDAO.php");  
require_once("../model/TbUser.php");  
      
class TbUserDAO extends BasicDAO
{  
    function TbUserDAO()
    {
        $this->modelName = 'TbUser';  
        parent::init();  
    }  
}  

// 那么，就可以采用面向对象的方式对数据库进行操作了。
$tbUserDAO = new TbUserDAO();
$tbUser = new TbUser();
$tbUser->setUserid("fetchingsoft@163.com");
$tbUser->setUsername("fetching");
 
$tbUserDAO->update($tbUser);
echo "执行成功！";
 
print_r($list);
</pre>          
