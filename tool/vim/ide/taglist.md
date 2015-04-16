 VIM IDE搭建（一）---ctags
分类： vi-gcc-make 2012-10-22 10:50 883人阅读 评论(0) 收藏 举报
vimidetagsnamespacesvariablesstructure


一.ctags 下载安装

    1.ctags 安装程序

    http://ctags.sourceforge.net/

    ctags-5.8.tar.gz


    2.安装ctags

      tar zxvf ctags-5.8.tar.gz 

      cd ctags-5.8

      make

      make install

二.ctags 参数设置

    1.查看ctags支持的语言：
      ctags --list-languages
      ctags --list-languages |grep [C,c]
      
    2.查看某种语言对应的文件扩展名
      ctags --list-maps
      ctags --list-maps |grep [C,c]

    3.添加某种语言对应的扩展名：以inl为扩展名的文件是c++文件
       ctags --langmap=c++:+.inl –R
       
    4.查看ctags 记录哪些语法元素
      ctags --list-kinds
      ctags --list-kinds=c 产看c语言的语法元素：
      [root@andes.com /andes/project/mkdemo/src]#ctags --list-kinds=c
      c  classes
      d  macro definitions
      e  enumerators (values inside an enumeration)
      f  function definitions
      g  enumeration names
      l  local variables [off]
      m  class, struct, and union members
      n  namespaces
      p  function prototypes [off]
      s  structure names
      t  typedefs
      u  union names
      v  variable definitions
      x  external and forward variable declarations [off] 

    5.根据4语法标记打开和关闭某个语法元素:
      打开某个语法元素：ctags -R --c-kinds=+x       
      关闭某个语法元素：ctags -R --c-kinds=-x
      x为某个语法简写，如4中描述的：c d e f g l

  

三.ctags插件与vim集成： 

    tags文件生成一般在工程根目录下生成一个tags文件，各个子目录中再生成各自tags文件。

    1.工程根目录下生成一个tags文件

    cd $PROJECT

    ctags -R *


    2.单个目录快捷生成tags文件

    "F12自动更新生成当前目录的tags文件

    set tags=tags

    set autochdir

    function! UpdateTagsFile()

    silent !ctags -R --fields=+ianS --extra=+q

    endfunction

    nmap <F12> :call UpdateTagsFile()<CR>


    3.设置tags寻找规则：

      "在$HOME/.vimrc 中添加如下两句

      set tags=tags;

      set autochdir


    注意第一个命令里的分号是必不可少的。这个命令让vim首先在当前目录里寻找 

    tags文件，如果没有找到tags文件，或者没有找到对应的目标，就到父目录中查 

    找，一直向上递归。因为tags文件中记录的路径总是相对于tags文件所在的路径， 

    所以要使用第二个设置项来改变vim的当前目录

    4.也可以手动加载tags文件

    vim中执行

    : set tags=tagapath

