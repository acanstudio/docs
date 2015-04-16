 VIM IDE搭建（四）--miniBufexplorer
分类： vi-gcc-make 2012-10-22 11:15 2413人阅读 评论(0) 收藏 举报
vimide
1.miniBufexplorer 为vim插件，直接解压拷贝到相关目录即可，
  下载官方地址：
  http://www.vim.org/scripts/script.php?script_id=159
  minibufexpl.vim
  将minibufexpl.vim 拷贝到 $HOME/.vim/plugin目录下
  
2.$HOME/.vimrc 中添加如下配置
let g:miniBufExplMapWindowNavVim = 1 
let g:miniBufExplMapWindowNavArrows = 1 
let g:miniBufExplMapCTabSwitchBufs = 1 
let g:miniBufExplModSelTarget = 1


3.解决FileExplorer窗口变小问题 须在$HOME/.vimrc中添加：
"解决FileExplorer窗口变小问题
let g:bufExplorerMaxHeight=30

let g:miniBufExplorerMoreThanOne=0


相关设置：

[cpp] view plaincopy

    """"""""""""""""""""""""""""""  
    " miniBufexplorer Config  
    """"""""""""""""""""""""""""""  
    let g:miniBufExplMapWindowNavArrows = 1  
    let g:miniBufExplMapWindowNavVim = 1  
    let g:miniBufExplMapCTabSwitchWindows = 1  
    "let g:miniBufExplMapCTabSwitchBufs = 1   
    let g:miniBufExplModSelTarget = 1  
      
    "解决FileExplorer窗口变小问题  
    let g:miniBufExplForceSyntaxEnable = 1  
    let g:miniBufExplorerMoreThanOne=2  
    

    
MiniBufExplorer插件的使用

    博客分类： vim 

vimMiniBufExplorer 

 

快速浏览和操作Buffer -- 插件: MiniBufExplorer

 

下载地址 [http://www.vim.org/scripts/script.php?script_id=159]

版本     6.3.2

安装     将下载的 minibufexpl.vim文件丢到 \~/.vim/plugin 文件夹中即可

手册     在minibufexpl.vim 文件的头部

 

 

在编程的时候不可能永远只编辑一个文件, 你肯定会打开很多源文件进行编辑,

如果每个文件都打开一个vim进行编辑的话那操作起来将是多麻烦啊, 所以vim有bu

ffer(缓冲区)的概念, 可以看vim的帮助:

:help buffer

vim自带的buffer管理工具只有:ls, :bnext, :bdelete 等的命令, 既不好用,

又不直观. 现在隆重向你推荐一款vim插件(plugin): MiniBufExplorer

 

使用方法:

重新启动vim, 当你只编辑一个buffer的时候MiniBufExplorer派不上用场, 当

你打开第二个buffer的时候, MiniBufExplorer窗口就自动弹出来了, 见下图:


 

上面那个狭长的窗口就是MiniBufExplorer窗口, 其中列出了当前所有已经打开

的buffer, 当你把光标置于这个窗口时, 有下面几个快捷键可以用:

 

<Tab>   向前循环切换到每个buffer名上

<S-Tab> 向后循环切换到每个buffer名上

<Enter> 在打开光标所在的buffer

d       删除光标所在的buffer

 

在命令模式下：

:bn   打开当前buffer的下一个buffer

:bp   打开当前buffer的前一个buffer

:b"num"   打开指定的buffer，"num"指的是buffer开始的那个数字，比如上图，我想打开list_audit.erb，输入:b7就ok了

 

以下的两个功能需要在~/.vimrc中增加:

let g:miniBufExplMapCTabSwitchBufs = 1

 

<C-Tab> 向前循环切换到每个buffer上,并在但前窗口打开

<C-S-Tab> 向后循环切换到每个buffer上,并在但前窗口打开&nbsp;　　注：MiniBufExplore默认是这两个快捷键，可是在ubuntu10.04中不能使用，原因可能是bash中已经定义了ctrl+tab快捷键所以我们可以更换此快捷键

在~/.vim/plugin/minibufexpl.vim中

找到

" &nbsp;noremap <C-TAB> &nbsp; :call <SID>CycleBuffer(1)<CR>:<BS>  noremap <C-TAB>   :call <SID>CycleBuffer(1)<CR>:<BS>

noremap <C-S-TAB> :call <SID>CycleBuffer(0)<CR>:<BS>

重新定义成自己的map即可

我的为

noremap <silent> <leader>n&nbsp;&nbsp; :call <SID>CycleBuffer(1)<CR>:<BS>

noremap <silent> <leader>N&nbsp; :call <SID>CycleBuffer(0)<CR>:<BS>

这样就可以用,n&nbsp; ,N 进行buffer切换(let mapleader = "," 我已在~/.vimrc中定义leader)

如果在~/.vimrc中设置了下面这句:

let g:miniBufExplMapWindowNavVim = 1

则可以用<C-h,j,k,l>切换到上下左右的窗口中去,就像:

C-w,h j k l    向"左,下,上,右"切换窗口.

在~/.vimrc中设置:

let g:miniBufExplMapWindowNavArrows = 1

是用<C-箭头键>切换到上下左右窗口中去

 

以下是MiniBufExplorer的几个命令：

 

                :MiniBufExplorer    " Open and/or goto Explorer

                 :CMiniBufExplorer   " Close the Explorer if it's open

                :UMiniBufExplorer   " Update Explorer without naviting

                :TMiniBufExplorer   " Toggle the Explorer window open and closed

 

如果你用gvim的话，MiniBufExplorer会出现多个窗口的烦人问题，我一直没能很好地解决这个问题，一般都是用:CMiniBufExplorer命令把MiniBufExplorer窗口给close掉

 

 

如果你在.vimrc（windows底下的是_vimrc）中配置了mapleader，如我的

let mapleader = ","       "Set mapleader

 

你就可以在normal模式下用,mbc代替 :CMiniBufExplorer命令

其他命令为：

 

                :MiniBufExplorer    ,mbe

                 :CMiniBufExplorer   ,mbc

                :UMiniBufExplorer   ,mbu

                :TMiniBufExplorer  ,mbt

 

推荐大家一个安装vim插件的脚本，可以实现一个命令就把常用插件安装好，很方便：一个具有类似于IDE功能的容易安装的VIM


