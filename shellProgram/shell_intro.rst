shell 脚本简介
=============

C shell和TC shell依照的是C语言的语法；Bourne shell则是基于老式的编程语言Algol；Bash shell和Korn shell起源于Bourne shell，它们更是Bourne shell
和C shell的结合。

UNIX/Linux的基本命令是熟练使用shell的基础，此外，掌握UNIX/Linux提供的grep、awk、sed等强大的工具，可以使shell的作用发挥到极致。

shell作为编程语言时，命令和shell程序可以通过编辑器输入并保存为文件，即shell脚本。为shell脚本赋予执行权限（chmod +x shellfile）后，可以像使用可执行程序那样在命令行执行该脚本，shell脚本
执行时，shell程序从shell脚本文件逐行读取并执行。shell脚本不需要编译执行，而是解释执行。这样会在执行效率上差一些，但shell脚本的优势是容易编写。用于简单任务的操作自动化方面，有非常强的
实用价值。
