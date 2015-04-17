winmanager（文件管理器）安装
=============================

[插件地址](http://www.vim.org/scripts/script.php?script_id=95)

安装和配置

<pre>
# 通过vundle安装
Bundle 'winmanager'

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

# 窗口切换：
Ctrl+W+w
Ctrl+W,然后箭头进行上下左右窗口切换
Ctrl+W,然后JKL;(vi键)进行上下左右窗口切换
</pre>
