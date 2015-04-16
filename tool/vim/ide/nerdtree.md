NerdTree插件安装
=================

   前面我安装了winmanager插件，但是感觉那个用起来不是很爽。今天无意间又发现一个更好的文件管理器插件—NerdTree。先上图，红色方框标记的就是NerdTree区域了：

   

   是不是挺炫？因为我又换了一种配色方案，并且设置了字体 set guifont=Consolas:h11。这种新的配色方案名字叫freya，有兴趣的可以去这里下：http://www.vim.org/scripts/script.php?script_id=1651 安装方法可以参考同系列博客第一篇。

   言归正传，现在让我们一起来安装NerdTree吧，从 http://www.vim.org/scripts/script.php?script_id=1658  下载，然后解压，将解压得到的plugin和doc文件夹与~/Vim/Vim73/目录下的同名文件夹合并。然后往_vimrc文件中增加下面配置代码：

" 设置NerdTree
map <F3> :NERDTreeMirror<CR>
map <F3> :NERDTreeToggle<CR>

   按F3即可显示或隐藏NerdTree区域了。

 

附：http://www.cnblogs.com/chijianqiang/archive/2012/11/06/vim-3.html

NERDTree提供了丰富的键盘操作方式来浏览和打开文件，我简单介绍一些常用的快捷键：

和编辑文件一样，通过h j k l移动光标定位
o 打开关闭文件或者目录，如果是文件的话，光标出现在打开的文件中
go 效果同上，不过光标保持在文件目录里，类似预览文件内容的功能
i和s可以水平分割或纵向分割窗口打开文件，前面加g类似go的功能
t 在标签页中打开
T 在后台标签页中打开
p 到上层目录
P 到根目录
K 到同目录第一个节点
J 到同目录最后一个节点
m 显示文件系统菜单（添加、删除、移动操作）
? 帮助
q 关闭

想了解更多操作方式，可以通过? 查看详细的帮助信息。
