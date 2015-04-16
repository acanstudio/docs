winmanager（文件管理器）安装
=============================

插件地址：http://www.vim.org/scripts/script.php?script_id=95
可以下载后手动安装，建议使用vundle插件管理工具安装

winmanager的相关配置

<pre>
" 设置winmanager
" 设置界面分割
let g:winManagerWindowLayout = "TagList|FileExplorer"
"设置winmanager的宽度，默认为25
let g:winManagerWidth = 30
"定义打开关闭winmanager快捷键为F8
nmap <silent> <F8> :WMToggle<cr>
"在进入vim时自动打开winmanager
let g:AutoOpenWinManager = 1

" F8键可以打开或关闭winmanager，还可以通过命令':WMToggle'实现同样的功能
</pre>

 VIM IDE搭建（三）--WinManager
分类： vi-gcc-make 2012-10-22 11:11 3023人阅读 评论(0) 收藏 举报
vimide

一.WinManager下载安装

    １．WinManager是vim插件，先下载：
    　　http://www.vim.org/scripts/script.php?script_id=95
    　　将解压后的文件分别放到：~/.vim/doc/ 　~/.vim/plugin　目录下

    2.编辑.vimrc ，增加对WinManager的支持
      let g:winManagerWindowLayout='FileExplorer|TagList'
      nmap wm :WMToggle<cr>
      
    3.vim 打卡文件
      输入 wm 即可调出WinManager
      再次输入wm即可关闭WinManager
      
      
    4.窗口切换：
      通过Ctrl+W+w 进行窗口的切换
      通过Ctrl+W,然后箭头进行上下左右窗口切换
      通过Ctrl+W,然后JKL;(vi键)进行上下左右窗口切换
      
    5.taglist 窗口的折叠
      Ctrl+* 或s 或-> 或<- 箭头打开

      - 折叠

二.vim 配置文件.vimrc中配置WinManager

[cpp] view plaincopy

    """"""""""""""""""""""""""""""  
    " WinManager Config   
    """"""""""""""""""""""""""""""  
      
    let g:winManagerWindowLayout='FileExplorer|TagList'  
      
    nmap wm :WMToggle<cr>  
    map <silent> <F9> :WMToggle<cr>
