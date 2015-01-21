Korn shell
===========================

Korn shell的语法和结构
-----------------------

    * shbang行  #!/bin/ksh  脚本的第一行，通知内核使用哪种shell脚本执行脚本
    * 注释      # This is a comment. 注释可以从行的任意位置开始，到行的末尾结束
    * 通配符    shell中具有特殊意义的字符，又称元字符，如：*、?和[]用于文件名；!是历史命令符；<、>、>>、2>和|用于标准I/O重定向和管道。为了屏蔽这些字符的特殊意义，需要使用\或单引号。
    * 显示输出  echo 命令，也可以使用内置的print命令，输出通配符时，要使用转移字符\或单引号。如： echo "Hello to you\!"; echo \*; print "How are you?";
    * 局部变量  局部变量的作用域被限定在当前shell中，当前脚本执行结束或shell退出后，它们将不再可用，可以直接定义变量并为其赋值，也可以使用typeset声明。如：variable_name=value; name="Tom Jones"; typeset variable_name=value;
    * 全局变量  又称环境变量，在当前运行的shell中创建，可由该shell派生的任意进程使用，一旦脚本结束或者定义该全局变量的shell退出，将超出作用域。使用export定义全局变量并可以为其赋值。如：export VARIABLE_NAME = value; export PRINTER Shakespeare;
    * 提取变量  使用$符号提取变量值，如：echo $variable_name; echo $name; echo $PRINTER;
    * 读取用户输入  使用read命令，read命令从用户输入中读取一行并将它赋值给该命令右侧的一个或多个变量，read命令可以接受一个或多个变量名，每个变量名被赋予一个单词。如：echo "What is your name"; read name; read name1 name2 ...;
    * 参数      可以从命令行传递给脚本，通过位置参数可以获取位置参数的值。还可以使用set -A创建数组。如：scriptname arg1 arg2 arg3; echo $1 $2 $3; echo $*; echo $# # 参数个数; 
    * 数组      使用位置参数创建数组，set命令后跟一系列的单词，每个词可以通过位置参数访问，最多允许使用9个位置。内置的shift命令将词表左侧第一个单词移开。数组的索引从1开始。如：set word1 word2 word3 ; shift # 将word1从词表中去掉; echo $1; echo $2; echo $*;set -A array_name word1 word2 ...; print ${array_name[0]}; ${array_name[1]} = newvalue;
    * 命令替换  在脚本中使用某个命令的输出。反引号引起的内容，将会在脚本中当做命令去执行，还有一种通过$( )方式实现命令替换的功能。如：set variable_name = `command`; echo "Today is `date`"; echo $( ls );
    * 算术运算  Korn shell支持整型算术运算，但要使用typeset -i把变量设置为整数，否则只能使用(())，即let命令实现运算。如：typeset -i num1; typeset -i num2; num2 = $num1 + $num2; (( n=5 + 5 ));
    * 运算符    Bourne shell使用内置的test命令运算符来测试数字和字符串。等式运算符：=、!=针对字符串，==、!=针对数字；关系运算符：>、>=、<、<=；逻辑运算符：&&、||、!。
    * 条件语句  if结构后跟着一个由圆括号括起的表达式，关键字then放在闭括号后，[[]]用于条件表达式，兼容[]条件表达式兼容。 ::

    # if 
    if command
    then
        block of statements
    fi
    if [[ string expression ]] 
    then
        block of statements
    fi
    if (( number expression ))
    then
        block statements
    fi
    # if/else
    if command
    then
        block of statements
    else
        block of statements
    fi
    if [[ expression ]]
    then
        block of statements
    else 
        block of statements
    fi
    if (( numeric expression ))
    then
        block of statements
    else
        block of statements
    fi
    # if/else/else if
    if command
    then
        block of statements
    elif command
    then
        block of statements
    elif command
    then
        block of statements
    else 
        block of statements
    fi
    if [[ expression ]] 
    then
        block of statements
    elif [[ expression ]]
    then
        block of statements
    elif [[ expression ]]
    then
        block of statements
    else 
        block of statements
    fi

    if (( number expresstion ))
    then
        block of statements
    elif (( number expression ))
    then
        block of statements
    elif (( number expression ))
    then
        block of statements
    else
        block of statements
    fi

    # case
    case variable_name in
        pattern1)
	    statements
	    ;;
	pattern2)
	    statements
	    ;;
	pattern3)
	    statements
	    ;;
	#*) default value
	    #;;
    esac
    case "$color" in
        blue)
	    echo $color is blue
	    ;;
	green)
	    echo $color is green
	    ;;
	red|orange)
	    echo $color is red or orange
	    ;;
	#*) echo "Not a color" # default
    esac

    * 循环条件 有while、until、for和select四种，此外还有两个用来控制循环的命令break和continue。  ::

    while command
    do
        block of statements
    done
    while [[ expression ]]
    do
        block of statements
    done
    while (( number expression ))
    do 
        block of statements
    done

    until command
    do
        block of statements
    done
    until [[ expression ]]
    do
        block of statements
    done
    until (( number expression ))
    do 
        block of statements
    done

    for 
    variable in word1 word2 word3 ...
    do
        block of statements
    done

    select variable in word_list
    do
        block of statements
    done
    PS3="Select an item from the menu"
    for item in blue red green
    do
        echo $item
    done

    * 文件测试  测试文件属性：-r: 当前用户是否可以读取该文件； -w： 是否可写；-x：是否可执行；-s：文件长度是否不为空；-d：是一个目录；-a：存在且不是目录。 ::

    #!/bin/csh -f
    
    if ( -f file ) then # ( -a file ) ( -s file ) ( -r file && -w file )
        echo file exists # is a directory; is not of zero length; is readable and writable
    endif

    * 函数    允许定义一段shell代码当为一个函数，然后以函数的形式通过函数名反复使用这段代码。可以使用function关键字，也可以不使用。 ::

    function_name() {
        block of code
    }
    function lister() {
        echo Your present working directory is `pwd`
	echo Your files are:
	ls
    }

Korn shell的一个脚本实例
-------------------------------------

::

    #!/bin/sh 
    # The Party Program -- Invitations to friends from the "guest" file

    guestfile=~/shell/guests
    if [[ ! -a "$guestfile" ]]
    then
        echo "${guestfile##*} no-existent"
	exit 1  # 状态1指程序执行过程中出现了错误
    fi

    export PLACE "Sarotini's"; 
    (( Time=$$(date +%H) + 1 ))
    set -A foods cheese crackers shrimp drinds "hot dogs" sandwiches
    typeset -i n=0
    for person in $(< $guestfile)
    do
        if [[ $person =~ root ]]; then # then通常另起一行，如果它前面是;则可以与if位于同一行
	    continue
	else
	    mail -v -s "Party" $person <<- FINIS  # Start of here document  # 用<<定义here文档作为信件的内容
	    cat <<-FINIS # 允许测试脚本想屏幕输出结果，方便调试
       	    Hi ${person}! Please join me at $PLACE for a party!
	    Meet me at $Time o'clock.
	    I'll bring the ice cream. Would you please bring ${foods[$n]} and 
	    anything else you would like to eat? Let me know if you can 
	    make it. Hope to see you soon.
	        Your pal,
	        ellie@`hostname`    # or 'uname -n'
FINIS   
           n=n+1
           shift food
           if (( ${#foods[*] == 0 ))
	   then
               set -A foods cheese crackers shrimp drinks "hot dogs" sandwiches
           fi
       fi
   done
   echo "Bye..."