四. 将系统头文件生成tags文件加入到vim搜索中

    系统头文件加入tags索引，并且:
    [cpp] view plaincopy

        mkdir -p ~/.vim;  

    [cpp] view plaincopy

        ctags -I __THROW -I __attribute_pure__ -I __nonnull -I __attribute__ --file-scope=yes --langmap=c:+.h --languages=c,c++ --links=yes --c-kinds=+p --c++-kinds=+p --fields=+iaS --extra=+q  -f ~/.vim/systags /usr/include/* /usr/include/sys/* /usr/include/bits/*  /usr/include/netinet/* /usr/include/arpa/* /usr/local/mysql/include/*  

    其关键是-I __THROW部分和--c-kinds=+p部分。设置-I后，ctags会在处理文件时，就会忽略-I后面写出来的符号; 避免带 __THROW函数无法检索到;

    而--c-kinds=+p 则告诉ctags需要为函数原型的声明也生成tag;

    --langmap=c:+.h表示.h视为c文件而不是c++文件。

    .vimrc 添加：
    set tags+=~/.vim/systags



 VIM IDE搭建（二）---taglist
分类： vi-gcc-make 2012-10-22 11:01 877人阅读 评论(0) 收藏 举报
vimidefoldfiletagsmenu


一.下载安装taglist 

    1.安装 taglist 为 VIM插件，直接解压拷贝到相关目录即可

    http://www.vim.org/scripts/script.php?script_id=273

    下载taglist_45.zip 后，将其解压 将doc plugin 目录分别copy 到$HOME/.vim 目录下。


    2.taglist 的配置需要在$HOME/.vimrc进行设置。 


      1>增加调用taglist的快捷方式tl

        nmap tl :TlistToggle<cr>


    3.跳转

        函数来回跳转

    　通过ctrl-]来执行跳转到定义，通过Ctrl+T来跳转回来


二. vim配置文件.vimrc中taglist 配置

    1.本节所用命令的帮助入口：

    :help helptags

    :help taglist.txt 


    2.使用下面的命令生成taglist帮助标签

    :helptags ~/.vim/doc


    3.下面介绍常用的taglist配置选项，你可以根据自己的习惯进行配置： 


    - Tlist_Ctags_Cmd：该选项用于指定你的Exuberant ctags程序的位置，如果它没在你PATH变量所定义的路径中，需要使用此选项设置一下

    - Tlist_Show_One_File：如果你不想同时显示多个文件中的tag，设置Tlist_Show_One_File为1。缺省为显示多个文件中的tag；

    - Tlist_Sort_Type：设置Tlist_Sort_Type为”name”可以使taglist以tag名字进行排序，缺省是按tag在文件中出现的顺序进行排序。按tag出现的范围（即所属的namespace或class）排序，已经加入taglist的TODO List，但尚未支持；

    - Tlist_Exit_OnlyWindow：如果你在想taglist窗口是最后一个窗口时退出VIM，设置Tlist_Exit_OnlyWindow为１；

    - Tlist_Use_Right_Window：如果你想taglist窗口出现在右侧，设置Tlist_Use_Right_Window为１。缺省显示在左侧。

    - Tlist_Show_Menu：在gvim中，如果你想显示taglist菜单，设置Tlist_Show_Menu为１。你可以使用Tlist_Max_Submenu_Items和Tlist_Max_Tag_Length来控制菜单条目数和所显示tag名字的长度；

    - Tlist_Use_SingleClick：缺省情况下，在双击一个tag时，才会跳到该tag定义的位置，如果你想单击tag就跳转，设置Tlist_Use_SingleClick为１；

    - Tlist_Auto_Open：如果你想在启动VIM后，自动打开taglist窗口，设置Tlist_Auto_Open为1；

    - Tlist_Close_On_Select：如果你希望在选择了tag后自动关闭taglist窗口，设置Tlist_Close_On_Select为1；

    - Tlist_File_Fold_Auto_Close：当同时显示多个文件中的tag时，设置Tlist_File_Fold_Auto_Close为１，可使taglist只显示当前文件tag，其它文件的tag都被折叠起来。

    - Tlist_GainFocus_On_ToggleOpen：在使用:TlistToggle打开taglist窗口时，如果希望输入焦点在taglist窗口中，设置Tlist_GainFocus_On_ToggleOpen为1；

    - Tlist_Process_File_Always：如果希望taglist始终解析文件中的tag，不管taglist窗口有没有打开，设置Tlist_Process_File_Always为1；

    - Tlist_WinHeight，Tlist_WinWidth： Tlist_WinHeight和Tlist_WinWidth可以设置taglist窗口的高度和宽度。Tlist_Use_Horiz_Window为１设置taglist窗口横向显示；

     

    4.可以用“:TlistOpen”打开taglist窗口，用“:TlistClose”关闭taglist窗口。或者使用“:TlistToggle”在打开和关闭间切换。

      可以自定义快捷键，在我的.vimrc中定义了下面的映射，使用“tl”键就可以打开/关闭taglist窗口：

      nmap tl :TlistToggle<cr>

三.taglist窗口快捷键

在taglist窗口中，可以使用下面的快捷键：

    <CR>          跳到光标下tag所定义的位置，用鼠标双击此tag功能也一样

    o             在一个新打开的窗口中显示光标下tag

    <Space>       显示光标下tag的原型定义

    u             更新taglist窗口中的tag

    s             更改排序方式，在按名字排序和按出现顺序排序间切换

    x             taglist窗口放大和缩小，方便查看较长的tag

    +             打开一个折叠，同zo

    -             将tag折叠起来，同zc

    *             打开所有的折叠，同zR

    =             将所有tag折叠起来，同zM

    [[            跳到前一个文件

    ]]            跳到后一个文件

    q             关闭taglist窗口

    <F1>          显示帮助 

四. .vimrc中ctags 和taglist配置项：

[cpp] view plaincopy

    """"""""""""""""""""""""""""""  
    " Taglist and Ctags  
    """"""""""""""""""""""""""""""  
    let Tlist_Ctags_Cmd = '/usr/local/bin/ctags'"指定ctags程序的位置  
    let Tlist_Show_One_File = 1            "不同时显示多个文件的tag，只显示当前文件的  
    let Tlist_Exit_OnlyWindow = 1          "如果taglist窗口是最后一个窗口，则退出vim  
    let Tlist_Use_Right_Window = 1         "在右侧窗口中显示taglist窗口  
    let Tlist_File_Fold_Auto_Close=1       "只显示当前文件tag，其它文件的tag都被折叠起来  
      
      
    "set tags=/andes/project/mkdemo/src/tags  
    "F12生成/更新tags文件  
    set tags=tags;  
    set autochdir  
      
      
    "设置F12快捷键 自动生成当前目录tags文件  
    function! UpdateTagsFile()  
    silent !ctags -R --fields=+ianS --extra=+q  
    endfunction  
    nmap <F12> :call UpdateTagsFile()<CR>  
      
      
    "设置taglist窗口打开快捷键tl  
    nmap tl :TlistToggle<cr>  





