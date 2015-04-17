miniBufexplorer
================

[插件地址](http://www.vim.org/scripts/script.php?script_id=159)

在编程的时候不可能永远只编辑一个文件, 你肯定会打开很多源文件进行编辑,如果每个文件都打开一个vim进行编辑的话那操作起来将是多麻烦啊, 所以vim有buffer(缓冲区)的概念, 可以看vim的帮助:

:help buffer

vim自带的buffer管理工具只有:ls, :bnext, :bdelete 等的命令, 既不好用,又不直观. 现在隆重向你推荐一款vim插件(plugin): MiniBufExplorer

安装和配置

<pre>
# 通过Vundle安装
Bundle 'fholgado/minibufexpl.vim'
Bundle 'minibufexpl.vim'

" 基本配置
let g:miniBufExplMapWindowNavVim = 1 " 启用<C-h,j,k,l>切换到上下左右的窗口中去
let g:miniBufExplMapWindowNavArrows = 1 " 是用<C-箭头键>切换到上下左右窗口中
let g:miniBufExplMapCTabSwitchBufs = 1 
let g:miniBufExplModSelTarget = 1
let g:miniBufExplMapCTabSwitchWindows = 1  

" 解决FileExplorer窗口变小问题
let g:bufExplorerMaxHeight=30
let g:miniBufExplorerMoreThanOne=0
let g:miniBufExplForceSyntaxEnable = 1  
</pre>


重新启动vim, 当你只编辑一个buffer的时候MiniBufExplorer派不上用场, 当你打开第二个buffer的时候, MiniBufExplorer窗口就自动弹出来了。Vim上面那个狭长的窗口就是MiniBufExplorer窗口, 其中列出了当前所有已经打开的buffer, 当你把光标置于这个窗口时, 有下面几个快捷键可以用:

<pre>
<Tab>   向前循环切换到每个buffer名上
<S-Tab> 向后循环切换到每个buffer名上
<Enter> 在打开光标所在的buffer
d       删除光标所在的buffer

# 在命令模式下：

:bn   打开当前buffer的下一个buffer
:bp   打开当前buffer的前一个buffer
:b"num"   打开指定的buffer，"num"指的是buffer开始的那个数字
</pre>

MiniBufExplorer的命令
----------------------

<pre>
:MiniBufExplorer    " Open and/or goto Explorer
:CMiniBufExplorer   " Close the Explorer if it's open
:UMiniBufExplorer   " Update Explorer without naviting
:TMiniBufExplorer   " Toggle the Explorer window open and closed

# 如果你用gvim的话，MiniBufExplorer会出现多个窗口的烦人问题
# 一般都是用:CMiniBufExplorer命令把MiniBufExplorer窗口给close掉

# 如果在.vimrc中配置了mapleader
"Set mapleader
let mapleader = ","       

# 那么就可以在normal模式下用命令的缩写形式

:MiniBufExplorer    ,mbe
:CMiniBufExplorer   ,mbc
:UMiniBufExplorer   ,mbu
:TMiniBufExplorer  ,mbt
</pre>

配置命令详解
-------------

<pre>
# .vimrc中添加配置
let g:miniBufExplMapCTabSwitchBufs = 1

# 默认情况下该配置的功能是
# <C-Tab> 向前循环切换到每个buffer上,并在当前窗口打开
# <C-S-Tab> 向后循环切换到每个buffer上,并在当前窗口打开
</pre>

在~/.vim/plugin/minibufexpl.vim中可以发现，该配置对应的详细配置是

<pre>
if g:miniBufExplMapCTabSwitchBufs
  noremap <C-TAB>   :call <SID>CycleBuffer(1)<CR>:<BS>
  noremap <C-S-TAB> :call <SID>CycleBuffer(0)<CR>:<BS>
endif

# 可以通过编辑修改这里的代码，自定义快捷键如
if g:miniBufExplMapCTabSwitchBufs
  noremap <silent> <leader>n :call <SID>CycleBuffer(1)<CR>:<BS>
  noremap <silent> <leader>N :call <SID>CycleBuffer(0)<CR>:<BS>
endif

# 这样就可以用,n&nbsp; ,N 进行buffer切换(let mapleader = "," 我已在~/.vimrc中定义leader)
</pre>
