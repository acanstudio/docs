关于Vim帮助手册
================

手册分两部分
-----------------

* 用户手册
* 参考手册

借助“跳转”功能可以更有效地使用帮助(CTRL-];CTRL-o)。

安装Vim之后
--------------

Vim默认运行在Vi兼容选项关闭模式，兼容模式会影响一些命令，如undo(多级撤销)；
<pre>
:set compatible? " 检查Vi兼容模式选项

" 使用默认的配置文件vimrc
:!cp -i $VIMRUNTIME/vimrc_example.vim ~/.vimrc " Unix
:!copy $VIMRUNTIME/vimrc_example.vim $VIM/_vimrc " Windows
:!copy $VIMRUNTIME/vimrc_example.vim $VIM/.vimrc " Amiga

" 查看Vim引导的配置文件列表
:scriptnames
</pre>

除了普通形态的Vim外，还有一种evim(easy vim)，这种形态下的Vim总是处于插入模式；

教程使用说明
--------------

* 阅读帮助手册
* 借助vimtutor(Vim教程)工具学习Vim命令

<pre>
>vimtutor // 启动vimtutor
>vimtutor fr // 选择语言
>vimtutor -g // 启用图形界面
</pre>

版权说明
-----------
