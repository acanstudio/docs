C shell与TC shell
===========================

C shell与TC shell的语法和结构
-----------------------------

    * shbang行  #/bin/csh 或 #!/bin/tcsh. 脚本的第一行，通知内核使用哪种shell脚本执行脚本
    * 注释      # This is a comment. 注释可以从行的任意位置开始，到行的末尾结束
    * 通配符    shell中具有特殊意义的字符，又称元字符，如：*、?和[]用于文件名；!是历史命令符；<、>、>>、<&和|用于标准I/O重定向和管道。为了屏蔽这些字符的特殊意义，需要使用\或单引号。
    * 显示输出  echo 命令，输出通配符时，要使用转移字符\或单引号。如： echo "Hello to you\!"; echo \*;
    * 局部变量  局部变量的作用域被限定在当前shell中，当前脚本执行结束或shell退出后，它们将不再可用，使用set创建并为局部变量赋值。如：set variable_name = value; set name = "Tom Jones"
    * 全局变量  又称环境变量，在当前运行的shell中创建，可由该shell派生的任意进程使用，一旦脚本结束或者定义该全局变量的shell退出，将超出作用域。使用setenv定义和赋值，如：setenv VARIABLE_NAME = value; setenv PRINTER Shakespeare
    * 提取变量  使用$符号提取变量值，如：echo $variable_name; echo $name; echo $PRINTER;
    * 读取用户输入  使用$<从用户输入中读取一行并将它赋值给一个变量。如：echo "What is your name?"; set name = $<;
    * 参数      可以从命令行传递给脚本，通过位置参数或argv数组可以获取位置参数的值。如：scriptname arg1 arg2 arg3; echo $1 $2 $3; echo $*; echo $argv[1] $argv[2] $argv[3]; echo $argv[*]; echo $#argv # 参数个数
    * 数组      是由一对圆括号括起的，用空格隔开的一列词组组成的词表。内置的shift命令将词表左侧第一个单词移开。数组的索引从1开始。如：set word_list = ( word1 word2 word3 ); shift word_list # 将word1从词表中去掉; echo $word_list[1]; echo $word_list[*];
    * 命令替换  在脚本中使用某个命令的输出。反引号引起的内容，将会在脚本中当做命令去执行。如：set variable_name = `command`; echo "Today is `date`";
    * 算术运算  以 "@ varname"存储算术运算结果，T&TC shell只提供了整形算术运算。如：@ varname = 5 + 5; echo $varname;
    * 运算符    等式运算符：==、!=；关系运算符：>、>=、<、<=；逻辑运算符：&&、||、!。
    * 条件语句  ::

    # if 
    if ( expression ) then
        block of statements
    endif
    # if/else
    if ( expression ) then
        block of statements
    else 
        block of statements
    endif
    # if/else/else if
    if ( expression ) then
        block of statements
    else if ( expression ) then
        block of statements
    else if ( expression ) then
        block of statements
    else 
        block of statements
    endif
    # switch
    switch variable_name
        case constant1:
	    statements
	    breaksw
	case constant2:
	    statements
	case constant3:
	    statements
	default:
	    statements
    endsw

    switch ( "$color" )
        case blue:
	    echo $color is blue
	    breaksw
	case green:
	    echo $color is green
	    breaksw
	case red:
	case orange:
	    echo color is red or orange
	    breaksw
	default:
	    echo "not a valid color"
    endsw

    * 循环条件 有while和foreach两种，此外还有两个用来控制循环的命令break和continue。  ::

    while ( expression ) 
        block of statements
    end

    foreach variable ( word list )
        block of statements
    end
    foreach color ( red green blue )
        echo $color
    end

    * 文件测试  测试文件属性：-r: 当前用户是否可以读取该文件； -w： 是否可写；-x：是否可执行；-e：文件是否存在；-o：是否属于当前用户；-z：文件长度是否为0；-d：是否是一个目录；-f：是否是一个普通文件。 ::

    #!/bin/csh -f
    
    if ( -e file ) then # ( -d file ) ( ! -z file ) ( -r file && -w file )
        echo file exists # is a directory; is not of zero length; is readable and writable
    endif

C&TC shell的一个脚本实例
-------------------------------------

::

    #!/bin/csh -f
    # -f 快速启动选项，不执行默认情况下会执行的初始化文件 .cshrc文件
    # The Party Program -- Invitations to friends from the "guest" file

    set guestfile = ~/shell/guests
    if ( ! -e "$guestfile" ) then
        echo "$guestfile:t no-existent"
	exit 1  # 状态1指程序执行过程中出现了错误
    endif

    setenv PLACE "Sarotini's"
    @ Time = `date +%H` + 1
    set food = ( cheese crackers shrimp drinds "hot dogs" sandwiches )
    foreach person ( `cat $guestfile` )
        if ( $person =~ root ) continue
	mail -v -s "Party" $person << FINIS  # Start of here document  # 用<<定义here文档作为信件的内容
	Hi $person! Please join me at $PLACE for a party!
	Meet me at $Time o'clock.
	I'll bring the ice cream. Would you please bring $food[1] and 
	anything else you would like to eat? Let me know if you can 
	make it. Hope to see you soon.
	    Your pal,
	    ellie@`hostname`    # or 'uname -n'
FINIS   
       shift food
       if ( $#food == 0 ) then
           set food = ( cheese crackers shrimp drinks "hot dogs" sandwiches )
       endif
   end
   echo "Bye..."