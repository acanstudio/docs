Vim帮助手册
============

本章介绍 vim 的手册本身。读者可以通过本章来了解本手册是如何解释 Vim 命令的。

手册的两个部分
----------------

Vim 的手册分成两个部分：

1. 用户手册 面向任务的使用说明书，由简入繁，能象书一样从头读到尾。

2. 参考手册 详细描述 Vim 的每一个命令的详细资料。

安装了Vim之后
--------------

本手册还经常假定 Vim 运行在 Vi 兼容选项关闭的模式下。对大部分命令而言，这无关紧要，但有时却是很重要的。例如，多级撤销 (undo) 功能就是这样。有一个简单的方法可以保证你的 Vim 设置是正常的 － 拷贝 Vim 自带的 vimrc 范例。在 Vim 里进行操作，你就无需知道这个文件在什么地方。不同系统的操作方法分别如下：

<pre>
# 使用vimrc维护Vim的配置
# 对于 Unix
:!cp -i $VIMRUNTIME/vimrc_example.vim ~/.vimrc
# 对于 MS-DOS, MS-Windows, OS/2
:!copy $VIMRUNTIME/vimrc_example.vim $VIM/_vimrc

# 检查一下compatible状态
:set compatible?
# 如果 Vim 报告 "nocompatible"，则一切正常；
# 如果返回 "compatible" 就有问题。
# 这时你需要检查一下为什么会出现这个问题。一种可能是 Vim 找不到你的配置文件。
# 用如下命令检查一下：
:scriptnames
</pre>

如果你的文件不在列表中，检查一下该文件的位置和文件名。如果它在列表中，那么一定是还有某个地方把 'compatible' 选项设回来了。

备注:
本手册是关于普通形态的 Vim 的，Vim 还有一种形态叫 "evim" (easy vim)，那也是 Vim，不过被设置成 "点击并输入" 风格，就像 Notepad 一样。它总是处于 "插入" 模式，感觉完全不同于通常形态下的 Vim。由于它比较简陋，将不在本手册中描述。

教程使用说明 
--------------

除了阅读文字 (烦！)，你还可以用 vimtutor (Vim 教程) 学习基本的 Vim 命令，这是一个 30 分钟的教程，它能教会你大部分基本的 Vim 功能。

在 Unix 中，如果 Vim 安装正常，你可以从命令行上运行以下命令：
<pre>
$ vimtutor
# 在 MS-Windows 中，你可以在 Program/vim 菜单中找到这个命令，或者在 $VIMRUNTIME目录中运行 vimtutor.bat 程序。
 
# 这个命令会建立一份教程文件的拷贝，你可以任意修改它而不用担心会损坏原始的文件。
# 这个教材有各种语言的版本。要检查你需要的版本是否可用，使用相应语言的双字母缩写。例如，对于法语版本：
$ vimtutor fr

# 在 Unix 上，如果你更喜欢 Vim 的 GUI 版本，
$ gvimtutor
$ vimtutor -g
</pre>

在非 Unix 系统中，你还要做些工作：

<pre>
# 拷贝教程文件。你可以用 Vim 来完成这个工作 (Vim 知道文件的位置)
$ vim -u NONE -c 'e $VIMRUNTIME/tutor/tutor' -c 'w! TUTORCOPY' -c 'q'
# 这个命令在当前路径下建立一个 "TUTORCOPY" 的文件。要使用其它语言的版本，
# 在文件名后加上双字母的语言缩写作为扩展名。对于法语：
$ vim -u NONE -c 'e $VIMRUNTIME/tutor/tutor.fr' -c 'w! TUTORCOPY' -c 'q'

# 用 Vim 编辑这个被拷贝的文件
$ vim -u NONE -c "set nocp" TUTORCOPY
# 其它几个参数用于保证 Vim 使用正确的模式启动。

# 学习完成后删除临时拷贝文件
$ del TUTORCOPY
</pre>
