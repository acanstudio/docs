taglist
========

[插件地址](http://www.vim.org/scripts/script.php?script_id=273)

安装和使用基础

<pre>
# 通过Vundle安装
Bundle 'taglist.vim'

# tags和taglist命令的帮助入口：
:help helptags
:help taglist.txt 

# 生成taglist帮助标签
:helptags ~/.vim/doc

# 标签来回跳转
通过ctrl-]来执行跳转到定义，通过Ctrl+T来跳转回来
</pre>

taglist常用配置
    * Tlist_Auto_Open：如果想在启动VIM后，自动打开taglist窗口，设置Tlist_Auto_Open为1；
    * Tlist_Close_On_Select：如果希望在选择了tag后自动关闭taglist窗口，设置Tlist_Close_On_Select为1；
    * Tlist_Ctags_Cmd：该选项用于指定ctags程序的位置，如果它没在你PATH变量所定义的路径中，需要使用此选项设置一下
    * Tlist_Exit_OnlyWindow：如果希望taglist窗口是最后一个窗口时退出VIM，设置Tlist_Exit_OnlyWindow为１；
    * Tlist_File_Fold_Auto_Close：当同时显示多个文件中的tag时，设置Tlist_File_Fold_Auto_Close为１，可使taglist只显示当前文件tag，其它文件的tag都被折叠起来。
    * Tlist_GainFocus_On_ToggleOpen：在使用:TlistToggle打开taglist窗口时，如果希望输入焦点在taglist窗口中，设置Tlist_GainFocus_On_ToggleOpen为1；
    * Tlist_Process_File_Always：如果希望taglist始终解析文件中的tag，不管taglist窗口有没有打开，设置Tlist_Process_File_Always为1；
    * Tlist_Show_One_File：如果你不想同时显示多个文件中的tag，设置Tlist_Show_One_File为1。缺省为显示多个文件中的tag；
    * Tlist_Sort_Type：设置Tlist_Sort_Type为”name”可以使taglist以tag名字进行排序，缺省是按tag在文件中出现的顺序进行排序。按tag出现的范围（即所属的namespace或class）排序，已经加入taglist的TODO List，但尚未支持；
    * Tlist_Show_Menu：在gvim中，如果你想显示taglist菜单，设置Tlist_Show_Menu为１。你可以使用Tlist_Max_Submenu_Items和Tlist_Max_Tag_Length来控制菜单条目数和所显示tag名字的长度；
    * Tlist_Use_Horiz_Window为１设置taglist窗口横向显示；
    * Tlist_Use_Right_Window：如果你想taglist窗口出现在右侧，设置Tlist_Use_Right_Window为１。缺省显示在左侧。
    * Tlist_Use_SingleClick：缺省情况下，在双击一个tag时，才会跳到该tag定义的位置，如果你想单击tag就跳转，设置Tlist_Use_SingleClick为１；
    * Tlist_WinHeight，Tlist_WinWidth： Tlist_WinHeight和Tlist_WinWidth可以设置taglist窗口的高度和宽度。
     
taglist相关命令和快捷键

<pre>
:TlistOpen # 打开taglist窗口
:TlistClose # 关闭taglist窗口
:TlistToggle # 在打开和关闭间切换。
:Tlist # taglist的打开关闭

# 可以自定义快捷键，如使用tl键就可以打开/关闭taglist窗口
nmap tl :TlistToggle<cr>

# 窗口快捷键
^CR     # 跳到光标下tag所定义的位置，用鼠标双击此tag功能也一样
o       # 在一个新打开的窗口中显示光标下tag
^Space  # 显示光标下tag的原型定义
u       # 新taglist窗口中的tag
s       # 改排序方式，在按名字排序和按出现顺序排序间切换
x       # aglist窗口放大和缩小，方便查看较长的tag
+       # 打开一个折叠，同zo
-       # 将tag折叠起来，同zc
*       # 打开所有的折叠，同zR
=       # 将所有tag折叠起来，同zM
[[      # 跳到前一个文件
]]      # 跳到后一个文件
q       # 关闭taglist窗口
^F1     # 显示帮助 
</pre>

四. .vimrc中ctags 和taglist配置项：

<pre>
" Taglist and Ctags  
let Tlist_Ctags_Cmd = '/path/to/ctags' " 指定ctags程序的位置  
let Tlist_Show_One_File = 1            " 不同时显示多个文件的tag，只显示当前文件的  
let Tlist_Exit_OnlyWindow = 1          " 如果taglist窗口是最后一个窗口，则退出vim  
let Tlist_Use_Right_Window = 1         " 在右侧窗口中显示taglist窗口  
let Tlist_File_Fold_Auto_Close=1       " 只显示当前文件tag，其它文件的tag都被折叠起来  
  
"set tags=/andes/project/mkdemo/src/tags  
"F12生成/更新tags文件  
set tags=tags;  
set autochdir  
  
"设置F12快捷键 自动生成当前目录tags文件  
function! UpdateTagsFile()  
silent !ctags -R --fields=+ianS --extra=+q  
endfunction  
nmap <F12> :call UpdateTagsFile()<CR>  
</pre>
