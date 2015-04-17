NerdTree插件安装
=================

[插件地址](http://www.vim.org/scripts/script.php?script_id=1658)

NERDTree是Vim最常用的插件之一，可以在Vim运行时显示目录和文件结构，操作起来非常方便，你可以在手不离开键盘的情况下快速浏览文件，并在文件和文件夹之间进行切换。

安装和配置

<pre>
# 通过vundle安装
Bundle 'scrooloose/nerdtree'
Bundle 'scrooloose/nerdcommenter.git'

" 设置NerdTree，通过F3启用和隐藏NerTree
map <F3> :NERDTreeMirror<CR>
map <F3> :NERDTreeToggle<CR>
</pre>


NERDTree提供了丰富的键盘操作方式来浏览和打开文件，下面是一些常用的快捷键

<pre>
h j k l # 移动光标定位
o       # 打开关闭文件或者目录，如果是文件的话，光标出现在打开的文件中
go      # 效果同上，不过光标保持在文件目录里，类似预览文件内容的功能
i s     # 可以水平分割或纵向分割窗口打开文件，前面加g类似go的功能
t       # 在标签页中打开
T       # 在后台标签页中打开
p       # 到上层目录
P       # 到根目录
K       # 到同目录第一个节点
J       # 到同目录最后一个节点
m       # 显示文件系统菜单（添加、删除、移动操作）
?       # 帮助
q       # 关闭
</pre>
