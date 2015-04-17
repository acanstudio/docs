 VIM IDE搭建（五）--cscope
分类： vi-gcc-make 2012-10-22 11:29 599人阅读 评论(0) 收藏 举报
vimide数据库flexdialogdatabase

一.下载安装

    1.检测是否
    [root@andes.com /andes/project/mkdemo/src/isql]#vim --version |grep cscope
    +cryptv +cscope +cursorshape +dialog_con +diff +digraphs -dnd -ebcdic 
    2.下载:cscope 源码
      http://cscope.sourceforge.net/
      http://sourceforge.net/projects/cscope/files/cscope/
    3. 安装：
      如未安装flex，需要先安装flex
            yum install flex
            ./configure
            make
            make install
    上述不行，使用下面命令：
            make distclean
            ./configure --with-flex

            make

二. 生成cscope文件列表和数据库：


生成脚本：

[cpp] view plaincopy

    #!/bin/bash  
    echo "Begin TO Generage cscope.files..."  
    find . -name "*.h" -o -name "*.c" -o -name "*.cc" > cscope.files  
    ls -l  cscope.files  
      
    echo "Begin TO Generage cscope.out"  
    cscope -Rbq -i cscope.files -I /usr/local/mysql/include  
    ls -l  cscope.*  
      
    echo "Begin TO Generage tags"  
    ctags -R  
    ls -l tags   
      
    echo "Complete Successfully"  


    cscope 选项说明：-I选项将自己要包含的头文件添加进去。

    -R: 在生成索引文件时，搜索子目录树中的代码

    -b: 只生成索引文件，不进入cscope的界面

    -k: 在生成索引文件时，不搜索/usr/include目录

    -q: 生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度

    -i: 如果保存文件列表的文件名不是cscope.files时，需要加此选项告诉cscope到哪儿去找源文件列表。可以使用“-”，表示由标准输入获得文件列表。

    -I dir: 在-I选项指出的目录中查找头文件

    -u: 扫描所有文件，重新生成交叉索引文件

    -C: 在搜索时忽略大小写

    -P path: 在以相对路径表示的文件前加上的path，这样，你不用切换到你数据库文件所在的目录也可以使用它了。

    -I 选项非常有用，用于将其他开发库头文件导入到cscope数据库，方便产看，开发；


三.cscope 与vim 集成设置：

需要在.vimrc中添加如下配置

[cpp] view plaincopy

    """"""""""""""""""""""""""""""  
    " Cscope Config  
    """"""""""""""""""""""""""""""  
    set cscopequickfix=s-,c-,d-,i-,t-,e- "quickfix支持  
    "快捷键设置  
    nmap css :cs find s <C-R>=expand("<cword>")<CR><CR>  
    nmap csg :cs find g <C-R>=expand("<cword>")<CR><CR>  
    nmap csc :cs find c <C-R>=expand("<cword>")<CR><CR>  
    nmap cst :cs find t <C-R>=expand("<cword>")<CR><CR>  
    nmap cse :cs find e <C-R>=expand("<cword>")<CR><CR>  
    nmap csf :cs find f <C-R>=expand("<cfile>")<CR><CR>  
    nmap csi :cs find i ^<C-R>=expand("<cfile>")<CR>$<CR>  
    nmap csd :cs find d <C-R>=expand("<cword>")<CR><CR>  

[cpp] view plaincopy

    "cscope数据库文件添加  
    if has("cscope")  
        "指定用来执行 cscope 的命令  
        set csprg=/usr/local/bin/cscope  
        "先搜索tags标签文件，再搜索cscope数据库  
        set csto=1  
        "使用|:cstag|(:cs find g)，而不是缺省的:tag  
        set cst  
        "不显示添加数据库是否成功   
        set nocsverb  
        " add any database in current directory  
        if filereadable("/andes/project/mkdemo/src/cscope.out")  
            cs add /andes/project/mkdemo/src/cscope.out /andes/project/mkdemo/src  
        endif  
        "显示添加成功与否  
        set csverb                               
    endif   


四.cscope 和 quickfix使用

    :cs find 的选项

    0或则S：查找本符号

    1或则G：查找本定义

    2或则D：查找本函数调用的函数

    3或则C：查找调用本函数的函数

    4或则T：查找本字符串

    6或则E：查找本EGREP模式

    7或则F：查找本文件

    8或则I：查找包含本文件的文件


    quickfix用法：

    1.调出办法  :cw   
    2.关闭办法 再一次:cw   或者在激活状态下:q
    3.显示列表移动
    :cn // 切换到下一个结果
    :cp // 切换到上一个结果



