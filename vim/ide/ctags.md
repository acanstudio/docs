ctags
========

使用taglist之前，需要安装ctags可执行程序

<pre>
# Linux系统下ctags的安装
cd /opt/soft/sourcepackage/
wget http://ctags.sourceforge.net/ctags-*.tar.gz
tar zxvf ctags-* -C /opt/soft/source
cd /opt/soft/source/ctages*
make ; make install

# Windows下载相应的windows发布包，解压后将ctags.exe文件复制到任一系统环境PATH变量的目录下即可
# 这里将ctags.exe复制到$VIM/vimfiles/bin目录下，并将该目录加入系统环境变量PATH
</pre>

ctags命令的用法

<pre>
# 查看ctags支持的语言：
$ ctags --list-languages
$ ctags --list-languages |grep [C,c]

# 查看某种语言对应的文件扩展名
$ ctags --list-maps
$ ctags --list-maps |grep [C,c]

# 添加某种语言对应的扩展名：以inl为扩展名的文件是c++文件
$ ctags --langmap=c++:+.inl -R
 
# 查看ctags记录哪些语法元素
$ ctags --list-kinds
# 产看c语言的语法元素：
$ ctags --list-kinds=c 

# 根据语法标记打开和关闭某个语法元素:
# 打开某个语法元素：
$ ctags -R --c-kinds=+x       
# 关闭某个语法元素：
$ ctags -R --c-kinds=-x
x为某个语法简写，是ctags --list-kinds=中描述的语法元素
</pre>

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
