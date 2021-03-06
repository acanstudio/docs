VIM键盘映射 (Map)
===================

使用:map命令，可以将键盘上的某个按键与Vim的命令绑定起来。例如

<pre>
# 使用以下命令，可以通过F5键将单词用花括号括起来：
:map <F5> i{e<Esc>a}<Esc>

# 其中：i{将插入字符{，然后使用Esc退回到命令状态；接着用e移到单词结尾，a}增加字符}，最后退至命令状态。
# 在执行以上命令之后，光标定位在一个单词上（例如amount），按下F5键，这时字符就会变成{amount}的形式。
</pre>

不同模式下的键盘映射

使用下表中不同形式的map命令，可以针对特定的模式设置键盘映射：

 
Command
命令 	Normal
  常规模式   	Visual
可视化模式 	Operator Pending
运算符模式 	Insert Only
插入模式 	Command Line
命令行模式
:map 	y 	y 	y 	  	 
:nmap 	y 	  	  	  	 
:vmap 	  	y 	  	  	 
:omap 	  	  	y 	  	 
:map! 	  	  	  	y 	y
:imap 	  	  	  	y 	 
:cmap 	  	  	  	  	y
键盘映射实例

使用以下命令，可以在Normal Mode和Visual/Select Mode下，利用Tab键和Shift-Tab键来缩进文本：

nmap <tab> V>
nmap <s-tab> V<
vmap <tab> >gv
vmap <s-tab> <gv

使用以下命令，指定F10键来新建标签页：

:map <F10> <Esc>:tabnew<CR>

其中：<Esc>代表Escape键；<CR>代表Enter键；而功能键则用<F10>表示。首先进入命令行模式，然后执行新建标签页的:tabnew命令，最后返回常规模式。

同理：对于组合键，可以用<C-Esc>代表Ctrl-Esc；使用<S-F1>表示Shift-F1。对于Mac用户，可以使用<D>代表Command键。

注意：Alt键可以使用<M-key>或<A-key>来表示。

关于键盘符号的详细说明，请使用:h key-notation命令查看帮助信息。

我们还可以针对函数设置键盘映射。 例如，将以下代码加入.vimrc文件，就可以利用快捷键，来打开或关闭针对搜索结果的高亮显示。
查看键盘映射

使用:map命令，可以列出所有键盘映射。其中第一列标明了映射在哪种模式下工作：

标记	模式
<space>	常规模式，可视化模式，运算符模式
n	常规模式
v	可视化模式
o	运算符模式
!	插入模式，命令行模式
i	插入模式
c	命令模式

使用:map!命令，则只列出插入和命令行模式的映射。而:imap，:vmap，:omap，:nmap命令则只是列出相应模式下的映射。
取消键盘映射

如果想要取消一个映射，可以使用以下命令：

:unmap <F10>

注意：必须为:unmap命令指定一个参数。如果未指定任何参数，那么系统将会报错，而不会取消所有的键盘映射。

针对不同模式下的键盘映射，需要使用与其相对应的unmap命令。例如：使用:iunmap命令，取消插入模式下的键盘映射；而取消常规模式下的键盘映射，则需要使用:nunmap命令。

如果想要取消所有映射，可以使用:mapclear命令。请注意，这个命令将会移除所有用户定义和系统默认的键盘映射。
参考

    http://yyq123.blogspot.com/2010/12/vim-map.html
    Vim按键映射

使用set命令

set pastetoggle=<F9>

使用imap命令

用Esc退出插入模式很麻烦，可以将另外的键映射到这个键上，如下,将Ctrl-i映射为Esc

imap <C-I> <Esc>

== the end ==
