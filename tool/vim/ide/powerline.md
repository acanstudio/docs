PowerLine安装
==============

     PowerLine是一个增强的Vim状态栏插件。当Vim处于NORMAL、INSERT、BLOCK等状态时，状态栏会呈现不同的颜色，同时状态栏还会显示当前编辑文件的格式（uft-8等）、文件类型（java、xml等）和光标位置等。

     从https://github.com/Lokaltog/vim-powerline下载PowerLine插件，将文件解压得到的autoload、doc、plugin和fontpatcher文件夹放到~/Vim/Vim73/bundler目录下，往_vimrc文件中增加下面配置即可：

"powerline
 set guifont=PowerlineSymbols\ for\ Powerline
 set nocompatible
 set t_Co=256
 let g:Powerline_symbols = 'fancy'
